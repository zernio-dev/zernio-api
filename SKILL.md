---
name: zernio-api
description: Official Zernio API reference for scheduling posts across 18 social and messaging platforms, running paid ads on 9 ad networks, and sending server-side conversions. Covers authentication, endpoints, webhooks, ads, conversions, automations, and platform-specific features. Use when building with the Zernio Social Media Scheduling API.
---

# Zernio API Reference

Schedule posts across 18 social and messaging platforms and run paid ads on 9 ad networks with a single API.

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

Read individual rule files for detailed documentation.

**Publishing & core:**

- [rules/authentication.md](rules/authentication.md) - API key format, usage examples, core concepts
- [rules/posts.md](rules/posts.md) - Create, schedule, retry, edit, unpublish, bulk upload
- [rules/platforms.md](rules/platforms.md) - Platform-specific data per platform
- [rules/media.md](rules/media.md) - Presigned uploads, direct upload, supported formats, platform limits
- [rules/queue.md](rules/queue.md) - Queue management, slots configuration

**Accounts & connections:**

- [rules/accounts.md](rules/accounts.md) - List accounts, health checks, per-platform helpers (Discord, TikTok creator-info, LinkedIn orgs, etc.)
- [rules/account-groups.md](rules/account-groups.md) - Group accounts for bulk operations
- [rules/connect.md](rules/connect.md) - OAuth flows, headless mode, ads connections, Bluesky/WhatsApp credentials, Telegram bot, HighLevel CRM

**Paid ads:**

- [rules/ads.md](rules/ads.md) - List, boost, create, analytics, audiences, Google negative keywords, Conversions API (`POST /v1/events`), Conversion Leak Report (9 ad networks)
- [rules/fix-conversion-tracking.md](rules/fix-conversion-tracking.md) - Recipe: scan a site's conversion tracking, install the Zernio site tag, serve it from the site's domain, forward server-side, load pixels behind the cookie banner, verify, rescan

**Messaging & engagement:**

- [rules/inbox.md](rules/inbox.md) - DMs, comments, reviews, private replies, chat config
- [rules/whatsapp.md](rules/whatsapp.md) - Templates, Flows, business profile, phone numbers, group chats
- [rules/contacts.md](rules/contacts.md) - Contacts CRUD, bulk import, custom fields
- [rules/broadcasts.md](rules/broadcasts.md) - Send a single message to many contacts
- [rules/sequences.md](rules/sequences.md) - `/v1/automations` (every kind), drip sequences, comment-to-DM automations
- [rules/twitter-actions.md](rules/twitter-actions.md) - Retweet, bookmark, follow
- [rules/reddit.md](rules/reddit.md) - Reddit search and subreddit feeds

**Analytics & reporting:**

- [rules/analytics.md](rules/analytics.md) - Platform analytics, demographics, best-time, content decay, Google Business performance
- [rules/webhooks.md](rules/webhooks.md) - Configure webhooks, verify signatures, events

**Google Business management:**

- [rules/gmb.md](rules/gmb.md) - Reviews, location details, attributes, services, food menus, photos, action links

**Tools:**

- [rules/tools.md](rules/tools.md) - Media download, hashtag checker, transcripts, validators (post / media / subreddit)

**Admin:**

- [rules/users.md](rules/users.md) - Workspace users + team invite tokens
- [rules/api-keys.md](rules/api-keys.md) - API key lifecycle (create, list, revoke)
- [rules/errors.md](rules/errors.md) - Error codes, rate limits, publishing logs, usage stats
- [rules/sdks.md](rules/sdks.md) - Direct API usage examples

## Supported Platforms

**Posting (18):** Instagram, TikTok, X, LinkedIn, Facebook, YouTube, Threads, Pinterest, Bluesky, Google Business, Snapchat, WhatsApp, Telegram, Discord, Slack, iMessage, LINE, WeChat

**Ads (9):** Meta (Facebook + Instagram), Google Ads, TikTok Ads, LinkedIn Ads, Pinterest Ads, X Ads, Reddit Ads, OpenAI Ads, Microsoft Ads

**Conversions (18 destinations):** the 9 ad networks plus Snapchat, The Trade Desk, Amazon Ads, AppLovin, GA4, Klaviyo, Attentive, Impact, Awin

---

*[Zernio](https://zernio.com) - Social Media Scheduling API for Developers*
