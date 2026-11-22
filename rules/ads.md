# Ads

Create and manage paid ad campaigns across 9 ad networks (Meta, Google, TikTok, LinkedIn, Pinterest, X, Reddit, OpenAI, Microsoft) from the same API used for organic posts.

**`platform` values:** `facebook`, `instagram`, `tiktok`, `linkedin`, `pinterest`, `google`, `twitter`, `reddit`, `openai`, `microsoft`

**Requires the Ads add-on.** Endpoints return `403 Ads add-on required` without it.

## Endpoints

| Method | Path | Purpose |
|--------|------|---------|
| GET    | `/v1/ads` | List ads with metrics (paginated) |
| GET    | `/v1/ads/{adId}` | Get a single ad with creative, targeting, status, metrics |
| PUT    | `/v1/ads/{adId}` | Update status, budget, name, or targeting (targeting = Meta only) |
| DELETE | `/v1/ads/{adId}` | Cancel an ad |
| GET    | `/v1/ads/{adId}/analytics` | Summary + daily timeline + optional demographic breakdowns |
| GET    | `/v1/ads/campaigns` | List virtual campaigns aggregated from child ads |
| PUT    | `/v1/ads/campaigns/{campaignId}/status` | Pause / resume every ad in a campaign in one call |
| GET    | `/v1/ads/tree` | Nested Campaign > Ad Set > Ad hierarchy with rolled-up metrics |
| GET    | `/v1/ads/accounts` | List platform ad accounts (e.g. Meta `act_123`, Google customer IDs) |
| POST   | `/v1/ads/boost` | Promote an existing published post into a paid campaign |
| POST   | `/v1/ads/create` | Create a standalone ad from scratch (headline, body, link, media) |
| GET    | `/v1/ads/interests` | Search interest-based targeting options |
| GET    | `/v1/ads/audiences` | List custom audiences (Meta, Google, TikTok, Pinterest) |
| POST   | `/v1/ads/audiences` | Create customer_list / website / lookalike audience (Meta) |
| GET    | `/v1/ads/audiences/{audienceId}` | Audience details + fresh Meta data |
| DELETE | `/v1/ads/audiences/{audienceId}` | Delete audience from platform + Zernio |
| POST   | `/v1/ads/audiences/{audienceId}/users` | Upload users to a customer_list audience (SHA-256 hashed server-side) |
| PATCH  | `/v1/ads/campaigns/{campaignId}/negative-keywords` | Add or remove single campaign-level negative keywords (Google) |
| PATCH  | `/v1/ads/accounts/negative-keyword-lists/{listId}/keywords` | Add or remove single keywords in a shared negative keyword list (Google) |
| GET    | `/v1/ads/conversions/destinations` | List pixels, conversion actions and other destinations of an ads account |
| POST   | `/v1/events/forwarding-rules` | Save which destinations receive your events |
| POST   | `/v1/events` | Send conversion events to every saved destination |
| GET    | `/v1/conversion-audit` | Conversion Leak Report: scan a site's ad tracking |

## Listing Ads

```bash
# Defaults to last 90 days
curl "https://zernio.com/api/v1/ads?platform=facebook&status=active&limit=50" \
  -H "Authorization: Bearer YOUR_API_KEY"
```

Query params: `source` (`zernio` = Zernio-created, `all` = include external ads from platform ad managers), `status`, `platform`, `accountId`, `adAccountId`, `profileId`, `campaignId`, `fromDate`, `toDate`, `page`, `limit` (max 500).

## Boost Existing Post

Fastest path to a live ad: boost a published organic post.

```bash
curl -X POST https://zernio.com/api/v1/ads/boost \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "postId": "POST_ID",
    "accountId": "ACCOUNT_ID",
    "adAccountId": "act_123456789",
    "name": "Spring sale boost",
    "goal": "engagement",
    "budget": { "amount": 20, "type": "daily" },
    "currency": "USD",
    "targeting": {
      "ageMin": 18,
      "ageMax": 45,
      "countries": ["US", "CA"]
    }
  }'
```

**Goal compatibility by platform:**

| Goal | Meta | TikTok | LinkedIn | X/Twitter | Pinterest | Google |
|------|:----:|:------:|:--------:|:---------:|:---------:|:------:|
| engagement | yes | yes | yes | yes | yes | yes |
| traffic | yes | yes | yes | yes | yes | yes |
| awareness | yes | yes | yes | yes | yes | yes |
| video_views | yes | yes | yes | yes | yes | yes |
| lead_generation | yes | yes | yes | no | no | no |
| conversions | yes | yes | yes | no | no | no |
| app_promotion | yes | yes | no | yes | no | no |

**Budget minimums:** TikTok $20, Pinterest $5, others $1. Lifetime budgets require `schedule.endDate`.

