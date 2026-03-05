# Analytics API

## Endpoints

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/v1/analytics` | Get general analytics |
| `GET` | `/v1/analytics/daily-metrics` | Daily aggregated metrics with platform breakdown |
| `GET` | `/v1/analytics/best-time-to-post` | Optimal posting times by day/hour |
| `GET` | `/v1/analytics/content-decay` | Engagement accumulation over time |
| `GET` | `/v1/analytics/posting-frequency` | Posting cadence vs engagement correlation |
| `GET` | `/v1/analytics/post-timeline` | Daily metric evolution for a specific post |
| `GET` | `/v1/analytics/youtube-daily-views` | YouTube video daily view counts |
| `GET` | `/v1/analytics/linkedin-aggregate` | LinkedIn account aggregate analytics |
| `GET` | `/v1/analytics/linkedin-post` | LinkedIn post analytics by URN |
| `GET` | `/v1/analytics/follower-stats` | Follower count history and growth |

## General Analytics

```bash
curl "https://getlate.dev/api/v1/analytics?profileId=PROFILE_ID" \
  -H "Authorization: Bearer YOUR_API_KEY"
```

Optional filter: `?postId=POST_ID` to get analytics for a specific post.

Analytics responses include `latePostId` field for correlation with scheduled posts (added Jan 2026).

## Daily Metrics (Feb 2026)

```bash
curl "https://getlate.dev/api/v1/analytics/daily-metrics?profileId=PROFILE_ID&startDate=2026-02-01&endDate=2026-02-28" \
  -H "Authorization: Bearer YOUR_API_KEY"
```

Returns daily aggregated analytics metrics with platform breakdown.

## Best Time to Post (Feb 2026)

```bash
curl "https://getlate.dev/api/v1/analytics/best-time-to-post?profileId=PROFILE_ID" \
  -H "Authorization: Bearer YOUR_API_KEY"
```

Returns optimal posting slots by day/hour based on historical engagement data.

## Content Decay (Feb 2026)

```bash
curl "https://getlate.dev/api/v1/analytics/content-decay?postId=POST_ID" \
  -H "Authorization: Bearer YOUR_API_KEY"
```

Shows how engagement accumulates over time after publishing — useful for understanding content shelf life.

## Posting Frequency (Feb 2026)

```bash
curl "https://getlate.dev/api/v1/analytics/posting-frequency?profileId=PROFILE_ID" \
  -H "Authorization: Bearer YOUR_API_KEY"
```

Correlates posting cadence with engagement rates — helps find the optimal number of posts per day/week.

## Post Timeline (Mar 2026)

```bash
curl "https://getlate.dev/api/v1/analytics/post-timeline?postId=POST_ID" \
  -H "Authorization: Bearer YOUR_API_KEY"
```

Returns daily timeline showing metric evolution for a specific post, broken down by platform.

## YouTube Daily Views

```bash
curl "https://getlate.dev/api/v1/analytics/youtube-daily-views?accountId=ACCOUNT_ID&videoId=VIDEO_ID&startDate=2026-01-01&endDate=2026-01-31" \
  -H "Authorization: Bearer YOUR_API_KEY"
```

Response:

```json
{
  "success": true,
  "videoId": "dQw4w9WgXcQ",
  "dateRange": {
    "startDate": "2026-01-01",
    "endDate": "2026-01-31"
  },
  "totalViews": 15420,
  "dailyViews": [
    {
      "date": "2026-01-01",
      "views": 523,
      "estimatedMinutesWatched": 1045,
      "averageViewDuration": 120,
      "subscribersGained": 12,
      "subscribersLost": 2,
      "likes": 45,
      "comments": 8,
      "shares": 15
    }
  ]
}
```

**Note:** Requires `yt-analytics.readonly` scope. If missing, response includes `reauthorizeUrl`.

## LinkedIn Analytics

```bash
# Aggregate analytics for account
curl "https://getlate.dev/api/v1/analytics/linkedin-aggregate?accountId=ACCOUNT_ID" \
  -H "Authorization: Bearer YOUR_API_KEY"

# Analytics for specific post
curl "https://getlate.dev/api/v1/analytics/linkedin-post?accountId=ACCOUNT_ID&postUrn=URN" \
  -H "Authorization: Bearer YOUR_API_KEY"
```

## Follower Stats

```bash
curl "https://getlate.dev/api/v1/analytics/follower-stats?profileId=PROFILE_ID" \
  -H "Authorization: Bearer YOUR_API_KEY"
```

Returns historical follower counts and growth metrics across all connected accounts.
