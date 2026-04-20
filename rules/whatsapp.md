# WhatsApp Business

WhatsApp-specific endpoints for the Business Cloud API: message templates, Flows, business profile, phone number lifecycle, and group chats. Everyday messaging goes through the unified inbox — see `inbox.md`.

All endpoints require `accountId` (the WhatsApp social account ID) unless noted otherwise.

## Templates

WhatsApp requires pre-approved templates for any message sent outside the 24h customer-service window. Templates are fetched directly from the WhatsApp Cloud API.

| Method | Path | Purpose |
|--------|------|---------|
| GET    | `/v1/whatsapp/templates?accountId=...` | List templates |
| POST   | `/v1/whatsapp/templates` | Create template (custom or from library) |
| GET    | `/v1/whatsapp/templates/{templateName}?accountId=...` | Get a single template |
| PATCH  | `/v1/whatsapp/templates/{templateName}` | Update components |
| DELETE | `/v1/whatsapp/templates/{templateName}?accountId=...` | Delete permanently |

### Create (custom) — requires Meta review

Custom templates go to `PENDING` → `APPROVED` or `REJECTED` (can take up to 24h).

```bash
curl -X POST https://zernio.com/api/v1/whatsapp/templates \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{
    "accountId": "ACCOUNT_ID",
    "name": "order_confirmation",
    "category": "UTILITY",
    "language": "en_US",
    "components": [
      { "type": "body", "text": "Your order {{1}} has been confirmed. Expected delivery: {{2}}",
        "example": { "body_text": [["ORD-12345", "March 31"]] } },
      { "type": "footer", "text": "Thank you for your purchase" },
      { "type": "buttons", "buttons": [ { "type": "quick_reply", "text": "Track Order" } ] }
    ]
  }'
```

`name` must be `^[a-z][a-z0-9_]*$`. `category` is `AUTHENTICATION`, `MARKETING`, or `UTILITY`.

### Create (library) — pre-approved, no wait

Browse Meta's library at business.facebook.com/wa/manage/message-templates/ — names like `appointment_reminder`, `auto_pay_reminder_1`, `address_update`. Omit `components` and pass `library_template_name`:

```bash
curl -X POST https://zernio.com/api/v1/whatsapp/templates \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{
    "accountId": "ACCOUNT_ID",
    "name": "my_appointment_reminder",
    "category": "UTILITY",
    "language": "en_US",
    "library_template_name": "appointment_reminder",
    "library_template_button_inputs": [
      { "type": "url", "url": { "base_url": "https://myapp.com/appointments/{{1}}" } }
    ]
  }'
```

Optional: `library_template_body_inputs` (e.g. `add_contact_number`, `add_learn_more_link`, `code_expiration_minutes`) and `library_template_button_inputs[].type` (`quick_reply`, `url`, `phone_number`).

### Update

Approved templates accept `components` updates only. Name / category / language are immutable.

## Flows

WhatsApp Flows are multi-screen native forms (lead capture, sign-up, surveys, appointments). Lifecycle: `DRAFT` → upload JSON → `PUBLISHED` → `DEPRECATED`.

| Method | Path | Purpose |
|--------|------|---------|
| GET    | `/v1/whatsapp/flows?accountId=...` | List flows (`DRAFT`, `PUBLISHED`, `DEPRECATED`, `BLOCKED`, `THROTTLED`) |
| POST   | `/v1/whatsapp/flows` | Create DRAFT (optionally clone) |
| GET    | `/v1/whatsapp/flows/{flowId}?accountId=...&fields=...` | Flow details + preview URL |
| PATCH  | `/v1/whatsapp/flows/{flowId}` | Update `name`/`categories` on a DRAFT |
| DELETE | `/v1/whatsapp/flows/{flowId}?accountId=...` | Delete DRAFT (irreversible) |
| GET    | `/v1/whatsapp/flows/{flowId}/json` | Get JSON asset + temporary download URL |
| PUT    | `/v1/whatsapp/flows/{flowId}/json` | Upload / update Flow JSON |
| POST   | `/v1/whatsapp/flows/{flowId}/publish` | Publish DRAFT (irreversible, immutable after) |
| POST   | `/v1/whatsapp/flows/{flowId}/deprecate` | Deprecate PUBLISHED |
| POST   | `/v1/whatsapp/flows/send` | Send a published flow as an interactive message |

**Flow categories:** `SIGN_UP`, `SIGN_IN`, `APPOINTMENT_BOOKING`, `LEAD_GENERATION`, `CONTACT_US`, `CUSTOMER_SUPPORT`, `SURVEY`, `OTHER`.

### Create + upload JSON + publish

```bash
# 1. Create DRAFT
curl -X POST https://zernio.com/api/v1/whatsapp/flows \
  -d '{"accountId": "ACC", "name": "lead_capture_form", "categories": ["LEAD_GENERATION"]}'

# 2. Upload Flow JSON (defines screens + components — see Meta docs)
curl -X PUT https://zernio.com/api/v1/whatsapp/flows/FLOW_ID/json \
  -d '{"accountId": "ACC", "flow_json": {"version": "6.0", "screens": [ ... ]}}'

# 3. Publish (irreversible)
curl -X POST https://zernio.com/api/v1/whatsapp/flows/FLOW_ID/publish \
  -d '{"accountId": "ACC"}'
```