Provide either `postId` (Zernio post ID) or `platformPostId` (platform's native post ID). Meta-only extras: `bidAmount`, `tracking.pixelId`, `tracking.urlTags`, `specialAdCategories` (`HOUSING`, `EMPLOYMENT`, `CREDIT`, `ISSUES_ELECTIONS_POLITICS`).

## Create Standalone Ad

Creates the full campaign > ad set > ad hierarchy with custom creative.

```bash
curl -X POST https://zernio.com/api/v1/ads/create \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "accountId": "ACCOUNT_ID",
    "adAccountId": "act_123456789",
    "name": "Black Friday campaign",
    "goal": "traffic",
    "budgetAmount": 50,
    "budgetType": "daily",
    "currency": "USD",
    "headline": "50% off everything",
    "body": "This weekend only. Shop now.",
    "linkUrl": "https://example.com/sale",
    "imageUrl": "https://example.com/creative.jpg",
    "callToAction": "SHOP_NOW",
    "countries": ["US"],
    "ageMin": 25,
    "ageMax": 54
  }'
```

**Headline limits:** Meta 255, Google 30, Pinterest 100.
**Body limits:** Google 90, Pinterest 500.
**`callToAction`** (Meta only): `LEARN_MORE`, `SHOP_NOW`, `SIGN_UP`, `BOOK_TRAVEL`, `CONTACT_US`, `DOWNLOAD`, `GET_OFFER`, `GET_QUOTE`, `SUBSCRIBE`, `WATCH_MORE`.

Platform-specific fields:
- **Google Display:** `longHeadline` (max 90), `businessName` (max 25)
- **Google Search:** `campaignType: "search"`, `keywords`, `additionalHeadlines`, `additionalDescriptions` (RSA)
- **Pinterest:** `boardId` (auto-creates if omitted)
- **TikTok:** use `imageUrl` for the video URL

## Update Ad

```bash
curl -X PUT https://zernio.com/api/v1/ads/AD_ID \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "status": "paused",
    "budget": { "amount": 30, "type": "daily" }
  }'
```

`status`, `budget`, `name` and `targeting` are mutable. **Targeting updates are Meta-only**: on other platforms the targeting is set at creation.

Google Search responsive search ads also take `headlines` (3-15), `descriptions` (2-4), `finalUrls`, and the display paths `path1` and `path2` (15 characters each; `path2` needs `path1`; an empty string clears one).

## Google Negative Keywords

Edit single negatives without resending the whole set. `remove` wins over `add` for the same keyword, existing keywords are not added twice, and bare strings mean broad match:

```bash
curl -X PATCH https://zernio.com/api/v1/ads/campaigns/CAMPAIGN_ID/negative-keywords \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{ "accountId": "ACCOUNT_ID", "add": ["free"], "remove": [{ "text": "jobs", "matchType": "phrase" }] }'
```

The same `add` / `remove` body on `PATCH /v1/ads/accounts/negative-keyword-lists/{listId}/keywords` edits a shared list, which changes every campaign using it. `PUT` on either path replaces the full set.

## Campaign Controls

Cascading status update in one platform call (not per-ad):

```bash
curl -X PUT https://zernio.com/api/v1/ads/campaigns/CAMPAIGN_ID/status \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{ "status": "paused", "platform": "facebook" }'
```

Response includes `updated`, `skipped`, `skippedReasons`. Ads in terminal states (`rejected`, `completed`, `cancelled`) are auto-skipped.

The `/v1/ads/tree` endpoint gives the full Campaign > Ad Set > Ad hierarchy with metrics rolled up at every level. Ads without a campaign/ad set ID are grouped into `Ungrouped` buckets.

## Analytics

```bash
curl "https://zernio.com/api/v1/ads/AD_ID/analytics?fromDate=2026-04-01&toDate=2026-04-30&breakdowns=age,gender,country" \
  -H "Authorization: Bearer YOUR_API_KEY"
```

Returns `analytics.summary`, `analytics.daily[]` (per-day metrics), and `analytics.breakdowns` (Meta and TikTok only).

**Supported breakdowns:**
- Meta: `age`, `gender`, `country`, `publisher_platform`, `device_platform`, `region`
- TikTok: `gender`, `age`, `country_code`, `platform`, `ac`, `language`

Max 90-day range; defaults to last 90 days.

## Targeting Interests

Interest IDs are platform-specific; always look them up per ad account before passing to boost/create/update.

```bash
curl "https://zernio.com/api/v1/ads/interests?q=fitness&accountId=ACCOUNT_ID" \
  -H "Authorization: Bearer YOUR_API_KEY"
```

Pass the returned interest objects (with both `id` and `name`) into `targeting.interests` on boost/update, or `interests` on create.

## Custom Audiences (Meta, Google, TikTok, Pinterest)

### Create (Meta only)

```bash
# Customer list
curl -X POST https://zernio.com/api/v1/ads/audiences \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{
    "accountId": "ACCOUNT_ID",
    "adAccountId": "act_123456789",
    "name": "Past customers 2025",
    "type": "customer_list"
  }'

# Website retargeting
# requires pixelId + retentionDays (1-180)

# Lookalike
# requires sourceAudienceId + country (2-letter) + ratio (0.01-0.20)
```

### Upload Users

```bash
curl -X POST https://zernio.com/api/v1/ads/audiences/AUDIENCE_ID/users \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{
    "users": [
      { "email": "alice@example.com" },
      { "phone": "+14155551234" },
      { "email": "bob@example.com", "phone": "+14155555678" }
    ]
  }'
```

Each user needs at least `email` or `phone`. PII is SHA-256 hashed server-side. Max 10,000 users per request.

## Conversions API

`POST /v1/events` sends server-side conversions to 18 destinations: the 9 ad networks plus Snapchat, The Trade Desk, Amazon Ads, AppLovin, GA4, Klaviyo, Attentive, Impact and Awin. Each event is formatted and hashed the way each platform needs.

```bash
# 1. Find the destination (a Meta pixel, a Google conversion action...)
curl "https://zernio.com/api/v1/ads/conversions/destinations?accountId=ACCOUNT_ID" \
  -H "Authorization: Bearer YOUR_API_KEY"

# 2. Save a forwarding rule, once per destination
curl -X POST https://zernio.com/api/v1/events/forwarding-rules \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{ "profileId": "PROFILE_ID", "accountId": "ACCOUNT_ID", "destinationId": "123456789012345" }'

# 3. Send events: each one reaches every saved destination
curl -X POST https://zernio.com/api/v1/events \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{
    "profileId": "PROFILE_ID",
    "events": [{
      "eventName": "Purchase",
      "eventTime": 1713600000,
      "eventId": "order_42",
      "value": 49.99,
      "currency": "USD",
      "user": { "email": "alice@example.com", "country": "US" },
      "consent": { "adStorage": "GRANTED", "adUserData": "GRANTED", "adPersonalization": "GRANTED" }
    }]
  }'
```

- **Required:** `eventName`, `eventTime` (unix **seconds**), `eventId`, `user`. Up to 1,000 events per request.
- **Hashing:** send plain identifiers; Zernio hashes them per platform.
- **Dedup:** `eventId` is unique per profile. Resending it replaces the stored event, and platforms use it to drop the pixel's copy.
- **Click ids** (`fbc`, `gclid`, `ttclid`...) go in `user.clickIds` and tie the event to its campaign.
- **Consent:** `consent` carries `adStorage`, `adUserData`, `adPersonalization` and `analyticsStorage` (`GRANTED` / `DENIED`). On profiles created since 2026-11-22, for a visitor in the EEA, UK or Switzerland a missing signal counts as denied: Google Ads still gets the event with the denied signals, GA4 only with `analyticsStorage` granted, and every other destination only with `adUserData` granted and no ad signal denied. An event left out is still stored, and its send shows `dropped` with `consent_denied` or `consent_missing`.
- **Test:** `testCode` routes events to each platform's test mode; test events are not stored.

`POST /v1/ads/conversions` (one destination per call) is legacy but keeps working. `GET /v1/accounts/{accountId}/conversion-destinations` is deprecated: use `GET /v1/ads/conversions/destinations`.

Docs: https://docs.zernio.com/products/conversions-api

## Conversion Leak Report

A free scan of a site's ad tracking: which pixels fire, which wait for consent, which keep sending after Reject, and which networks also get server-side conversions.

- Web: [is-bleeding.com](https://is-bleeding.com). Terminal: `npx is-bleeding yourshop.com` (`--prompt` prints a fix prompt for a coding agent, `--json` the raw response).
- `GET /v1/conversion-audit?url=yourshop.com` scans (the API key is optional); `GET /v1/conversion-audit/latest` reads a host's stored report without scanning again.
- `GET /v1/conversion-audit/meta-audit` reads what Meta Events Manager recorded for a connected pixel over the last 7 days (server share, Event Match Quality, dedup coverage).
- `POST /v1/conversion-audit/monitors` with `url` rescans the homepage daily and emails when tracking breaks (up to 25 domains; `GET` lists them).

To fix what it finds, follow [fix-conversion-tracking.md](fix-conversion-tracking.md).

## Ad Account Prerequisites

Some platforms need a separate ads connection before their ad endpoints work; otherwise `GET /v1/ads/accounts` returns `422 Platform ads connection required`:

- **Meta, LinkedIn, Pinterest:** `GET /v1/connect/{platform}/ads` reuses the posting account's token when it can.
- **TikTok, X:** `GET /v1/connect/tiktok/ads` or `/v1/connect/twitter/ads`, with their own token (X needs the posting `accountId`).
- **Google, Reddit, Microsoft:** standalone, `GET /v1/connect/googleads/ads`, `/v1/connect/redditads/ads`, `/v1/connect/microsoftads/ads`.
- **OpenAI:** an API key from ChatGPT Ads Manager, `POST /v1/connect/openai-ads/credentials`.

See [connect.md](connect.md).
