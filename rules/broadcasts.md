# Broadcasts

Send a single message to many contacts at once (newsletters, announcements, WhatsApp template blasts). Broadcasts support targeting by tags / subscription, text + attachments on Meta/Telegram/etc., and pre-approved templates on WhatsApp.

Supported platforms: `instagram`, `facebook`, `telegram`, `twitter`, `bluesky`, `reddit`, `whatsapp`.

Lifecycle: `draft` → (add recipients) → `scheduled` or `sending` → `completed` / `failed` / `cancelled`.

## Endpoints

| Method | Path | Purpose |
|--------|------|---------|
| GET    | `/v1/broadcasts` | List with delivery stats |
| POST   | `/v1/broadcasts` | Create draft |
| GET    | `/v1/broadcasts/{broadcastId}` | Full config + stats |
| PATCH  | `/v1/broadcasts/{broadcastId}` | Update (draft only) |
| DELETE | `/v1/broadcasts/{broadcastId}` | Delete (draft only) |
| GET    | `/v1/broadcasts/{broadcastId}/recipients` | List recipients with individual delivery status |
| POST   | `/v1/broadcasts/{broadcastId}/recipients` | Add recipients (by contact IDs, phone numbers, or segment) |
| POST   | `/v1/broadcasts/{broadcastId}/send` | Start sending immediately |
| POST   | `/v1/broadcasts/{broadcastId}/schedule` | Schedule for a future `scheduledAt` |
| POST   | `/v1/broadcasts/{broadcastId}/cancel` | Cancel scheduled or in-progress broadcast |

## Typical flow

```bash
# 1. Create draft (Telegram example — text + segmentFilters for auto-recipient)
curl -X POST https://zernio.com/api/v1/broadcasts \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{
    "profileId": "PROFILE_ID",
    "accountId": "TELEGRAM_ACCOUNT_ID",
    "platform": "telegram",
    "name": "Weekly digest",
    "message": { "text": "Here are this week\u2019s updates..." },
    "segmentFilters": { "tags": ["newsletter"], "isSubscribed": true }
  }'

# 2. Add recipients from the segment (or by explicit IDs / phone numbers)
curl -X POST https://zernio.com/api/v1/broadcasts/BID/recipients \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{"useSegment": true}'

# 3a. Send now
curl -X POST https://zernio.com/api/v1/broadcasts/BID/send \
  -H "Authorization: Bearer YOUR_API_KEY"

# 3b. Or schedule
curl -X POST https://zernio.com/api/v1/broadcasts/BID/schedule \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{"scheduledAt": "2026-05-01T09:00:00Z"}'
```

## WhatsApp broadcasts

WhatsApp requires a pre-approved template — set `template` instead of (or alongside) `message`. Variable values come from each recipient's contact fields at send time.

```json
{
  "profileId": "...",
  "accountId": "WA_ACCOUNT_ID",
  "platform": "whatsapp",
  "name": "May sale",
  "template": {
    "name": "may_sale_launch",
    "language": "en_US",
    "components": [
      { "type": "body", "parameters": [{ "type": "text", "text": "{{contact.name}}" }] }
    ]
  }
}
```

For variable mapping by contact field (like sequences do), build the components payload on your side before creating the broadcast. Create templates via `/v1/whatsapp/templates` (see `whatsapp.md`).

## Adding recipients

Three ways (can mix in one request):

- `contactIds: []` — specific contacts you already have in Zernio
- `phones: []` — raw phone numbers for WhatsApp / Telegram (auto-creates contacts if missing)
- `useSegment: true` — auto-populate from the broadcast's own `segmentFilters`

Response returns `added` / `skipped` counts. Contacts are skipped when already on the recipient list or when no matching channel exists for the broadcast's platform.

## Listing recipients

Per-recipient delivery status with `pending`, `sent`, `delivered`, `read`, `failed`:

```bash
curl "https://zernio.com/api/v1/broadcasts/BID/recipients?status=failed&limit=50" \
  -H "Authorization: Bearer YOUR_API_KEY"
```

Each recipient has `messageId`, timestamps (`sentAt`, `deliveredAt`, `readAt`), and `error` when failed — useful for retry / reporting dashboards.
