# Platform-Specific Features

## Supported Platforms

| Platform | OAuth | Features |
|----------|-------|----------|
| Twitter/X | Yes | Posts, threads, images, videos |
| Instagram | Yes | Feed, Stories, Reels, Carousels |
| Facebook | Yes | Pages, Reels, Stories |
| LinkedIn | Yes | Posts, images, videos, documents |
| TikTok | Yes | Videos with privacy controls |
| YouTube | Yes | Videos, Shorts |
| Pinterest | Yes | Pins with images/videos |
| Reddit | Yes | Posts with subreddit targeting |
| Bluesky | App password | Posts, images, videos |
| Threads | Yes | Posts, images, videos |
| Google Business | Yes | Updates, photos, offers |
| Telegram | Bot token | Messages, images, videos |
| Snapchat | Yes | Stories, Spotlight |
| Discord | Yes | Webhook posts, embeds, polls, forum threads, announcements |

## Platform-Specific Data

Platform-specific data goes inside each platform entry in the `platforms` array:

```json
{
  "platforms": [
    {
      "platform": "instagram",
      "accountId": "acc_123",
      "platformSpecificData": { ... }
    }
  ]
}
```

### Twitter/X

```json
{
  "platformSpecificData": {
    "threadItems": [
      { "content": "Second tweet" },
      { "content": "Third tweet", "mediaItems": [{ "type": "image", "url": "..." }] }
    ]
  }
}
```

### Instagram

```json
{
  "platformSpecificData": {
    "contentType": "story",
    "firstComment": "First comment!",
    "collaborators": ["username"],
    "shareToFeed": true,
    "userTags": [{ "username": "user", "x": 0.5, "y": 0.5 }],
    "trialParams": { "graduationStrategy": "SS_PERFORMANCE" },
    "audioName": "My Custom Audio",
    "thumbOffset": 5000
  }
}
```

- `contentType: "story"` publishes as a Story. Default posts become Reels or feed based on media.
- `trialParams` for Trial Reels (non-followers first): `MANUAL` or `SS_PERFORMANCE` (auto-graduate)
- `audioName` sets custom label for original audio in Reels
- `thumbOffset` selects thumbnail frame (milliseconds from start)

### TikTok

```json
{
  "platformSpecificData": {
    "privacyLevel": "PUBLIC_TO_EVERYONE",
    "allowComment": true,
    "allowDuet": true,
    "allowStitch": true,
    "contentPreviewConfirmed": true,
    "expressConsentGiven": true,
    "draft": false,
    "commercialContentType": "none",
    "videoMadeWithAi": false,
    "videoCoverTimestampMs": 1000,
    "photoCoverIndex": 0,
    "autoAddMusic": false,
    "description": "Extended description for photo posts (max 4000 chars)"
  }
}
```

**Required fields:**
- `contentPreviewConfirmed` and `expressConsentGiven` must be `true`
- `allowDuet`, `allowStitch` required for videos; `allowComment` for all

**Optional fields:**
- `draft: true` sends to Creator Inbox instead of publishing
- `commercialContentType`: `none`, `brand_organic`, `brand_content`
- `brandPartnerPromote`: Whether the post promotes a brand partner
- `isBrandOrganicPost`: Whether the post is a brand organic post
- `mediaType`: `video` or `photo` (auto-detected from media)
- `description` for photo posts when content exceeds 90 chars

### YouTube

```json
{
  "platformSpecificData": {
    "title": "Video Title",
    "visibility": "public",
    "firstComment": "Check out my other videos!",
    "containsSyntheticMedia": false
  }
}
```

- `visibility`: `public`, `private`, `unlisted`
- `firstComment`: Optional comment posted after upload (max 10,000 chars)
- `containsSyntheticMedia`: Set `true` for AI-generated content disclosure
- Videos ≤3 min auto-detected as Shorts; >3 min as regular videos
- Use top-level `tags` array for video tags (≤500 chars total)

### Reddit

```json
{
  "platformSpecificData": {
    "subreddit": "socialmedia",
    "title": "Post title (defaults to first line of content, max 300 chars)",
    "flairId": "a1b2c3d4-e5f6-7890-abcd-ef1234567890",
    "url": "https://example.com",
    "forceSelf": false,
    "nativeVideo": true,
    "videogif": false,
    "videoPosterUrl": "https://example.com/poster.jpg"
  }
}
```

- Posts are **link** (with `url`/media), **native video** (via `nativeVideo`), or **self** (text-only).
- `subreddit`: without `r/` prefix. Defaults to the account's configured subreddit. List available with `GET /v1/accounts/{id}/reddit-subreddits`.
- `flairId`: required by some subreddits. List with `GET /v1/accounts/{id}/reddit-flairs?subreddit=name`.
- `nativeVideo` (default `true` when media is a video) uploads to Reddit's CDN and submits with `kind=video`, rendering as an embedded player. Reddit transcodes server-side (1080p / 30 fps cap). Set `false` to fall back to a legacy link post. If the subreddit blocks video, falls back automatically.
- `videogif: true` submits as a silent looping videogif (`kind=videogif`).
- `videoPosterUrl`: optional thumbnail for native video. If omitted, Reddit extracts the first frame.
- `forceSelf: true` creates a text/self post even when a URL or media is provided.

