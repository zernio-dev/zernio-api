# Contacts & Custom Fields

Zernio contacts are people you message on any DM-capable platform. Each contact can have multiple platform channels (Instagram DM, WhatsApp, Telegram, etc.), tags, and arbitrary custom fields. Broadcasts and sequences target contact lists.

Supported platforms: `instagram`, `facebook`, `telegram`, `twitter`, `bluesky`, `reddit`, `whatsapp`.

## Contacts

| Method | Path | Purpose |
|--------|------|---------|
| GET    | `/v1/contacts` | List with search / filters |
| POST   | `/v1/contacts` | Create (optionally with a channel) |
| GET    | `/v1/contacts/{contactId}` | Contact + all channels |
| PATCH  | `/v1/contacts/{contactId}` | Partial update |
| DELETE | `/v1/contacts/{contactId}` | Permanent delete (removes channels too) |
| GET    | `/v1/contacts/{contactId}/channels` | List channels for a contact |
| POST   | `/v1/contacts/bulk` | Import up to 1,000 contacts at once (skips duplicates) |

### List

```bash
curl "https://zernio.com/api/v1/contacts?profileId=PROFILE_ID&tag=vip&platform=whatsapp&search=anna&limit=100" \
  -H "Authorization: Bearer YOUR_API_KEY"
```

Filters: `profileId` (omit to list across all profiles), `search` (full-text), `tag`, `platform`, `isSubscribed` (`true`/`false`), `limit` (max 200), `skip`. Response includes `filters.tags[]` for building UI facets.

### Create (with channel in one call)

```bash
curl -X POST https://zernio.com/api/v1/contacts \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{
    "profileId": "PROFILE_ID",
    "name": "Anna Smith",
    "email": "anna@example.com",
    "tags": ["vip", "newsletter"],
    "accountId": "WA_ACCOUNT_ID",
    "platform": "whatsapp",
    "platformIdentifier": "+14155551234"
  }'
```

`accountId + platform + platformIdentifier` together create a channel. Omit them to create a contact without any messaging channel yet. Returns `409` on duplicate.

### Bulk import

```bash
curl -X POST https://zernio.com/api/v1/contacts/bulk \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{
    "profileId": "PROFILE_ID",
    "accountId": "WA_ACCOUNT_ID",
    "platform": "whatsapp",
    "contacts": [
      { "name": "Alice", "platformIdentifier": "+14155551234", "tags": ["vip"] },
      { "name": "Bob",   "platformIdentifier": "+14155556789", "email": "bob@example.com" }
    ]
  }'
# -> { "created": 2, "skipped": 0, "errors": [], "total": 2 }
```

Max 1,000 per request. Each contact needs `name` and `platformIdentifier`.

## Custom Fields

Typed fields you can attach to contacts. Supported types: `text`, `number`, `date`, `boolean`, `select`.

| Method | Path | Purpose |
|--------|------|---------|
| GET    | `/v1/custom-fields` | List definitions (optional `profileId` filter) |
| POST   | `/v1/custom-fields` | Create definition |
| PATCH  | `/v1/custom-fields/{fieldId}` | Update `name` / `options` (type is immutable) |
| DELETE | `/v1/custom-fields/{fieldId}` | Delete definition and clear values on all contacts |
| PUT    | `/v1/contacts/{contactId}/fields/{slug}` | Set value on a contact |
| DELETE | `/v1/contacts/{contactId}/fields/{slug}` | Clear value on a contact |

### Define a field

```bash
# Plain text field
curl -X POST https://zernio.com/api/v1/custom-fields \
  -d '{"profileId": "...", "name": "Job Title", "type": "text"}'

# Select (dropdown) — requires options[]
curl -X POST https://zernio.com/api/v1/custom-fields \
  -d '{"profileId": "...", "name": "Tier", "type": "select", "options": ["free", "pro", "enterprise"]}'
```

`slug` is auto-generated from `name` if omitted (e.g. "Job Title" → `job_title`). Returns `409` if the slug collides.

### Set / clear per-contact value

```bash
curl -X PUT https://zernio.com/api/v1/contacts/CONTACT_ID/fields/job_title \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{"value": "Head of Marketing"}'

curl -X DELETE https://zernio.com/api/v1/contacts/CONTACT_ID/fields/job_title \
  -H "Authorization: Bearer YOUR_API_KEY"
```

The `value` shape must match the field's `type` (string for `text`, number for `number`, ISO date for `date`, boolean for `boolean`, one of `options[]` for `select`).

Custom field values appear on contacts as `customFields: { slug: value, ... }` and can drive sequence variable mapping (see `sequences.md`).
