# Error Handling

## Error Response Format

```json
{
  "error": "Invalid API key",
  "code": "UNAUTHORIZED",
  "details": {}
}
```

## HTTP Status Codes

| Status | Code | Meaning |
|--------|------|---------|
| 400 | `BAD_REQUEST` | Invalid parameters |
| 401 | `UNAUTHORIZED` | Invalid/missing API key |
| 403 | `FORBIDDEN` | Insufficient permissions |
| 404 | `NOT_FOUND` | Resource not found |
| 409 | `DUPLICATE_CONTENT` | Identical content already posted (see below) |
| 422 | `VALIDATION_ERROR` | Validation failed |
| 429 | `RATE_LIMITED` | Too many requests |
| 500 | `INTERNAL_ERROR` | Server error |

## Duplicate Content Detection (Jan 2026)

HTTP 409 is returned when identical content has already been posted to the same account:

```json
{
  "error": "This exact content was already posted to this account",
  "code": "DUPLICATE_CONTENT",
  "details": {
    "accountId": "acc_123",
    "existingPostId": "post_456"
  }
}
```

## Platform Target Errors (Feb 2026)

When a post targets multiple platforms and some fail, the post status is `"partial"`. Each platform target includes three error fields:

```json
{
  "status": "partial",
  "platformTargets": [
    {
      "platform": "twitter",
      "status": "published"
    },
    {
      "platform": "linkedin",
      "status": "failed",
      "errorMessage": "Token expired, please reconnect",
      "errorCategory": "auth_expired",
      "errorSource": "platform"
    }
  ]
}
```

**Error categories:** `auth_expired`, `user_content`, `platform_rejected`, `rate_limited`, `media_error`, `system`

**Error sources:** `user` (content issue), `platform` (platform-side rejection), `system` (Late internal error)

## Rate Limits

| Plan | Requests/Minute |
|------|-----------------|
| Free | 60 |
| Build | 120 |
| Accelerate | 600 |
| Unlimited | 1,200 |

### Rate Limit Headers

```
X-RateLimit-Limit: 60
X-RateLimit-Remaining: 45
X-RateLimit-Reset: 1709294400
```

### Handle Rate Limits

```typescript
async function fetchWithRetry(url: string, options: RequestInit) {
  const response = await fetch(url, options);

  if (response.status === 429) {
    const resetTime = response.headers.get('X-RateLimit-Reset');
    const waitMs = (Number(resetTime) * 1000) - Date.now();
    await sleep(Math.max(waitMs, 1000));
    return fetchWithRetry(url, options);
  }

  return response;
}
```

## Publishing Logs

Check post logs for platform-specific errors:

```bash
curl "https://getlate.dev/api/v1/posts/POST_ID/logs" \
  -H "Authorization: Bearer YOUR_API_KEY"
```

## Logs API

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/v1/logs` | List all publishing logs |
| `GET` | `/v1/logs/{logId}` | Get specific log entry |
| `GET` | `/v1/posts/{postId}/logs` | Get logs for a post |

Logs are retained for 7 days.
