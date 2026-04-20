# Tools API

Media download and utility tools. Available to paid plans only.

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
curl "https://zernio.com/api/v1/tools/instagram/download?url=https://www.instagram.com/p/ABC123/" \
  -H "Authorization: Bearer YOUR_API_KEY"
```

Response:

```json
{
  "success": true,
  "downloadUrl": "https://storage.zernio.com/downloads/abc123.mp4"
}
```

## Utility Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/v1/tools/instagram/hashtag-checker` | Check hashtag restrictions |
| `GET` | `/v1/tools/youtube/transcript` | Get YouTube video transcript |
| `POST` | `/v1/tools/validate/post-length` | Check content length against platform limits |
| `POST` | `/v1/tools/validate/post` | Dry-run platform validation for a prospective post |
| `POST` | `/v1/tools/validate/media` | Check that a media URL meets platform requirements |
| `POST` | `/v1/tools/validate/subreddit` | Validate that a subreddit accepts the intended post type |

## Validators

Run the same checks Zernio applies internally, without actually publishing. Useful before letting a user schedule a post.

```bash
# Character length vs platform limits
curl -X POST https://zernio.com/api/v1/tools/validate/post-length \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{"content": "long text...", "platforms": ["twitter", "bluesky"]}'

# Full pre-publish dry run (platform-specific rules, missing required fields, etc.)
curl -X POST https://zernio.com/api/v1/tools/validate/post \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{"content": "...", "platforms": [{"platform":"instagram","accountId":"..."}], "mediaItems":[...]}'

# Media checks (dimensions, duration, codec, file size)
curl -X POST https://zernio.com/api/v1/tools/validate/media \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{"url": "https://example.com/video.mp4", "platform": "instagram", "contentType": "reel"}'

# Reddit: does the subreddit accept link/text/video posts and is a flair required
curl -X POST https://zernio.com/api/v1/tools/validate/subreddit \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{"accountId": "ACC", "subreddit": "socialmedia"}'
```

## Hashtag Checker

```bash
curl -X POST https://zernio.com/api/v1/tools/instagram/hashtag-checker \
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
curl "https://zernio.com/api/v1/tools/youtube/transcript?url=https://youtube.com/watch?v=dQw4w9WgXcQ" \
  -H "Authorization: Bearer YOUR_API_KEY"
```

Returns timestamped transcript of the video.
