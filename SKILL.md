---
name: zernio-api
description: Official Zernio API reference for scheduling posts across 14 social media platforms. Covers authentication, endpoints, webhooks, and platform-specific features. Use when building with the Zernio Social Media Scheduling API.
---

# Zernio API Reference

Schedule posts across 14 social media platforms with a single API.

**Base URL:** `https://zernio.com/api/v1`

**Docs:** [docs.zernio.com](https://docs.zernio.com)

## Quick Start

```bash
# 1. Create profile
curl -X POST https://zernio.com/api/v1/profiles \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{"name": "My Brand"}'

# 2. Connect account (opens OAuth)
curl "https://zernio.com/api/v1/connect/twitter?profileId=PROFILE_ID" \
  -H "Authorization: Bearer YOUR_API_KEY"

# 3. Create post
curl -X POST https://zernio.com/api/v1/posts \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{"content": "Hello!", "platforms": [{"platform": "twitter", "accountId": "ACC_ID"}], "publishNow": true}'
```

## Rule Files

Read individual rule files for detailed documentation:

- [rules/authentication.md](rules/authentication.md) - API key format, usage examples, core concepts
- [rules/posts.md](rules/posts.md) - Create, schedule, retry posts, bulk upload
- [rules/accounts.md](rules/accounts.md) - List accounts, health checks, follower stats
- [rules/connect.md](rules/connect.md) - OAuth flows, Bluesky app password, Telegram bot token
- [rules/platforms.md](rules/platforms.md) - Platform-specific data for all 14 platforms
- [rules/webhooks.md](rules/webhooks.md) - Configure webhooks, verify signatures, events
- [rules/media.md](rules/media.md) - Presigned uploads, supported formats, platform limits
- [rules/queue.md](rules/queue.md) - Queue management, slots configuration
- [rules/analytics.md](rules/analytics.md) - YouTube daily views, LinkedIn analytics
- [rules/tools.md](rules/tools.md) - Media download, hashtag checker, transcripts
- [rules/errors.md](rules/errors.md) - Error codes, rate limits, publishing logs
- [rules/sdks.md](rules/sdks.md) - Direct API usage examples

## Supported Platforms

Twitter/X, Instagram, Facebook, LinkedIn, TikTok, YouTube, Pinterest, Reddit, Bluesky, Threads, Google Business, Telegram, WhatsApp, Snapchat

---

*[Zernio](https://zernio.com) - Social Media Scheduling API for Developers*
