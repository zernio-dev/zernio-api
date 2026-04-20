# Ads

Create and manage paid ad campaigns across 7 ad networks from the same API used for organic posts.

**Supported ad platforms:** `facebook`, `instagram`, `tiktok`, `linkedin`, `pinterest`, `google`, `twitter`

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
| POST   | `/v1/ads/conversions` | Send conversion events to Meta / Google Conversions API |
| GET    | `/v1/accounts/{accountId}/conversion-destinations` | List pixels (Meta) or conversion actions (Google) |

## Listing Ads

```bash
# Defaults to last 90 days; max 90-day range
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

Only `status`, `budget`, `name`, and `targeting` are mutable. **Targeting updates are Meta-only** — on other platforms the targeting must be set at creation and cannot be modified.

## Campaign Controls

Cascading status update in one platform call (not per-ad):

```bash
curl -X PUT https://zernio.com/api/v1/ads/campaigns/CAMPAIGN_ID/status \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{ "status": "paused", "platform": "facebook" }'
```

Response includes `updated`, `skipped`, `skippedReasons` — ads in terminal states (`rejected`, `completed`, `cancelled`) are auto-skipped.

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

Relay conversion events to Meta (via Graph API) or Google Ads (Data Manager `ingestEvents`). Platform is inferred from `accountId`.

```bash
# List valid destinations first
curl "https://zernio.com/api/v1/accounts/ACCOUNT_ID/conversion-destinations" \
  -H "Authorization: Bearer YOUR_API_KEY"

# Send events
curl -X POST https://zernio.com/api/v1/ads/conversions \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{
    "accountId": "META_ACCOUNT_ID",
    "destinationId": "123456789012345",
    "events": [
      {
        "eventName": "Purchase",
        "eventTime": 1713600000,
        "eventId": "order_42",
        "userData": { "email": "alice@example.com" },
        "customData": { "value": 49.99, "currency": "USD" }
      }
    ]
  }'
```

**destinationId semantics:**
- **Meta:** pixel / dataset ID (e.g. `"123456789012345"`)
- **Google:** full resource name (e.g. `"customers/1234567890/conversionActions/987654321"`)

**Hashing:** all PII (email, phone, names, external IDs) is hashed server-side per each platform's normalization spec (including Google's Gmail-specific dot / plus-suffix stripping). **Send plaintext** — hashing yourself is redundant and will break matching.

**Batching is automatic.** Meta caps at 1,000 events per request and rejects the entire batch if any event is malformed. Google caps at 2,000. Zernio chunks automatically.

**Dedup:** always send a stable `eventId`. Meta uses it to dedupe against pixel events; Google maps it to `transactionId`.

**EEA/UK (Google):** include `consent.adUserData` and `consent.adPersonalization` (`GRANTED` / `DENIED`) under the Feb 2026 restrictions. Meta ignores `consent`.

**Meta test mode:** pass `testCode` ("test_event_code" passthrough). Ignored by Google.

Response exposes `eventsReceived`, `eventsFailed`, `failures[]` (with `eventIndex`, `eventId`, `message`, `code`), and a `traceId` (`fbtrace_id` for Meta, `requestId` for Google). Meta is all-or-nothing per chunk; Google reports request-level success only.

## Ad Account Prerequisites

Some platforms need a separate ads connection before their ad endpoints work — otherwise `GET /v1/ads/accounts` returns `422 Platform ads connection required`:

- **TikTok Ads** — connect via `/v1/connect/tiktokads`
- **X/Twitter Ads** — connect via `/v1/connect/twitterads`
- **Instagram ads** — require a linked Facebook account (Meta inherits this from Facebook ad accounts)

Meta (Facebook), LinkedIn, Pinterest, and Google Ads use the same connection as organic posting.
