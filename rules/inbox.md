# Inbox

Unified DMs, comments, and reviews across all messaging-capable platforms. **Requires the Inbox add-on.** Endpoints return `403 Inbox addon required` without it.

## Conversations & Messages (DMs)

Supported: Facebook, Instagram, Twitter/X, Bluesky, Reddit, Telegram, WhatsApp.

| Method | Path | Purpose |
|--------|------|---------|
| GET    | `/v1/inbox/conversations` | List conversations across all accounts (aggregated) |
| POST   | `/v1/inbox/conversations` | Start a new conversation (**Twitter/X only**) |
| GET    | `/v1/inbox/conversations/{conversationId}` | Conversation details (needs `accountId` query) |
| PUT    | `/v1/inbox/conversations/{conversationId}` | Archive / activate |
| GET    | `/v1/inbox/conversations/{conversationId}/messages` | List messages |
| POST   | `/v1/inbox/conversations/{conversationId}/messages` | Send message |
| PATCH  | `/v1/inbox/conversations/{conversationId}/messages/{messageId}` | Edit message (**Telegram only**) |
| DELETE | `/v1/inbox/conversations/{conversationId}/messages/{messageId}` | Delete message |
| POST   | `/v1/inbox/conversations/{conversationId}/typing` | Typing indicator |
| POST   | `/v1/inbox/conversations/{conversationId}/messages/{messageId}/reactions` | Add reaction |
| DELETE | `/v1/inbox/conversations/{conversationId}/messages/{messageId}/reactions` | Remove reaction |

### List conversations

```bash
curl "https://zernio.com/api/v1/inbox/conversations?platform=instagram&status=active&limit=50" \
  -H "Authorization: Bearer YOUR_API_KEY"
```

Filters: `profileId`, `platform`, `accountId`, `status` (`active`/`archived`), `sortOrder`, `limit` (max 100), `cursor`. Cross-account, deduplicated. Response `meta.failedAccounts[]` lists accounts that couldn't be fetched (rate limits, auth issues) with `retryAfter` seconds.

**Twitter/X limitation:** encrypted "X Chat" messages are not accessible via the X API. Those conversations may appear empty or show only your outgoing messages.

### Create conversation (Twitter/X only)

```bash
curl -X POST https://zernio.com/api/v1/inbox/conversations \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{
    "accountId": "ACCOUNT_ID",
    "participantUsername": "elonmusk",
    "message": "Hello"
  }'
```

Provide either `participantId` (numeric) or `participantUsername`. `422 DM_NOT_ALLOWED` if the recipient doesn't accept DMs — set `"skipDmCheck": true` to bypass after you've verified. Requires X API Pro tier for BYOK users. Rate limits: 200 req / 15 min, 1,000 / 24h per user, 15,000 / 24h per app.

### Send message

Supports text, attachments, quick replies, buttons, templates, WhatsApp interactive messages (list / CTA URL / Flow), Telegram keyboards, and Facebook message tags.

```bash
# Plain text + quick replies (Meta)
curl -X POST https://zernio.com/api/v1/inbox/conversations/CONV_ID/messages \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{
    "accountId": "ACCOUNT_ID",
    "message": "Pick an option",
    "quickReplies": [
      { "title": "Pricing", "payload": "PRICING" },
      { "title": "Support", "payload": "SUPPORT" }
    ]
  }'

# Attachment via public URL
# {"accountId": "...", "attachmentUrl": "https://...", "attachmentType": "image"}

# Or multipart/form-data with a binary `attachment` field (max 25 MB)
```

**Option matrix:**

| Field | Purpose | Platforms |
|-------|---------|-----------|
| `quickReplies` | Up to 13, mutually exclusive with `buttons` | Meta |
| `buttons` | Up to 3, types `url` / `postback` / `phone` (FB only) | Meta |
| `template` | Generic carousel, up to 10 elements | Instagram, Facebook |
| `interactive` | WhatsApp `list` / `cta_url` / `flow` (Meta Cloud API verbatim) | WhatsApp |
| `replyMarkup` | Telegram inline / reply keyboard | Telegram |
| `messagingType` + `messageTag` | Send outside 24h window (`CONFIRMED_EVENT_UPDATE`, `POST_PURCHASE_UPDATE`, `ACCOUNT_UPDATE`, `HUMAN_AGENT`) | Facebook. IG: `HUMAN_AGENT` only |
| `replyTo` | Quote-reply to platform message ID (WhatsApp `wamid`, Telegram message ID) | WhatsApp, Telegram |

Message lifecycle fields (`isEdited`, `editedAt`, `editHistory[]`, `isDeleted`, `deletedAt`, `deliveryStatus`, `deliveredAt`, `readAt`, `deliveryError`) are populated from webhook events. Deleted messages retain their original text and attachments for moderation/compliance.

### Edit / delete messages

