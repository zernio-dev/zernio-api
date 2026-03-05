# Tools API

Media download, validation, and utility tools. Available to paid plans only.

**Rate limits:** Build (50/day), Accelerate (500/day), Unlimited (unlimited)

## Download Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/v1/tools/instagram/download` | Download Instagram media |
| `GET` | `/v1/tools/tiktok/download` | Download TikTok video |
| `GET` | `/v1/tools/twitter/download` | Download Twitter media |
| `GET` | `/v1/tools/youtube/download` | Download YouTube video |
| `GET` | `/v1/tools/linkedin/download` | Download LinkedIn media |
| `GET` | `/v1/tools/facebook/download` | Download Facebook media |
| `GET` | `/v1/tools/bluesky/download` | Download Bluesky media |

## Download Example

```bash
curl "https://getlate.dev/api/v1/tools/instagram/download?url=https://www.instagram.com/p/ABC123/" \
  -H "Authorization: Bearer YOUR_API_KEY"
```

Response:

```json
{
  "success": true,
  "downloadUrl": "https://storage.getlate.dev/downloads/abc123.mp4"
}
```

## Validation Endpoints (Feb 2026)

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/v1/tools/validate/post-length` | Check character count per platform with limits |
| `POST` | `/v1/tools/validate/post` | Full pipeline dry-run across all platforms |
| `POST` | `/v1/tools/validate/media` | Validate media URL against platform size limits |
| `GET` | `/v1/tools/validate/subreddit` | Verify subreddit existence and fetch info |

### Validate Post Length

```bash
curl -X POST https://getlate.dev/api/v1/tools/validate/post-length \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"content": "Your post content here", "platforms": ["twitter", "linkedin"]}'
```

Returns character count per platform and whether content exceeds platform limits.

### Validate Post (Dry Run)

```bash
curl -X POST https://getlate.dev/api/v1/tools/validate/post \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "content": "Your post content",
    "platforms": [{"platform": "twitter", "accountId": "acc_123"}],
    "mediaItems": [{"type": "image", "url": "https://..."}]
  }'
```

Runs the full publishing pipeline as a dry-run — validates content, media, and platform requirements without actually posting.

### Validate Media

```bash
curl -X POST https://getlate.dev/api/v1/tools/validate/media \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"url": "https://example.com/image.jpg", "platforms": ["instagram", "twitter"]}'
```

Validates a public media URL against platform size and format limits.

### Validate Subreddit

```bash
curl "https://getlate.dev/api/v1/tools/validate/subreddit?name=distrilicious" \
  -H "Authorization: Bearer YOUR_API_KEY"
```

Verifies subreddit exists and returns basic info (subscriber count, rules, etc.).

## Utility Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/v1/tools/instagram/hashtag-checker` | Check hashtag restrictions |
| `GET` | `/v1/tools/youtube/transcript` | Get YouTube video transcript |

## Hashtag Checker

```bash
curl -X POST https://getlate.dev/api/v1/tools/instagram/hashtag-checker \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"hashtags": ["travel", "photography", "banned123"]}'
```

Response:

```json
{
  "results": [
    { "hashtag": "travel", "status": "safe" },
    { "hashtag": "photography", "status": "safe" },
    { "hashtag": "banned123", "status": "restricted", "reason": "Community guidelines" }
  ]
}
```

Status values: `safe`, `restricted`, `banned`, `unknown`

## YouTube Transcript

```bash
curl "https://getlate.dev/api/v1/tools/youtube/transcript?url=https://youtube.com/watch?v=dQw4w9WgXcQ" \
  -H "Authorization: Bearer YOUR_API_KEY"
```

Returns timestamped transcript of the video.
