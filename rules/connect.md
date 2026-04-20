# Connect (OAuth) API

## Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/v1/connect/{platform}` | Start OAuth flow (returns `authUrl`) |
| `POST` | `/v1/connect/{platform}` | Complete OAuth callback (exchange `code` + `state`) |
| `GET` | `/v1/connect/{platform}/ads` | Connect ads for a platform (same-token, separate-token, or standalone flow) |
| `POST` | `/v1/connect/bluesky/credentials` | Connect Bluesky (app password) |
| `POST` | `/v1/connect/whatsapp/credentials` | Connect WhatsApp Business |
| `GET` | `/v1/connect/telegram` | Generate Telegram access code |
| `POST` | `/v1/connect/telegram` | Direct connect via chat ID |
| `PATCH` | `/v1/connect/telegram` | Poll connection status |
| `GET` | `/v1/connect/pending-data` | Exchange a `pendingDataToken` for pending OAuth data (headless flows) |

## Platform Selection Endpoints

Some platforms require selecting a page/location after OAuth:

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/v1/connect/facebook/select-page` | List Facebook pages |
| `POST` | `/v1/connect/facebook/select-page` | Select Facebook page |
| `GET` | `/v1/connect/linkedin/organizations` | List available LinkedIn orgs |
| `POST` | `/v1/connect/linkedin/select-organization` | Select LinkedIn org |
| `GET` | `/v1/connect/googlebusiness/locations` | List GMB locations |
| `POST` | `/v1/connect/googlebusiness/select-location` | Select GMB location |
| `GET` | `/v1/connect/pinterest/select-board` | List Pinterest boards |
| `POST` | `/v1/connect/pinterest/select-board` | Select Pinterest board |
| `GET` | `/v1/connect/snapchat/select-profile` | List Snapchat profiles |
| `POST` | `/v1/connect/snapchat/select-profile` | Select Snapchat profile |

## OAuth Flow

### Standard OAuth (most platforms)

```bash
# Get OAuth URL
curl "https://zernio.com/api/v1/connect/twitter?profileId=PROFILE_ID&callbackUrl=https://yourapp.com/callback" \
  -H "Authorization: Bearer YOUR_API_KEY"
```

Returns `{ "url": "https://twitter.com/oauth/..." }` - redirect user there.

### Bluesky (App Password)

```bash
curl -X POST https://zernio.com/api/v1/connect/bluesky/credentials \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "profileId": "PROFILE_ID",
    "identifier": "user.bsky.social",
    "appPassword": "xxxx-xxxx-xxxx-xxxx"
  }'
```

### Telegram

**Option 1: Access Code Flow (recommended)**

```bash
# 1. Generate access code
curl "https://zernio.com/api/v1/connect/telegram?profileId=PROFILE_ID" \
  -H "Authorization: Bearer YOUR_API_KEY"

# Returns: { "code": "LATE-ABC123", "botUsername": "LateScheduleBot", ... }

# 2. User adds bot to channel and sends code to bot
# 3. Poll for connection status
curl -X PATCH "https://zernio.com/api/v1/connect/telegram?code=LATE-ABC123" \
  -H "Authorization: Bearer YOUR_API_KEY"

# Returns: { "status": "pending" } or { "status": "connected", "account": {...} }
```

**Option 2: Direct Chat ID (power users)**

```bash
curl -X POST https://zernio.com/api/v1/connect/telegram \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "profileId": "PROFILE_ID",
    "chatId": "-1001234567890"
  }'
```

The Zernio bot must already be added as admin in your channel/group.

## Connecting Ads

`GET /v1/connect/{platform}/ads` creates a dedicated ads SocialAccount for the given platform. Behavior depends on how the platform handles ad tokens:

- **Same-token platforms** (`facebook`, `instagram`, `linkedin`, `pinterest`): copies the OAuth token from the existing posting account and creates an ads account (`metaads`, `linkedinads`, `pinterestads`). No extra OAuth.
- **Separate-token platforms** (`tiktok`, `twitter`): starts a platform marketing-API OAuth flow and creates an ads account (`tiktokads`, `xads`) with its own token. **Requires `accountId`** of the existing posting account.
- **Standalone platforms** (`googleads`): starts Google Ads OAuth and creates a standalone ads account with no parent.

```bash
# Same-token — Meta inherits from the posting account
curl "https://zernio.com/api/v1/connect/instagram/ads?profileId=PROFILE_ID" \
  -H "Authorization: Bearer YOUR_API_KEY"

# Separate-token — TikTok Ads needs the existing posting accountId
curl "https://zernio.com/api/v1/connect/tiktok/ads?profileId=PROFILE_ID&accountId=POSTING_ACC" \
  -H "Authorization: Bearer YOUR_API_KEY"

# Standalone
curl "https://zernio.com/api/v1/connect/googleads/ads?profileId=PROFILE_ID" \
  -H "Authorization: Bearer YOUR_API_KEY"
```

Response is either `{ authUrl, state }` (redirect the user) or `{ alreadyConnected: true, accountId, ... }` if an ads account already exists. Returns `403 Ads add-on required` without the add-on. Ads accounts appear in `GET /v1/accounts` with the ads platform values (`metaads`, `tiktokads`, `xads`, `googleads`, `linkedinads`, `pinterestads`). See `ads.md` for using them.

## Headless Flows

Pass `headless=true` to `/v1/connect/{platform}` and Zernio redirects the user to your `redirect_url` with OAuth data params instead of showing Zernio's default selection UI. Use this to build a custom connect experience.

Typical headless flow for platforms that need a selection step (Facebook pages, LinkedIn orgs, GBP locations, Pinterest boards, Snapchat profiles):

1. Redirect user to `authUrl` from `GET /v1/connect/{platform}?headless=true`.
2. After OAuth, user lands back at your `redirect_url` with `pendingDataToken=...&step=select_page` (or similar).
3. Call `GET /v1/connect/pending-data?token=...` if you need the raw OAuth data. **One-time use, expires in 10 minutes.** No auth required.
4. List the available entities (pages, orgs, locations, boards, profiles) via the matching `GET /v1/connect/{platform}/select-*` endpoint.
5. Complete by calling the `POST /v1/connect/{platform}/select-*` with `pendingDataToken` (preferred) or legacy `tempToken`.

When connecting via an API key, pass the token in the `X-Connect-Token` header on these headless endpoints.

## Supported Platforms

| Platform | Auth Method | Notes |
|----------|-------------|-------|
| Twitter/X | OAuth 2.0 PKCE | Requires code verifier |
| Instagram | OAuth 2.0 | 2-step token exchange |
| Facebook | OAuth 2.0 | Requires page selection |
| LinkedIn | OAuth 2.0 | Optional org selection |
| TikTok | OAuth 2.0 | UX compliance required |
| YouTube | Google OAuth | access_type=offline |
| Pinterest | OAuth 2.0 | Requires board selection |
| Reddit | OAuth 2.0 | Strict user-agent |
| Bluesky | App password | No OAuth, uses AT Protocol |
| Threads | OAuth 2.0 | Similar to Instagram |
| Google Business | Google OAuth | Requires location selection |
| Telegram | Chat ID | Uses Late's bot |
| Snapchat | OAuth 2.0 | Allowlist-only |
| Discord | OAuth 2.0 | Bot webhook per channel; per-account settings via `/v1/accounts/{id}/discord-settings` |
| WhatsApp | Meta Cloud API | Credentials flow via `/v1/connect/whatsapp/credentials` |
