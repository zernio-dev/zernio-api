# Automations, Sequences & Comment Automations

`/v1/automations` creates and manages every kind of automation in one shape: a `trigger`, what it checks, the `steps` it takes and its `limits`. The older `/v1/sequences` and `/v1/comment-automations` routes below keep working on the same automations.

## Automations (`/v1/automations`)

| Kind | `trigger.type` |
|------|----------------|
| Event automation (a post failed, an order came in...) | The event, e.g. `post.failed` |
| Scheduled check of recent posts (e.g. boost winners) | `schedule` with `subjects.each: "post"` |
| Ad rule (campaigns, ad sets, ads, keywords, search terms) | `schedule` with `subjects.each`: `campaign`, `adset`, `ad`, `keyword` or `search_term` |
| Sequence | `enrollment` |
| Comment to DM | `comment.keyword`, `comment.live`, `story.reply`, `story.mention` |
| Flow (`kind: "workflow"`, the workflow builder's graph) | Read from the graph's trigger node |

```bash
curl -X POST https://zernio.com/api/v1/automations \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{
    "name": "Failed Instagram posts to Slack",
    "trigger": {
      "type": "post.failed",
      "profileId": "PROFILE_ID",
      "filter": { "field": "platform", "op": "equals", "value": "instagram" }
    },
    "steps": [{ "type": "notify.send", "to": [{ "accountId": "SLACK_ACCOUNT_ID", "channelId": "C0123456789" }] }],
    "limits": { "cooldown": "1h" }
  }'
```

- `GET /v1/automations/catalog` lists every event, fact and step the API accepts.
- `GET /v1/automations?kind=sequence,comment` lists by kind (`workflow` for flows). Reading a comment-to-DM automation returns it in the shape `POST` and `PATCH` take: the easiest way to write one.
- `PATCH /v1/automations/{automationId}` takes the fields you change, with the `version` you read as `If-Match`. `POST .../activate` and `.../pause` switch it.
- Ad rules: `POST /v1/automations/backtest` replays a draft over the last 30 days; an `approval.request` step makes each change wait for a team member.
- A refused automation answers `400` with each problem in `details.problems`.
- Sequences and comment to DM need the inbox add-on; ad rules need the Ads add-on to create or edit.

Docs: https://docs.zernio.com/guides/automations

## Sequences

Supported platforms: `instagram`, `facebook`, `telegram`, `twitter`, `bluesky`, `reddit`, `whatsapp`.

Lifecycle: `draft` → `active` (running) ↔ `paused`. Enrollments move through `active` → `completed` / `exited` / `paused`.

| Method | Path | Purpose |
|--------|------|---------|
| GET    | `/v1/sequences` | List with enrollment stats |
| POST   | `/v1/sequences` | Create |
| GET    | `/v1/sequences/{sequenceId}` | Full details with steps |
| PATCH  | `/v1/sequences/{sequenceId}` | Update (can edit active sequences without pausing) |
| DELETE | `/v1/sequences/{sequenceId}` | Delete (stops active enrollments) |
| POST   | `/v1/sequences/{sequenceId}/activate` | Draft / paused → active |
| POST   | `/v1/sequences/{sequenceId}/pause` | Active → paused (enrollments stop, resume on reactivate) |
| POST   | `/v1/sequences/{sequenceId}/enroll` | Enroll contacts |
| DELETE | `/v1/sequences/{sequenceId}/enroll/{contactId}` | Unenroll |
| GET    | `/v1/sequences/{sequenceId}/enrollments` | List enrollments with progress |

### Create

```bash
curl -X POST https://zernio.com/api/v1/sequences \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{
    "profileId": "PROFILE_ID",
    "accountId": "WA_ACCOUNT_ID",
    "platform": "whatsapp",
    "name": "Onboarding 3-day drip",
    "exitOnReply": true,
    "exitOnUnsubscribe": true,
    "steps": [
      {
        "order": 1,
        "delayMinutes": 0,
        "template": {
          "name": "welcome_msg",
          "language": "en_US",
          "variableMapping": {
            "1": { "field": "name" }
          }
        }
      },
      {
        "order": 2,
        "delayMinutes": 1440,
        "message": { "text": "Any questions after your first day? Reply here." }
      },
      {
        "order": 3,
        "delayMinutes": 2880,
        "template": { "name": "day_3_followup", "language": "en_US" }
      }
    ]
  }'
```

**Step shape:**

- `order` (int) — step position.
- `delayMinutes` (int) — wait time from enrollment (step 1) or from previous step (subsequent steps).
- `message.text` — plain message (non-WhatsApp platforms).
- `template` — WhatsApp template with `name`, `language`, and optional `variableMapping`.

**Template variable mapping** (WhatsApp):
```json
"variableMapping": {
  "1": { "field": "name" },
  "2": { "field": "custom", "customValue": "May promo" },
  "3": { "field": "email" }
}
```
Keys are positional (`"1"`, `"2"`…). `field` is one of `name`, `phone`, `email`, `company`, `custom`. When `custom`, provide a static `customValue`. At send time Zernio resolves each variable from the enrolled contact.

**Exit conditions** (defaults both true):
- `exitOnReply: true` — contact exits as soon as they reply on that channel.
- `exitOnUnsubscribe: true` — contact exits when marked unsubscribed.

### Enrolling contacts

```bash
curl -X POST https://zernio.com/api/v1/sequences/SEQ_ID/enroll \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{"contactIds": ["CID1", "CID2", "CID3"]}'
# -> { "enrolled": 2, "skipped": 1 }
```

Already-enrolled contacts are skipped, as are contacts without a channel for the sequence's platform. `channelIds` is optional — auto-detected when omitted.

Unenroll one contact:
```bash
curl -X DELETE https://zernio.com/api/v1/sequences/SEQ_ID/enroll/CONTACT_ID \
  -H "Authorization: Bearer YOUR_API_KEY"
```

### Listing enrollments

```bash
curl "https://zernio.com/api/v1/sequences/SEQ_ID/enrollments?status=active&limit=50" \
  -H "Authorization: Bearer YOUR_API_KEY"
```

Returns per-enrollment `currentStepIndex`, `nextStepAt`, `stepsSent`, `lastStepSentAt`, and `exitReason` (populated on `exited`). Filter by `status`: `active`, `completed`, `exited`, `paused`.

## Comment Automations (IG + FB only)

Keyword-triggered DM automations on specific Instagram or Facebook posts. When someone comments with a matching keyword, Zernio sends them an automatic DM and optionally replies publicly.

| Method | Path | Purpose |
|--------|------|---------|
| GET    | `/v1/comment-automations` | List (optional `profileId` filter) with stats |
| POST   | `/v1/comment-automations` | Create |
| GET    | `/v1/comment-automations/{automationId}` | Details + recent trigger logs |
| PATCH  | `/v1/comment-automations/{automationId}` | Update fields / toggle active |
| DELETE | `/v1/comment-automations/{automationId}` | Delete (removes all logs) |
| GET    | `/v1/comment-automations/{automationId}/logs` | Paginated full trigger log |

### Create

```bash
curl -X POST https://zernio.com/api/v1/comment-automations \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{
    "profileId": "PROFILE_ID",
    "accountId": "IG_ACCOUNT_ID",
    "platformPostId": "17895695665740000",
    "name": "Launch giveaway DM",
    "keywords": ["GIVEAWAY", "me"],
    "matchMode": "contains",
    "dmMessage": "Thanks for entering! Here\u2019s your link: https://example.com/giveaway",
    "commentReply": "DM sent \u2728"
  }'
```

- `keywords: []` (empty array) means **any comment triggers** — use sparingly.
- `matchMode`: `contains` (default) or `exact`.
- `commentReply` is optional; post the public reply alongside the DM.
- Only **one active automation per post** (returns `409` otherwise).

### Logs

```bash
curl "https://zernio.com/api/v1/comment-automations/AID/logs?status=failed&limit=50" \
  -H "Authorization: Bearer YOUR_API_KEY"
```

Each log entry: `commentId`, `commenterId`, `commenterName`, `commentText`, `status` (`sent`/`failed`/`skipped`), `error` (populated on failure), `createdAt`. Useful for debugging keyword miss rates and DM delivery issues.
