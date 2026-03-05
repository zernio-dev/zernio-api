# End-to-End Integration Guide

Complete walkthrough of a Late API integration: onboard a user, connect their social accounts, upload media, and publish to multiple platforms.

---

## Step 1: Create a Profile

A profile is the container for a user's connected social accounts. Create one per user or brand.

```typescript
const resp = await fetch('https://getlate.dev/api/v1/profiles', {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${apiKey}`,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({ name: 'My Brand' })
});
const profile = await resp.json();
// profile.id — save this as the user's profileId
```

**Response schema:**
```json
{
  "id": "prof_abc123",
  "name": "My Brand",
  "description": null,
  "color": null,
  "default": false,
  "createdAt": "2026-01-15T10:00:00Z"
}
```

---

## Step 2: Connect a Social Account (OAuth)

This is a redirect flow — you get an `authUrl` from Late, redirect the user there, and they land back on your `redirectUrl` after authorizing.

```typescript
// GET — not POST, no body, params in query string
const connectResp = await fetch(
  `https://getlate.dev/api/v1/connect/twitter?profileId=${profileId}&redirectUrl=${encodeURIComponent(redirectUrl)}`,
  { headers: { 'Authorization': `Bearer ${apiKey}` } }
);
const { authUrl } = await connectResp.json();
// Redirect the user to authUrl
```

**Response schema:**
```json
{
  "authUrl": "https://twitter.com/oauth/authorize?..."
}
```

**For Reddit — extra steps after OAuth:**

```typescript
// After user returns from OAuth, list subreddits they moderate
const subredditsResp = await fetch(
  `https://getlate.dev/api/v1/connect/reddit/subreddits?profileId=${profileId}`,
  { headers: { 'Authorization': `Bearer ${apiKey}` } }
);
const { subreddits } = await subredditsResp.json();
// subreddits: [{ name: "r/mysubreddit", members: 1200 }, ...]

// Set default subreddit
await fetch('https://getlate.dev/api/v1/connect/reddit/update-subreddits', {
  method: 'POST',
  headers: { 'Authorization': `Bearer ${apiKey}`, 'Content-Type': 'application/json' },
  body: JSON.stringify({ profileId, subreddit: 'mysubreddit' })
});
```

**Platforms requiring post-OAuth selection:**

| Platform | Extra step |
|----------|------------|
| Facebook | `GET /v1/connect/facebook/pages` → `POST /v1/connect/facebook/select-page` |
| LinkedIn | `GET /v1/connect/linkedin/organizations` → `POST /v1/connect/linkedin/select-organization` |
| Pinterest | `GET /v1/connect/pinterest/select-board` → `POST /v1/connect/pinterest/select-board` |
| Reddit | `GET /v1/connect/reddit/subreddits` → `POST /v1/connect/reddit/update-subreddits` |
| Google Business | `GET /v1/connect/google-business/locations` → `POST /v1/connect/google-business/select-location` |
| Snapchat | `GET /v1/connect/snapchat/select-profile` → `POST /v1/connect/snapchat/select-profile` |

---

## Step 3: Get Account IDs

After connecting, list accounts to get the `accountId` values needed for posting.

```typescript
const accountsResp = await fetch(
  `https://getlate.dev/api/v1/accounts?profileId=${profileId}`,
  { headers: { 'Authorization': `Bearer ${apiKey}` } }
);
const accounts = await accountsResp.json();
// accounts: [{ id: "acc_xyz", platform: "twitter", name: "@myhandle", ... }, ...]
```

**Response schema (single account):**
```json
{
  "id": "acc_xyz789",
  "platform": "twitter",
  "name": "@myhandle",
  "profilePicture": "https://...",
  "followers": 5200,
  "status": "active"
}
```

---

## Step 4: Upload Media (Optional)

Use the presign flow — do not send files directly in the post body.

```typescript
// 1. Get presigned URL
const presignResp = await fetch('https://getlate.dev/api/v1/media/presign', {
  method: 'POST',
  headers: { 'Authorization': `Bearer ${apiKey}`, 'Content-Type': 'application/json' },
  body: JSON.stringify({ fileName: 'image.jpg', fileType: 'image/jpeg' })
});
const { uploadUrl, publicUrl } = await presignResp.json();

