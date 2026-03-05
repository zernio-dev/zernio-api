# Profiles API

## Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| `POST` | `/v1/profiles` | Create profile |
| `GET` | `/v1/profiles` | List profiles |
| `GET` | `/v1/profiles/{id}` | Get profile |
| `PATCH` | `/v1/profiles/{id}` | Update profile |
| `DELETE` | `/v1/profiles/{id}` | Delete profile |

## Create Profile

```bash
curl -X POST https://getlate.dev/api/v1/profiles \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"name": "My Brand", "description": "Optional description", "color": "#3B82F6"}'
```

**Request body:**
- `name` (required) — profile name
- `description` (optional) — profile description
- `color` (optional) — hex color for UI display

## List Profiles

```bash
curl "https://getlate.dev/api/v1/profiles" \
  -H "Authorization: Bearer YOUR_API_KEY"
```

Returns profiles sorted by creation date.

## Get Profile

```bash
curl "https://getlate.dev/api/v1/profiles/PROFILE_ID" \
  -H "Authorization: Bearer YOUR_API_KEY"
```

## Update Profile

```bash
curl -X PATCH "https://getlate.dev/api/v1/profiles/PROFILE_ID" \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"name": "Updated Name", "description": "New description", "color": "#EF4444", "default": true}'
```

**Updatable fields:** `name`, `description`, `color`, `default`

## Delete Profile

```bash
curl -X DELETE "https://getlate.dev/api/v1/profiles/PROFILE_ID" \
  -H "Authorization: Bearer YOUR_API_KEY"
```

Permanently deletes the profile. Returns 200 on success, 404 if not found.

## Concepts

- Profiles are containers that group social accounts (think "brands" or "projects")
- Example: "Personal Brand" profile with Twitter + LinkedIn, "Company" profile with business accounts
- Each connected social account belongs to exactly one profile
- One profile can be set as `default`
- Profile `id` is used as `profileId` in connect and post endpoints
