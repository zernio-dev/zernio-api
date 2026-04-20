# Google Business Profile Management

Endpoints for managing Google Business Profile (GBP) listings beyond posting: reviews, location details, attributes, services, menus, photos, and action links.

All endpoints take the Zernio GBP account ID at `{accountId}` (from `/v1/accounts`). Every endpoint accepts an optional `?locationId=` query to override the account's default selected location — use `GET /v1/accounts/{accountId}/gmb-locations` to list the valid IDs.

## Connection flow (headless)

| Method | Path | Purpose |
|--------|------|---------|
| GET    | `/v1/connect/googlebusiness/locations` | List manageable locations (requires `pendingDataToken` from OAuth callback, or legacy `tempToken`) |
| POST   | `/v1/connect/googlebusiness/select-location` | Save the user's chosen location and finalize the connection |

After OAuth, Google redirects to your URL with `step=select_location&pendingDataToken=...`. Use that token to list locations and select one. See `connect.md` for the general OAuth flow.

## Reviews

| Method | Path | Purpose |
|--------|------|---------|
| GET    | `/v1/accounts/{accountId}/gmb-reviews` | List reviews (paginated, includes owner replies) |
| POST   | `/v1/accounts/{accountId}/gmb-reviews/batch` | Fetch reviews across multiple locations in one request |

```bash
curl "https://zernio.com/api/v1/accounts/ACC/gmb-reviews?pageSize=50" \
  -H "Authorization: Bearer YOUR_API_KEY"
```

Response includes `reviews[]` (with `reviewer`, `rating` 1–5, `starRating` enum, `comment`, `reviewReply`), `averageRating`, `totalReviewCount`, `nextPageToken`.

**Batch across locations:**
```bash
curl -X POST https://zernio.com/api/v1/accounts/ACC/gmb-reviews/batch \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{"locationNames": ["accounts/123/locations/456", "accounts/123/locations/789"], "pageSize": 50}'
```

Reply to a review via the unified inbox: `POST /v1/inbox/reviews/{reviewId}/reply` (see `inbox.md`). Delete a reply (GBP only): `DELETE /v1/inbox/reviews/{reviewId}/reply`.

## Location Details

| Method | Path | Purpose |
|--------|------|---------|
| GET    | `/v1/accounts/{accountId}/gmb-location-details` | Hours, description, phone, website, categories, services |
| PUT    | `/v1/accounts/{accountId}/gmb-location-details` | Update with a required `updateMask` |

GET supports `readMask` (comma-separated fields): `name`, `title`, `phoneNumbers`, `categories`, `storefrontAddress`, `websiteUri`, `regularHours`, `specialHours`, `serviceArea`, `serviceItems`, `profile`, `openInfo`, `metadata`, `moreHours`.

PUT proxies Google's Business Information API `locations.patch` — any valid `updateMask` works. Common fields:

```bash
# Update business hours + holiday closures
curl -X PUT https://zernio.com/api/v1/accounts/ACC/gmb-location-details \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{
    "updateMask": "regularHours,specialHours",
    "regularHours": {
      "periods": [
        { "openDay": "MONDAY", "openTime": "09:00", "closeDay": "MONDAY", "closeTime": "17:00" },
        { "openDay": "SATURDAY", "openTime": "10:00", "closeDay": "SATURDAY", "closeTime": "14:00" }
      ]
    },
    "specialHours": {
      "specialHourPeriods": [
        { "startDate": { "year": 2026, "month": 12, "day": 25 }, "closed": true }
      ]
    }
  }'

# Update categories
# updateMask: "categories" + primaryCategory.name = "categories/gcid:laundromat"

# Update services (see also GMB Services below)
# updateMask: "serviceItems"
```

## Attributes

Amenities / services flags (e.g. `has_delivery`, `has_outdoor_seating`, `pay_credit_card_types_accepted`). Available attributes vary by category.

| Method | Path | Purpose |
|--------|------|---------|
| GET    | `/v1/accounts/{accountId}/gmb-attributes` | Current attribute values |
| PUT    | `/v1/accounts/{accountId}/gmb-attributes` | Update with `attributeMask` |

Types: `BOOL` (`values: [true]`), `ENUM`, `URL`, `REPEATED_ENUM` (uses `repeatedEnumValue.setValues` / `unsetValues`).

