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
| 422 | `VALIDATION_ERROR` | Validation failed |
| 429 | `RATE_LIMITED` | Too many requests |
| 500 | `INTERNAL_ERROR` | Server error |

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
curl "https://zernio.com/api/v1/posts/POST_ID/logs" \
  -H "Authorization: Bearer YOUR_API_KEY"
```

## Logs API

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/v1/logs` | List all publishing logs |
| `GET` | `/v1/logs/{logId}` | Get specific log entry |
| `GET` | `/v1/posts/{postId}/logs` | Get logs for a post |

Logs are retained for 7 days.

## Usage & Quota

Check the authenticated user's current plan, limits, and usage — useful for surfacing "X of Y uploads used this cycle" UI or pre-flight quota checks:

```bash
curl "https://zernio.com/api/v1/usage-stats" \
  -H "Authorization: Bearer YOUR_API_KEY"
```

Response:
```json
{
  "planName": "Pro",
  "billingPeriod": "monthly",
  "signupDate": "2024-01-15T10:30:00Z",
  "billingAnchorDay": 15,
  "limits":   { "uploads": 500, "profiles": 10 },
  "usage":    { "uploads": 127, "profiles": 3, "lastReset": "2024-11-01T00:00:00Z" }
}
```

Use `usage.lastReset` + `billingAnchorDay` to know when the next cycle flips.
