---
name: late-api
description: Official Late API (getlate.dev) reference for scheduling, publishing, and managing social media posts across 13 platforms. Use this skill whenever the user is working with getlate.dev, the Late social media scheduling API, connecting social accounts via Late OAuth, uploading media to Late, debugging Late API errors, or integrating Late into their app. Covers authentication, all endpoints, webhooks, platform-specific data, media uploads, and error handling. Always use this skill rather than guessing Late API shapes — Claude frequently gets the endpoint paths and field names wrong without it.
---

# Late API Reference

Schedule posts across 13 social media platforms with a single API.

**Base URL:** `https://getlate.dev/api/v1`

**Docs:** [docs.getlate.dev](https://docs.getlate.dev)

## Quick Start

```bash
# 1. Create profile
curl -X POST https://getlate.dev/api/v1/profiles \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{"name": "My Brand"}'

# 2. Connect account (opens OAuth) — redirectUrl is required
curl "https://getlate.dev/api/v1/connect/twitter?profileId=PROFILE_ID&redirectUrl=https://yourapp.com/callback" \
  -H "Authorization: Bearer YOUR_API_KEY"
# Returns { "authUrl": "https://twitter.com/oauth/..." } — redirect user there

# 3. Create post
curl -X POST https://getlate.dev/api/v1/posts \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{"content": "Hello!", "platforms": [{"platform": "twitter", "accountId": "ACC_ID"}], "publishNow": true}'
```

## Common Gotchas

These are the most frequent mistakes when using the Late API without reference docs:

| What Claude gets wrong | What's actually correct |
|------------------------|------------------------|
| `POST /profiles/{id}/connect` | `GET /v1/connect/{platform}?profileId=...&redirectUrl=...` |
| `POST /profiles/{id}/publish` | `POST /v1/posts` |
| `scheduledAt: "..."` | `scheduledFor: "..."` (ISO 8601) |
| `overrides: { twitter: {...} }` | `customContent: { twitter: "..." }` |
| `profiles: [{ profileKey: "..." }]` | `platforms: [{ platform: "twitter", accountId: "..." }]` |
| `platformSpecificData.postType: "story"` | `platformSpecificData.contentType: "story"` |
| Connect returns `{ url: "..." }` | Connect returns `{ authUrl: "..." }` |
| Media → use `uploadUrl` in post | Media → use `publicUrl` in post (not `uploadUrl`) |
| Upload media with FormData | Media uses presign flow: `POST /v1/media/presign` → PUT to `uploadUrl` |

## End-to-End Integration Guide

See [rules/integration-guide.md](rules/integration-guide.md) for a complete walkthrough: profile creation → OAuth connect → media upload → post with scheduling, custom content, and platform-specific data.

## Rule Files

Read individual rule files for detailed documentation:

- [rules/authentication.md](rules/authentication.md) - API key format, usage examples, core concepts
- [rules/profiles.md](rules/profiles.md) - Create, list, update, delete profiles (account containers)
- [rules/posts.md](rules/posts.md) - Create, schedule, retry, unpublish posts, bulk upload; response schema
- [rules/accounts.md](rules/accounts.md) - List accounts, health checks, follower stats, Reddit flairs
- [rules/connect.md](rules/connect.md) - OAuth flows, headless mode, Bluesky, Telegram, Reddit subreddits; response schemas
- [rules/platforms.md](rules/platforms.md) - Platform-specific data for all 13 platforms including Reddit
- [rules/webhooks.md](rules/webhooks.md) - Configure webhooks, verify signatures, events
- [rules/media.md](rules/media.md) - Presigned uploads, custom media per platform, auto-compression; response schemas
- [rules/queue.md](rules/queue.md) - Queue management, slots configuration
- [rules/analytics.md](rules/analytics.md) - Daily metrics, best times, content decay, YouTube, LinkedIn
- [rules/tools.md](rules/tools.md) - Media download, validation endpoints, hashtag checker
- [rules/errors.md](rules/errors.md) - Error codes, rate limits, duplicate detection, platform errors
- [rules/sdks.md](rules/sdks.md) - Direct API usage examples
- [rules/integration-guide.md](rules/integration-guide.md) - Full end-to-end integration walkthrough

## Pricing Context

Late plans affect architecture decisions in multi-tenant apps:

| Plan | Profiles | Posts/mo | Use case |
|------|----------|----------|----------|
| Free | 2 | 20 | Testing only |
| Build ($19/mo) | 10 | 120 | Personal tools only — 120 posts/mo is a shared pool |
| Accelerate ($49/mo) | 50 (stackable) | Unlimited | Minimum viable for multi-user SaaS |
| Unlimited ($999/mo) | Unlimited | Unlimited | Scale |

**For multi-user SaaS**: each end-user = 1 profile. Build plan's 120 post/mo limit is account-level (shared across all users) — makes it unusable for SaaS. Use Accelerate minimum.

## Supported Platforms

Twitter/X, Instagram, Facebook, LinkedIn, TikTok, YouTube, Pinterest, Reddit, Bluesky, Threads, Google Business, Telegram, Snapchat

---

*[Late](https://getlate.dev) - Social Media Scheduling API for Developers*