// 2. Upload file to presigned URL (direct PUT, no auth header)
await fetch(uploadUrl, {
  method: 'PUT',
  body: fileBuffer,
  headers: { 'Content-Type': 'image/jpeg' }
});

// 3. Use publicUrl (not uploadUrl) in your post
```

**Response schema:**
```json
{
  "uploadUrl": "https://s3.amazonaws.com/...?X-Amz-Signature=...",
  "publicUrl": "https://cdn.getlate.dev/media/abc123.jpg"
}
```

---

## Step 5: Create a Post

```typescript
const postResp = await fetch('https://getlate.dev/api/v1/posts', {
  method: 'POST',
  headers: { 'Authorization': `Bearer ${apiKey}`, 'Content-Type': 'application/json' },
  body: JSON.stringify({
    content: 'Main post text (fallback for all platforms)',
    platforms: [
      { platform: 'twitter', accountId: 'acc_tw123' },
      { platform: 'linkedin', accountId: 'acc_li456' },
      {
        platform: 'instagram',
        accountId: 'acc_ig789',
        platformSpecificData: {
          contentType: 'story',  // 'story' only; feed/reel auto-detected from media
          firstComment: 'First comment text'
        }
      }
    ],

    // Optional: platform-specific text variants (key = platform name)
    customContent: {
      twitter: 'Short version for Twitter (280 chars)',
      linkedin: 'Professional version for LinkedIn'
    },

    // Optional: shared media (use publicUrl from presign, not uploadUrl)
    mediaItems: [
      { type: 'image', url: publicUrl }
    ],

    // Scheduling: pick exactly one
    publishNow: true,
    // OR: scheduledFor: '2026-03-15T09:00:00Z',  // ISO 8601, not "scheduledAt"
    // OR: queuedFromProfile: profileId,
  })
});
const post = await postResp.json();
```

**Response schema:**
```json
{
  "id": "post_def456",
  "content": "Main post text",
  "status": "scheduled",
  "scheduledFor": "2026-03-15T09:00:00Z",
  "platforms": [
    {
      "platform": "twitter",
      "accountId": "acc_tw123",
      "status": "scheduled",
      "latePostId": "lp_001"
    }
  ],
  "createdAt": "2026-03-05T10:00:00Z"
}
```

Note: if publishing to 3 platforms and 2 succeed but 1 fails, `status` = `"partial"`. Check each platform entry's `status` field.

---

## Step 6: Handle Webhook Events (Recommended)

Set up a webhook to know when posts publish or fail:

```typescript
// Register webhook
await fetch('https://getlate.dev/api/v1/webhooks', {
  method: 'POST',
  headers: { 'Authorization': `Bearer ${apiKey}`, 'Content-Type': 'application/json' },
  body: JSON.stringify({
    url: 'https://yourapp.com/webhooks/late',
    events: ['post.published', 'post.failed']
  })
});
```

See [webhooks.md](webhooks.md) for signature verification and event payload schemas.

---

## Common Patterns

### SaaS multi-tenant: one profile per user

```typescript
// On user signup
const profile = await createProfile({ name: user.name });
await db.users.update(user.id, { lateProfileId: profile.id });

// When user connects a platform
const { authUrl } = await getConnectUrl(profile.id, platform, callbackUrl);
redirect(authUrl);
```

### Post to all connected accounts

```typescript
const accounts = await listAccounts(profileId);
const platforms = accounts.map(acc => ({
  platform: acc.platform,
  accountId: acc.id
}));
await createPost({ content, platforms, publishNow: true });
```

### Reddit-specific post with flair

```typescript
// Get available flairs first
const flairsResp = await fetch(
  `https://getlate.dev/api/v1/accounts/${accountId}/reddit-flairs?subreddit=mysubreddit`,
  { headers: { 'Authorization': `Bearer ${apiKey}` } }
);
const { flairs } = await flairsResp.json();
// flairs: [{ id: "flair_123", text: "Discussion" }, ...]

await createPost({
  content: 'Post text',
  platforms: [{
    platform: 'reddit',
    accountId,
    platformSpecificData: {
      title: 'Required Reddit title',  // required, permanent once posted
      subreddit: 'mysubreddit',
      flair: { id: 'flair_123', text: 'Discussion' }
    }
  }],
  publishNow: true
});
```
