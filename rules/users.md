# Users & Team Invites

Workspace user directory and invite-link generation.

## Users

| Method | Path | Purpose |
|--------|------|---------|
| GET    | `/v1/users` | List all users in the workspace + `currentUserId` of the caller |
| GET    | `/v1/users/{userId}` | Single user details |

```bash
curl https://zernio.com/api/v1/users \
  -H "Authorization: Bearer YOUR_API_KEY"
```

Each user has `role` (e.g. `owner`, `member`), `isRoot`, `profileAccess[]` (either `["all"]` or an array of profile IDs), and timestamps.

## Invites

Generate a one-time, 7-day invite link to grant access to profiles.

| Method | Path | Purpose |
|--------|------|---------|
| POST   | `/v1/invite/tokens` | Create invite token + URL |

```bash
# Invite with access to specific profiles
curl -X POST https://zernio.com/api/v1/invite/tokens \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{
    "scope": "profiles",
    "profileIds": ["64f0a1b2c3d4e5f6a7b8c9d0", "64f0a1b2c3d4e5f6a7b8c9d1"]
  }'

# Invite with access to all profiles
# {"scope": "all"}
```

Response:
```json
{
  "token": "inv_abc123def456ghi789",
  "scope": "profiles",
  "invitedProfileIds": ["..."],
  "expiresAt": "2024-11-08T10:30:00Z",
  "inviteUrl": "https://zernio.com/invite/inv_abc123def456ghi789"
}
```

**Constraints:**
- `scope: profiles` requires a non-empty `profileIds[]`. All IDs must be owned by the authenticated user (returns `403` otherwise).
- Invites are **single-use** and expire after 7 days.
- Share `inviteUrl` with the invitee — they'll sign up / log in at that link and the profile access is attached automatically.
