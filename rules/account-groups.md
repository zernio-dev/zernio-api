# Account Groups

Group social accounts for bulk operations in your UI (e.g. "Marketing Accounts", "Personal Brand"). Groups are a thin organizational layer — they don't change account behavior, they just label sets of account IDs.

| Method | Path | Purpose |
|--------|------|---------|
| GET    | `/v1/account-groups` | List groups (all users, no filters) |
| POST   | `/v1/account-groups` | Create (`name` + `accountIds[]`) |
| PUT    | `/v1/account-groups/{groupId}` | Rename or change `accountIds` |
| DELETE | `/v1/account-groups/{groupId}` | Delete group (accounts unaffected) |

Group names must be unique per user — POST and PUT return `409` on collision.

```bash
curl -X POST https://zernio.com/api/v1/account-groups \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{
    "name": "Marketing Accounts",
    "accountIds": ["64e1f0a9e2b5af0012ab34cd", "64e1f0a9e2b5af0012ab34ce"]
  }'
```

The `/v1/account-groups/{groupId}` PUT accepts partial updates — send only `name`, only `accountIds`, or both.
