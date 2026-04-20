# Accounts API

## Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/v1/accounts` | List accounts |
| `PUT` | `/v1/accounts/{accountId}` | Update account |
| `DELETE` | `/v1/accounts/{accountId}` | Disconnect account |
| `GET` | `/v1/accounts/health` | Check all accounts health |
| `GET` | `/v1/accounts/{accountId}/health` | Check specific account health |
| `GET` | `/v1/accounts/follower-stats` | Get follower statistics |

## Platform-Specific Account Endpoints

Per-account helpers for managing platform configuration, fetching platform-specific lists, and looking up data you need at post time.

### Facebook / LinkedIn / Pinterest / YouTube / GMB / Reddit / Discord

| Method | Endpoint | Description |
|--------|----------|-------------|
| `PUT` | `/v1/accounts/{accountId}/facebook-page` | Change the selected Facebook Page |
| `GET` | `/v1/accounts/{accountId}/linkedin-organizations` | List LinkedIn organizations the user admins |
| `PUT` | `/v1/accounts/{accountId}/linkedin-organization` | Switch between personal and organization mode |
| `GET` | `/v1/accounts/{accountId}/linkedin-mentions` | Resolve person/org URNs for LinkedIn mentions |
| `GET` | `/v1/accounts/{accountId}/linkedin-aggregate-analytics` | LinkedIn account analytics (see `analytics.md`) |
| `GET` | `/v1/accounts/{accountId}/linkedin-post-analytics` | LinkedIn post analytics |
| `GET` | `/v1/accounts/{accountId}/linkedin-post-reactions` | LinkedIn post reactions breakdown |
| `GET` | `/v1/accounts/{accountId}/pinterest-boards` | List Pinterest boards |
| `PUT` | `/v1/accounts/{accountId}/pinterest-boards` | Set default Pinterest board |
| `GET` | `/v1/accounts/{accountId}/youtube-playlists` | List YouTube playlists (for adding published videos) |
| `GET` | `/v1/accounts/{accountId}/gmb-locations` | List GBP locations for the connected account |
| `GET` | `/v1/accounts/{accountId}/gmb-reviews` | Google Business reviews (see `gmb.md` for the full GBP surface) |
| `GET` | `/v1/accounts/{accountId}/reddit-subreddits` | List user's subreddits |
| `PUT` | `/v1/accounts/{accountId}/reddit-subreddits` | Set default subreddit |
| `GET` | `/v1/accounts/{accountId}/reddit-flairs?subreddit=...` | List flairs available in a subreddit |
| `GET` | `/v1/accounts/{accountId}/tiktok/creator-info` | TikTok Creator Info (privacy levels, duet/stitch permissions, max duration) — required before posting |
| `GET` | `/v1/accounts/{accountId}/discord-settings` | Get Discord webhook identity + connected channel |
| `PATCH` | `/v1/accounts/{accountId}/discord-settings` | Update webhook identity or switch to a different channel in the same guild |
| `GET` | `/v1/accounts/{accountId}/discord-channels` | List available Discord text / announcement / forum channels |
| `GET` | `/v1/accounts/{accountId}/conversion-destinations` | List Meta pixels / Google conversion actions available for the Conversions API (see `ads.md`) |

### TikTok Creator Info

TikTok requires fetching creator-specific settings before posting — use this to populate `platformSpecificData` correctly (allowed privacy levels, whether the creator disabled duet/stitch, max video duration).

```bash
curl "https://zernio.com/api/v1/accounts/ACCOUNT_ID/tiktok/creator-info" \
  -H "Authorization: Bearer YOUR_API_KEY"
```

### Discord

```bash
# Current identity + channel
curl "https://zernio.com/api/v1/accounts/ACCOUNT_ID/discord-settings" \
  -H "Authorization: Bearer YOUR_API_KEY"

# Change webhook display name / avatar (account-level defaults)
curl -X PATCH https://zernio.com/api/v1/accounts/ACCOUNT_ID/discord-settings \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{"accountId": "ACCOUNT_ID", "webhookUsername": "My Brand", "webhookAvatarUrl": "https://example.com/logo.png"}'

# Switch connected channel (same guild)
curl -X PATCH https://zernio.com/api/v1/accounts/ACCOUNT_ID/discord-settings \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{"accountId": "ACCOUNT_ID", "channelId": "9999999999999999999"}'

# List channels to pick from
curl "https://zernio.com/api/v1/accounts/ACCOUNT_ID/discord-channels" \
  -H "Authorization: Bearer YOUR_API_KEY"
```

`webhookUsername` must be 1-80 chars and cannot contain "clyde" or "discord". Channel type must be text (0), announcement (5), or forum (15).

### Chat config

Per-account configuration for bot-like surfaces (Facebook Messenger persistent menu, Instagram ice breakers, Telegram commands) lives in `inbox.md` — see the "Chat config" section.

## List Accounts

```bash
curl "https://zernio.com/api/v1/accounts?profileId=PROFILE_ID" \
  -H "Authorization: Bearer YOUR_API_KEY"
```

## Account Health Check

```bash
curl "https://zernio.com/api/v1/accounts/health" \
  -H "Authorization: Bearer YOUR_API_KEY"
```

Response indicates if tokens are valid or need reconnection.

## Account Groups

See `account-groups.md` for the full surface (list / create / update / delete).
