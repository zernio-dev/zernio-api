# Validate API

Run the same checks Zernio applies internally, without publishing.

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/v1/tools/validate/post-length` | Check content length against platform limits |
| `POST` | `/v1/tools/validate/post` | Dry-run platform validation for a prospective post |
| `POST` | `/v1/tools/validate/media` | Check that a media URL meets platform requirements |
| `POST` | `/v1/tools/validate/subreddit` | Validate that a subreddit accepts the intended post type |

## Examples

Useful before letting a user schedule a post.

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
