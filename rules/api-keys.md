# API Keys

Programmatic lifecycle for Bearer tokens. The full key value is returned **once, at creation time** — store it immediately. Subsequent list calls only return a `keyPreview` like `sk_12345678...abcdef01`.

| Method | Path | Purpose |
|--------|------|---------|
| GET    | `/v1/api-keys` | List keys (preview only) |
| POST   | `/v1/api-keys` | Create a new key |
| DELETE | `/v1/api-keys/{keyId}` | Revoke permanently |

## Create

```bash
curl -X POST https://zernio.com/api/v1/api-keys \
  -H "Authorization: Bearer YOUR_MASTER_API_KEY" \
  -d '{
    "name": "Analytics Read-Only",
    "scope": "profiles",
    "profileIds": ["6507a1b2c3d4e5f6a7b8c9d0"],
    "permission": "read",
    "expiresIn": 90
  }'
```

| Field | Values | Default | Notes |
|-------|--------|---------|-------|
| `name` | string | — | **Required.** Display label. |
| `scope` | `full`, `profiles` | `full` | `full` = all profiles; `profiles` = restricted to `profileIds[]`. |
| `profileIds` | array | — | **Required when `scope: profiles`.** |
| `permission` | `read-write`, `read` | `read-write` | `read` limits the key to `GET` requests only. |
| `expiresIn` | integer | — | Days until expiry. Omit for a non-expiring key. |

The response `apiKey.key` field is the full key — save it now, it won't be shown again.

## Delete

```bash
curl -X DELETE https://zernio.com/api/v1/api-keys/KEY_ID \
  -H "Authorization: Bearer YOUR_MASTER_API_KEY"
```

Revocation takes effect immediately. A revoked key cannot be restored.

## Security notes

- Narrow `scope` and `permission` for third-party / automation keys. A read-only, profile-scoped key is safe to hand to a reporting integration.
- Rotate keys periodically by creating a new one, deploying it, then deleting the old one.
- The same key value is never returned a second time — if you lose it, delete and recreate.