### Discord

```json
{
  "platformSpecificData": {
    "channelId": "1234567890123456789",
    "embeds": [
      {
        "title": "Release v2.0",
        "description": "Ships today \ud83d\ude80",
        "color": 5814783,
        "fields": [
          { "name": "Platform", "value": "All", "inline": true }
        ]
      }
    ],
    "poll": {
      "question": { "text": "Favorite feature?" },
      "answers": [
        { "poll_media": { "text": "Threads" } },
        { "poll_media": { "text": "Polls" } }
      ],
      "duration": 24,
      "allow_multiselect": false
    },
    "tts": false,
    "webhookUsername": "My Brand",
    "webhookAvatarUrl": "https://example.com/logo.png",
    "crosspost": true,
    "forumThreadName": "Weekly update",
    "forumAppliedTags": ["1111111111111111111"],
    "threadFromMessage": {
      "name": "Discussion",
      "autoArchiveDuration": 1440
    }
  }
}
```

- `channelId` (**required**): target channel snowflake. Use `GET /v1/accounts/{id}/discord-channels` to list available channels.
- **Text limit:** 2,000 characters. Attachments: images (JPEG, PNG, GIF, WebP), videos (MP4), documents, up to 10 files, 25 MB each.
- `embeds[]`: up to 10 rich embed objects (combined max 6,000 chars). Each supports `title` (256), `description` (4,096), `url`, `color` (decimal integer — convert from hex), `image.url`, `thumbnail.url`, `footer.text` (2,048) / `icon_url`, `author.name` (256) / `url` / `icon_url`, and up to 25 `fields` (`name` 256, `value` 1,024, optional `inline`).
- `poll`: native Discord poll. Max 10 answers, duration 1-768 hours (default 24). **Cannot** be combined with media attachments.
- `webhookUsername` / `webhookAvatarUrl`: override the default webhook identity for this post. Defaults come from account-level settings (see `PATCH /v1/accounts/{id}/discord-settings`).
- `tts: true`: text-to-speech.
- `crosspost: true`: auto-crosspost to every server following this announcement channel (channel type 5). No-op on regular text channels.
- `forumThreadName`: **required** when posting to a forum channel (type 15). `forumAppliedTags[]`: up to 5 tag snowflake IDs.
- `threadFromMessage`: creates a follow-up thread under the published message. `autoArchiveDuration` minutes: `60`, `1440`, `4320`, `10080`. `rateLimitPerUser` seconds: 0-21600.

### Pinterest

```json
{
  "platformSpecificData": {
    "title": "Pin Title",
    "boardId": "board_123",
    "link": "https://example.com",
    "coverImageUrl": "https://example.com/cover.jpg",
    "coverImageKeyFrameTime": 5
  }
}
```

- `title`: Pin title (max 100 chars, defaults to first line of content)
- `boardId`: Target board (uses first available if omitted)
- `coverImageUrl`: Optional cover image for video pins
- `coverImageKeyFrameTime`: Key frame time in seconds for video cover

### Google Business

```json
{
  "platformSpecificData": {
    "callToAction": {
      "type": "LEARN_MORE",
      "url": "https://example.com"
    }
  }
}
```

Action types: `BOOK`, `ORDER`, `SHOP`, `LEARN_MORE`, `SIGN_UP`, `CALL`

### LinkedIn

```json
{
  "platformSpecificData": {
    "firstComment": "First comment on the post",
    "disableLinkPreview": false
  }
}
```

Supports up to 20 images, single PDF documents (max 100MB), and link previews for URLs.

### Facebook

```json
{
  "platformSpecificData": {
    "contentType": "story",
    "firstComment": "First comment!",
    "pageId": "123456789"
  }
}
```

Set `contentType: "story"` to publish as a Facebook Page Story (24-hour ephemeral). Supports up to 10 images for feed posts.

### Threads

```json
{
  "platformSpecificData": {
    "threadItems": [
      { "content": "Second post in thread" },
      { "content": "Third post", "mediaItems": [{ "type": "image", "url": "..." }] }
    ]
  }
}
```

Creates reply chains (Threads equivalent of Twitter threads). Supports up to 10 images per carousel.

### Telegram

```json
{
  "platformSpecificData": {
    "parseMode": "HTML",
    "disableWebPagePreview": false,
    "disableNotification": false,
    "protectContent": false
  }
}
```

Parse modes: `HTML`, `Markdown`, `MarkdownV2`. Supports up to 10 images or videos in albums. Max 4096 chars for text-only, 1024 for media captions.

### Snapchat

```json
{
  "platformSpecificData": {
    "contentType": "story"
  }
}
```

Content types:
- `story` - Ephemeral (24 hours), no text caption
- `saved_story` - Permanent on Public Profile, title max 45 chars
- `spotlight` - Video for entertainment feed, description max 160 chars