`flow_json` accepts either a JSON object or a JSON string. Meta validates on upload — `validation_errors[]` in the response has `error_type`, `message`, `line_start/end`, `column_start/end`. See https://developers.facebook.com/docs/whatsapp/flows/reference/flowjson.

**Published flows are immutable.** To change a published flow, create a new flow (`cloneFlowId` in POST `/v1/whatsapp/flows` copies the JSON).

### Send a flow

```bash
curl -X POST https://zernio.com/api/v1/whatsapp/flows/send \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{
    "accountId": "ACC",
    "to": "+14155551234",
    "flow_id": "FLOW_ID",
    "flow_cta": "Get a Quote",
    "flow_action": "navigate",
    "flow_action_payload": { "screen": "LEAD_FORM" },
    "body": "Fill out this quick form to get a personalized quote."
  }'
```

- `flow_action`: `navigate` (opens `flow_action_payload.screen` directly) or `data_exchange` (posts to your Flow endpoint first).
- `flow_token`: auto-generated UUID if omitted. Use it to correlate responses in webhooks.
- `flow_cta`: button label, max 20 chars.
- `draft: true` lets you test an unpublished DRAFT flow.

Flow responses come back through the `message.received` webhook with `metadata.interactiveType = "nfm_reply"`, `flowResponseJson`, and `flowResponseData`.

## Business Profile

| Method | Path | Purpose |
|--------|------|---------|
| GET    | `/v1/whatsapp/business-profile?accountId=...` | Get profile (about, address, description, email, websites, vertical, picture) |
| POST   | `/v1/whatsapp/business-profile` | Partial update (only provided fields change) |
| POST   | `/v1/whatsapp/business-profile/photo` (multipart) | Upload profile picture |
| GET    | `/v1/whatsapp/business-profile/display-name?accountId=...` | Current display name + Meta review status |
| POST   | `/v1/whatsapp/business-profile/display-name` | Request display name change |

**Limits:** `about` 139 chars, `description` 512 chars, max 2 `websites`, profile picture JPEG/PNG ≤ 5 MB (recommended 640×640).

Display name changes go through Meta review (`PENDING_REVIEW` → `APPROVED` / `DECLINED`, 1–3 business days). Must be 3–512 chars and follow WhatsApp naming guidelines (must represent your business).

## Phone Numbers

Zernio provisions WhatsApp phone numbers via Telnyx + Meta pre-verification. Purchase flow is payment-first — users don't pick a specific number.

| Method | Path | Purpose |
|--------|------|---------|
| GET    | `/v1/whatsapp/phone-numbers` | List (optional `status`, `profileId` filters) |
| POST   | `/v1/whatsapp/phone-numbers/purchase` | Buy a number (first: Stripe Checkout URL; subsequent: provisioned inline) |
| GET    | `/v1/whatsapp/phone-numbers/{phoneNumberId}` | Poll for provisioning status |
| DELETE | `/v1/whatsapp/phone-numbers/{phoneNumberId}` | Release (disconnects account, decrements subscription, releases from Telnyx) |

**Statuses:** `pending_payment`, `provisioning`, `active`, `suspended`, `releasing`, `released`. Listing excludes `released` by default.

**Purchase response:**
```json
// First number — payment required
{ "message": "...", "checkoutUrl": "https://checkout.stripe.com/..." }

// Subsequent numbers — subscription quantity increments, provisioned inline
{ "message": "...", "phoneNumber": { "id": "...", "phoneNumber": "+1...", "status": "provisioning", ... } }
```

Requires a paid plan. Maximum number count is plan-dependent.

## Group Chats

Actual WhatsApp group conversations (not Zernio contact groups).

| Method | Path | Purpose |
|--------|------|---------|
| GET    | `/v1/whatsapp/wa-groups` | List active groups (`after` cursor, `limit` ≤ 1024) |
| POST   | `/v1/whatsapp/wa-groups` | Create group + optional invite link |
| GET    | `/v1/whatsapp/wa-groups/{groupId}` | Info: subject, description, participants, settings |
| POST   | `/v1/whatsapp/wa-groups/{groupId}` | Update `subject`, `description`, `joinApprovalMode` |
| DELETE | `/v1/whatsapp/wa-groups/{groupId}` | Delete group and remove all participants |
| POST   | `/v1/whatsapp/wa-groups/{groupId}/participants` | Add up to 8 phone numbers (E.164) |
| DELETE | `/v1/whatsapp/wa-groups/{groupId}/participants` | Remove participants |
| POST   | `/v1/whatsapp/wa-groups/{groupId}/invite-link` | Rotate invite link (revokes previous) |
| GET    | `/v1/whatsapp/wa-groups/{groupId}/join-requests` | Pending requests (`approval_required` mode only) |
| POST   | `/v1/whatsapp/wa-groups/{groupId}/join-requests` | Approve requests |
| DELETE | `/v1/whatsapp/wa-groups/{groupId}/join-requests` | Reject requests |

**Limits:** `subject` 128 chars, `description` 2048 chars, add up to 8 participants per request.
**`joinApprovalMode`:** `approval_required` (admin approves each join via invite link) or `auto_approve`.
