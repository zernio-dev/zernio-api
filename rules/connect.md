# Connect (OAuth) API

## Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/v1/connect/{platform}` | Start OAuth flow |
| `GET` | `/v1/connect/get-connect-url` | Start OAuth flow (generic, supports headless) |
| `POST` | `/v1/connect/callback` | Exchange OAuth code for tokens |
| `GET` | `/v1/connect/pending-data` | Get pending OAuth data (headless mode) |
| `POST` | `/v1/connect/bluesky/credentials` | Connect Bluesky (app password) |
| `GET` | `/v1/connect/telegram` | Generate Telegram access code |
| `POST` | `/v1/connect/telegram` | Direct connect via chat ID |
| `PATCH` | `/v1/connect/telegram` | Poll connection status |

## Platform Selection Endpoints

Some platforms require selecting a page/location/subreddit after OAuth:

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/v1/connect/facebook/pages` | List Facebook pages |
| `POST` | `/v1/connect/facebook/select-page` | Select Facebook page |
| `GET` | `/v1/connect/linkedin/organizations` | List available LinkedIn orgs |
| `POST` | `/v1/connect/linkedin/select-organization` | Select LinkedIn org |
| `GET` | `/v1/connect/google-business/locations` | List GMB locations |
| `POST` | `/v1/connect/google-business/select-location` | Select GMB location |
| `GET` | `/v1/connect/pinterest/select-board` | List Pinterest boards |
| `POST` | `/v1/connect/pinterest/select-board` | Select Pinterest board |
| `GET` | `/v1/connect/reddit/subreddits` | List Reddit subreddits (during connect) |
| `POST` | `/v1/connect/reddit/update-subreddits` | Set default subreddit (during connect) |
| `GET` | `/v1/connect/snapchat/select-profile` | List Snapchat profiles |
| `POST` | `/v1/connect/snapchat/select-profile` | Select Snapchat profile |

## OAuth Flow

### Standard OAuth (most platforms)

```bash
# Get OAuth URL
curl "https://getlate.dev/api/v1/connect/twitter?profileId=PROFILE_ID&redirectUrl=https://yourapp.com/callback" \
  -H "Authorization: Bearer YOUR_API_KEY"
```

Returns `{ "authUrl": "https://twitter.com/oauth/..." }` — redirect user there.

**Query parameters:**
- `profileId` (required) — the Late profile ID
- `redirectUrl` (required) — where to redirect after OAuth completes

### Headless Mode (custom UI)

For apps that want full control over the OAuth UI:

```bash
# Get connect URL with headless mode
curl "https://getlate.dev/api/v1/connect/get-connect-url?profileId=PROFILE_ID&redirectUrl=https://yourapp.com/callback&headless=true" \
  -H "Authorization: Bearer YOUR_API_KEY"
```

Returns `{ "authUrl": "...", "connectToken": "tok_..." }`.

Use `connectToken` in subsequent calls via `X-Connect-Token` header:

```bash
# Get pending OAuth data after user completes consent
curl "https://getlate.dev/api/v1/connect/pending-data?token=PENDING_DATA_TOKEN" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "X-Connect-Token: tok_..."
```

### Bluesky (App Password)

```bash
curl -X POST https://getlate.dev/api/v1/connect/bluesky/credentials \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "profileId": "PROFILE_ID",
    "identifier": "user.bsky.social",
    "password": "xxxx-xxxx-xxxx-xxxx"
  }'
```

### Telegram

**Option 1: Access Code Flow (recommended)**

```bash
# 1. Generate access code
curl "https://getlate.dev/api/v1/connect/telegram?profileId=PROFILE_ID" \
  -H "Authorization: Bearer YOUR_API_KEY"

# Returns: { "code": "LATE-ABC123", "botUsername": "LateScheduleBot", ... }

# 2. User adds bot to channel and sends code to bot
# 3. Poll for connection status
curl -X PATCH "https://getlate.dev/api/v1/connect/telegram?code=LATE-ABC123" \
  -H "Authorization: Bearer YOUR_API_KEY"

# Returns: { "status": "pending" } or { "status": "connected", "account": {...} }
```

**Option 2: Direct Chat ID (power users)**

```bash
curl -X POST https://getlate.dev/api/v1/connect/telegram \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{
    "profileId": "PROFILE_ID",
    "chatId": "-1001234567890"
  }'
```

The Late bot must already be added as admin in your channel/group.

## Reddit OAuth Flow

Reddit requires subreddit selection after connecting:

```bash
# 1. Start OAuth
curl "https://getlate.dev/api/v1/connect/reddit?profileId=PROFILE_ID&redirectUrl=https://yourapp.com/callback" \
  -H "Authorization: Bearer YOUR_API_KEY"
# Returns: { "authUrl": "https://www.reddit.com/api/v1/authorize?..." }

# 2. After user completes OAuth, list available subreddits
curl "https://getlate.dev/api/v1/connect/reddit/subreddits?profileId=PROFILE_ID" \
  -H "Authorization: Bearer YOUR_API_KEY"

# 3. Set default subreddit
curl -X POST "https://getlate.dev/api/v1/connect/reddit/update-subreddits" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"profileId": "PROFILE_ID", "subreddit": "distrilicious"}'
```

You can also manage subreddits post-connect via the Accounts API:
- `GET /v1/accounts/{accountId}/reddit-subreddits`
- `PUT /v1/accounts/{accountId}/reddit-subreddits`

## Supported Platforms

| Platform | Auth Method | Notes |
|----------|-------------|-------|
| Twitter/X | OAuth 2.0 PKCE | Requires code verifier |
| Instagram | OAuth 2.0 | 2-step token exchange |
| Facebook | OAuth 2.0 | Requires page selection |
| LinkedIn | OAuth 2.0 | Optional org selection; headless restructured Jan 2026 |
| TikTok | OAuth 2.0 | UX compliance required |
| YouTube | Google OAuth | access_type=offline |
| Pinterest | OAuth 2.0 | Requires board selection |
| Reddit | OAuth 2.0 | Strict user-agent; requires subreddit selection |
| Bluesky | App password | No OAuth, uses AT Protocol |
| Threads | OAuth 2.0 | Similar to Instagram |
| Google Business | Google OAuth | Requires location selection |
| Telegram | Chat ID | Uses Late's bot |
| Snapchat | OAuth 2.0 | Allowlist-only |