```bash
curl -X PUT https://zernio.com/api/v1/accounts/ACC/gmb-attributes \
  -d '{
    "attributes": [
      { "name": "has_delivery", "values": [true] },
      { "name": "has_takeout",  "values": [true] },
      { "name": "has_outdoor_seating", "values": [false] }
    ],
    "attributeMask": "has_delivery,has_takeout,has_outdoor_seating"
  }'
```

## Services

| Method | Path | Purpose |
|--------|------|---------|
| GET    | `/v1/accounts/{accountId}/gmb-services` | List services |
| PUT    | `/v1/accounts/{accountId}/gmb-services` | **Replace entire list** (Google API requires full replacement) |

Each item is either `structuredServiceItem` (Google catalog `serviceTypeId`, e.g. `job_type_id:plumbing_drain_repair`) or `freeFormServiceItem` (custom `category` + `label.displayName`). Both accept an optional `price`.

```bash
curl -X PUT https://zernio.com/api/v1/accounts/ACC/gmb-services \
  -d '{
    "serviceItems": [
      {
        "freeFormServiceItem": {
          "category": "categories/gcid:plumber",
          "label": { "displayName": "Pipe Repair", "description": "Emergency and scheduled pipe repair" }
        },
        "price": { "currencyCode": "USD", "units": "150" }
      }
    ]
  }'
```

Individual service updates are not supported — always send the full desired list.

## Food Menus

For restaurants / food businesses with menu support.

| Method | Path | Purpose |
|--------|------|---------|
| GET    | `/v1/accounts/{accountId}/gmb-food-menus` | Full menu structure |
| PUT    | `/v1/accounts/{accountId}/gmb-food-menus` | Replace menus (optional `updateMask`) |

Menus have `labels[]` (localized display name), `sections[]`, and nested `items[]` with `price`, `dietaryRestriction[]` (`VEGETARIAN`, `VEGAN`, `GLUTEN_FREE`, ...), `allergen[]`, etc.

## Media (Photos)

| Method | Path | Purpose |
|--------|------|---------|
| GET    | `/v1/accounts/{accountId}/gmb-media` | List photos / videos (`pageSize` max 100, `pageToken`) |
| POST   | `/v1/accounts/{accountId}/gmb-media` | Upload from URL (`sourceUrl`, `mediaFormat`, `category`, `description`) |
| DELETE | `/v1/accounts/{accountId}/gmb-media?mediaId=...` | Delete a media item |

**Categories** (where the photo appears): `COVER`, `PROFILE`, `LOGO`, `EXTERIOR`, `INTERIOR`, `FOOD_AND_DRINK`, `MENU`, `PRODUCT`, `TEAMS`, `ADDITIONAL`.

```bash
curl -X POST https://zernio.com/api/v1/accounts/ACC/gmb-media \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{
    "sourceUrl": "https://example.com/photos/interior.jpg",
    "mediaFormat": "PHOTO",
    "category": "INTERIOR",
    "description": "Dining area with outdoor seating"
  }'
```

## Place Action Links

Booking / ordering / reservation buttons that appear on the listing.

| Method | Path | Purpose |
|--------|------|---------|
| GET    | `/v1/accounts/{accountId}/gmb-place-actions` | List action links |
| POST   | `/v1/accounts/{accountId}/gmb-place-actions` | Create (`uri` + `placeActionType`) |
| PATCH  | `/v1/accounts/{accountId}/gmb-place-actions` | Update by resource `name` |
| DELETE | `/v1/accounts/{accountId}/gmb-place-actions?name=...` | Delete |

**Action types:** `APPOINTMENT`, `ONLINE_APPOINTMENT`, `DINING_RESERVATION`, `FOOD_ORDERING`, `FOOD_DELIVERY`, `FOOD_TAKEOUT`, `SHOP_ONLINE`.

```bash
curl -X POST https://zernio.com/api/v1/accounts/ACC/gmb-place-actions \
  -d '{"uri": "https://order.ubereats.com/joespizza", "placeActionType": "FOOD_ORDERING"}'
```

## Performance analytics

For impressions, clicks, calls, direction requests, and top search keywords, see `analytics.md`:
- `/v1/analytics/googlebusiness/performance`
- `/v1/analytics/googlebusiness/search-keywords`