- **Edit:** Telegram only. `PATCH` with `text` and/or `replyMarkup`.
- **Delete:** Telegram (bot's own or admin), X/Twitter (own DMs), Bluesky (self-view only), Reddit (sender view only). Facebook, Instagram, WhatsApp **not supported** (returns 400).

### Typing indicator

Facebook (20s), Telegram (5s). All others 200 no-op.

### Reactions

Telegram (subset of Unicode), WhatsApp (any emoji, one per message per sender). All others 400.

## Comments

Supported: Facebook, Instagram, Twitter/X, Bluesky, Threads, YouTube, LinkedIn, Reddit.

| Method | Path | Purpose |
|--------|------|---------|
| GET    | `/v1/inbox/comments` | List posts with comment counts (cross-account) |
| GET    | `/v1/inbox/comments/{postId}` | List comments on a post |
| POST   | `/v1/inbox/comments/{postId}` | Reply to a post or comment |
| DELETE | `/v1/inbox/comments/{postId}` | Delete a comment (needs `commentId` query) |
| POST   | `/v1/inbox/comments/{postId}/{commentId}/hide` | Hide |
| DELETE | `/v1/inbox/comments/{postId}/{commentId}/hide` | Unhide |
| POST   | `/v1/inbox/comments/{postId}/{commentId}/like` | Like / upvote |
| DELETE | `/v1/inbox/comments/{postId}/{commentId}/like` | Unlike |
| POST   | `/v1/inbox/comments/{postId}/{commentId}/private-reply` | Send private DM to commenter (IG/FB only) |

`postId` accepts Zernio post ID or platform post ID. LinkedIn third-party posts accept full activity URN or numeric ID. Reddit needs `subreddit` query param; use `commentId` query to get replies to a specific comment.

**Capability flags** returned per comment: `canReply`, `canDelete`, `canHide` (FB/IG/Threads), `canLike` (FB/X/Bluesky/Reddit), `isHidden`, `isLiked`.

Reply:
```bash
curl -X POST https://zernio.com/api/v1/inbox/comments/POST_ID \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{"accountId": "ACC", "message": "Thanks!", "commentId": "COMMENT_ID"}'
```

Bluesky requires `parentCid`, `rootUri`, `rootCid`. Bluesky liking requires `cid` in the request body; unliking requires `likeUri` query.

**Hide** supported on Facebook, Instagram, Threads, X/Twitter (reply must be in a conversation you started).
**Like** supported on Facebook, X/Twitter, Bluesky, Reddit.
**Delete** supported on Facebook, Instagram, Bluesky, Reddit, YouTube, LinkedIn.

### Private reply (IG + FB)

Turn a public comment into a private DM. **One reply per comment, 7-day window, text only.**

```bash
curl -X POST https://zernio.com/api/v1/inbox/comments/POST_ID/COMMENT_ID/private-reply \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{"accountId": "ACC", "message": "Hi — reaching out privately."}'
```

## Reviews

Supported: Facebook Pages, Google Business.

| Method | Path | Purpose |
|--------|------|---------|
| GET    | `/v1/inbox/reviews` | List reviews (aggregated, with rating/date filters) |
| POST   | `/v1/inbox/reviews/{reviewId}/reply` | Reply to a review |
| DELETE | `/v1/inbox/reviews/{reviewId}/reply` | Delete reply (**Google Business only**) |

Filters: `platform`, `minRating`, `maxRating`, `hasReply`, `sortBy` (`date`/`rating`). Response includes `summary.totalReviews` and `summary.averageRating`.

Google Business `reviewId` needs to be URL-encoded.

## Chat config (per-account settings)

| Method | Path | Purpose |
|--------|------|---------|
| GET / PUT / DELETE | `/v1/accounts/{accountId}/messenger-menu` | Facebook Messenger persistent menu (max 3 top-level, 5 nested) |
| GET / PUT / DELETE | `/v1/accounts/{accountId}/instagram-ice-breakers` | Instagram ice breakers (max 4, question ≤ 80 chars) |
| GET / PUT / DELETE | `/v1/accounts/{accountId}/telegram-commands` | Telegram bot commands (`command` without leading `/`, `description`) |

Example — set Telegram commands:
```bash
curl -X PUT https://zernio.com/api/v1/accounts/ACCOUNT_ID/telegram-commands \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{
    "commands": [
      { "command": "help", "description": "Show help" },
      { "command": "pricing", "description": "See pricing plans" }
    ]
  }'
```

## Media uploads for messages

When you need to attach a file you only have locally, upload it first and pass the returned URL as `attachmentUrl`:

```bash
curl -X POST https://zernio.com/api/v1/media/upload-direct \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -F "file=@./photo.jpg"
# -> { "url": "https://...", "filename": "...", "contentType": "...", "size": 123456 }
```

Files auto-delete after 7 days. Max 25 MB. Optional `contentType` field overrides MIME detection.

## Common behavior

- All inbox responses include a `meta.failedAccounts[]` array when one or more underlying account fetches fail (rate limits, auth). Use `retryAfter` to back off.
- Cursor-based pagination: pass `pagination.nextCursor` back as `cursor`.
- `accountId` can always be used to scope to a single connected account.
