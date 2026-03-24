# API Usage

Use the Zernio API directly with HTTP requests:

```typescript
const BASE_URL = 'https://zernio.com/api/v1';

async function zernioApi(endpoint: string, options: RequestInit = {}) {
  const response = await fetch(`${BASE_URL}${endpoint}`, {
    ...options,
    headers: {
      'Authorization': `Bearer ${process.env.ZERNIO_API_KEY}`,
      'Content-Type': 'application/json',
      ...options.headers,
    },
  });

  if (!response.ok) {
    const error = await response.json();
    throw new Error(error.error || 'API request failed');
  }

  return response.json();
}

// Usage
const posts = await zernioApi('/posts');
const newPost = await zernioApi('/posts', {
  method: 'POST',
  body: JSON.stringify({
    content: 'Hello!',
    platforms: [{ platform: 'twitter', accountId: 'acc_123' }],
    publishNow: true
  })
});
```
