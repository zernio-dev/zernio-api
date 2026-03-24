<p align="center">
  <a href="https://zernio.com">
    <img src="https://zernio.com/brand/icon-primary.png" alt="Zernio" width="60">
  </a>
</p>

<h1 align="center">Zernio API Skill</h1>

<p align="center">
  <a href="https://clawhub.ai/mikipalet/zernio-api"><img src="https://img.shields.io/badge/ClawHub-zernio--api-blue" alt="ClawHub"></a>
  <a href="LICENSE"><img src="https://img.shields.io/badge/License-MIT-yellow.svg" alt="License"></a>
</p>

Official Claude Code skill for the [Zernio](https://zernio.com) Social Media Scheduling API.

## Installation

```bash
npx clawhub@latest install zernio-api
```

## What's Included

- **Authentication** - API key setup and usage
- **Quick Start** - Get posting in 4 steps
- **All Endpoints** - Posts, accounts, profiles, webhooks, media
- **Platform-Specific** - Features for all 14 platforms
- **Webhooks** - Setup and signature verification
- **Error Handling** - Common errors and solutions
- **Code Examples** - curl, TypeScript, Python

## 14 Platforms Supported

Twitter/X, Instagram, Facebook, LinkedIn, TikTok, YouTube, Pinterest, Reddit, Bluesky, Threads, Google Business, Telegram, WhatsApp, Snapchat

## Usage

Once installed, Claude Code will help you with Zernio API integration:

```
/zernio-api
```

## Example Prompts

- "How do I create a post with the Zernio API?"
- "Show me how to set up webhooks"
- "What's the endpoint for connecting a Twitter account?"
- "How do I upload media to Zernio?"

## Quick Example

```typescript
const response = await fetch('https://zernio.com/api/v1/posts', {
  method: 'POST',
  headers: {
    'Authorization': `Bearer ${process.env.ZERNIO_API_KEY}`,
    'Content-Type': 'application/json'
  },
  body: JSON.stringify({
    content: 'Hello from Zernio API!',
    platforms: [{ platform: 'twitter', accountId: 'acc_123' }],
    publishNow: true
  })
});
```

## Resources

- **Website:** [zernio.com](https://zernio.com)
- **Docs:** [docs.zernio.com](https://docs.zernio.com)
- **Dashboard:** [zernio.com/dashboard](https://zernio.com/dashboard)
- **API Keys:** [zernio.com/dashboard/api-keys](https://zernio.com/dashboard/api-keys)

## License

MIT

---

*[Zernio](https://zernio.com) - Social Media Scheduling API for Developers*
