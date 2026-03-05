---
name: late-api
description: Official Late API reference for scheduling posts across 13 social media platforms. Covers authentication, endpoints, webhooks, and platform-specific features. Use when building with the Late Social Media Scheduling API.
---

# Late API Reference

Schedule posts across 13 social media platforms with a single API.

**Base URL:** `https://getlate.dev/api/v1`

**Docs:** [getlate.dev/docs](https://getlate.dev/docs)

## Quick Start

```bash
# 1. Create profile
curl -X POST https://getlate.dev/api/v1/profiles \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{"name": "My Brand"}'

# 2. Connect account (opens OAuth)
curl "https://getlate.dev/api/v1/connect/twitter?profileId=PROFILE_ID" \
  -H "Authorization: Bearer YOUR_API_KEY"

# 3. Create post
curl -X POST https://getlate.dev/api/v1/posts \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{"content": "Hello!", "platforms": [{"platform": "twitter", "accountId": "ACC_ID"}], "publishNow": true}'
```

## Rule Files

Read individual rule files for detailed documentation:

- [rules/authentication.md](rules/authentication.md) - API key format, usage examples, core concepts
- [rules/profiles.md](rules/profiles.md) - Create, list, update, delete profiles (account containers)
- [rules/posts.md](rules/posts.md) - Create, schedule, retry, unpublish posts, bulk upload
- [rules/accounts.md](rules/accounts.md) - List accounts, health checks, follower stats, Reddit flairs
- [rules/connect.md](rules/connect.md) - OAuth flows, headless mode, Bluesky, Telegram, Reddit subreddits
- [rules/platforms.md](rules/platforms.md) - Platform-specific data for all 13 platforms including Reddit
- [rules/webhooks.md](rules/webhooks.md) - Configure webhooks, verify signatures, events
- [rules/media.md](rules/media.md) - Presigned uploads, custom media per platform, auto-compression
- [rules/queue.md](rules/queue.md) - Queue management, slots configuration
- [rules/analytics.md](rules/analytics.md) - Daily metrics, best times, content decay, YouTube, LinkedIn
- [rules/tools.md](rules/tools.md) - Media download, validation endpoints, hashtag checker
- [rules/errors.md](rules/errors.md) - Error codes, rate limits, duplicate detection, platform errors
- [rules/sdks.md](rules/sdks.md) - Direct API usage examples

## Supported Platforms

Twitter/X, Instagram, Facebook, LinkedIn, TikTok, YouTube, Pinterest, Reddit, Bluesky, Threads, Google Business, Telegram, Snapchat

---

*[Late](https://getlate.dev) - Social Media Scheduling API for Developers*
