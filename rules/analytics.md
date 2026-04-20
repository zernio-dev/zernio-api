# Analytics API

## Endpoints

All analytics endpoints require the **analytics add-on** (except YouTube daily views, which requires the YouTube `yt-analytics.readonly` scope on the account).

| Method | Endpoint | Description |
|--------|----------|-------------|
| `GET` | `/v1/analytics` | General per-account analytics summary |
| `GET` | `/v1/analytics/daily-metrics` | Daily timeline of engagement/impressions |
| `GET` | `/v1/analytics/post-timeline` | Per-post performance across platforms |
| `GET` | `/v1/analytics/best-time` | Optimal posting times based on your history |
| `GET` | `/v1/analytics/posting-frequency` | Current posting cadence vs. recommended |
| `GET` | `/v1/analytics/content-decay` | How long posts keep driving impressions |
| `GET` | `/v1/analytics/youtube/daily-views` | YouTube video daily views (with watch time, subs) |
| `GET` | `/v1/analytics/youtube/demographics` | YouTube audience demographics |
| `GET` | `/v1/analytics/instagram/account-insights` | Instagram account-level insights (reach, profile views) |
| `GET` | `/v1/analytics/instagram/demographics` | Instagram follower demographics |
| `GET` | `/v1/analytics/googlebusiness/performance` | GBP performance (calls, directions, website clicks, searches) |
| `GET` | `/v1/analytics/googlebusiness/search-keywords` | Top search queries that surfaced your GBP listing |
| `GET` | `/v1/accounts/{accountId}/linkedin-aggregate-analytics` | LinkedIn account analytics |
| `GET` | `/v1/accounts/{accountId}/linkedin-post-analytics` | LinkedIn post analytics |
| `GET` | `/v1/accounts/{accountId}/linkedin-post-reactions` | LinkedIn post reactions breakdown |
| `GET` | `/v1/accounts/follower-stats` | Follower count history with growth metrics |

## YouTube Daily Views

```bash
curl "https://zernio.com/api/v1/analytics/youtube/daily-views?accountId=ACCOUNT_ID&videoId=VIDEO_ID&startDate=2024-01-01&endDate=2024-01-31" \
  -H "Authorization: Bearer YOUR_API_KEY"
```

Response:

```json
{
  "success": true,
  "videoId": "dQw4w9WgXcQ",
  "dateRange": {
    "startDate": "2024-01-01",
    "endDate": "2024-01-31"
  },
  "totalViews": 15420,
  "dailyViews": [
    {
      "date": "2024-01-01",
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
curl "https://zernio.com/api/v1/accounts/ACCOUNT_ID/linkedin-aggregate-analytics" \
  -H "Authorization: Bearer YOUR_API_KEY"

# Analytics for specific post
curl "https://zernio.com/api/v1/accounts/ACCOUNT_ID/linkedin-post-analytics?postUrn=URN" \
  -H "Authorization: Bearer YOUR_API_KEY"
```

## Follower Stats

```bash
curl "https://zernio.com/api/v1/accounts/follower-stats?profileId=PROFILE_ID&fromDate=2026-03-01&toDate=2026-04-01&granularity=daily" \
  -H "Authorization: Bearer YOUR_API_KEY"
```

Parameters: `accountIds` (comma-separated; default all accounts), `profileId`, `fromDate` / `toDate` (default last 30 days), `granularity` (`daily`, `weekly`, `monthly`; default `daily`). Follower counts refresh once per day.

## Instagram Insights

```bash
# Account-level reach, impressions, profile views
curl "https://zernio.com/api/v1/analytics/instagram/account-insights?accountId=ACC&fromDate=2026-04-01&toDate=2026-04-30" \
  -H "Authorization: Bearer YOUR_API_KEY"

# Audience demographics (age, gender, country, city)
curl "https://zernio.com/api/v1/analytics/instagram/demographics?accountId=ACC" \
  -H "Authorization: Bearer YOUR_API_KEY"
```

## YouTube Demographics

```bash
curl "https://zernio.com/api/v1/analytics/youtube/demographics?accountId=ACC&videoId=VIDEO_ID" \
  -H "Authorization: Bearer YOUR_API_KEY"
```

## Google Business Performance

```bash
# Calls, direction requests, website clicks, views by search type
curl "https://zernio.com/api/v1/analytics/googlebusiness/performance?accountId=ACC&fromDate=2026-04-01&toDate=2026-04-30" \
  -H "Authorization: Bearer YOUR_API_KEY"

# Top keywords your listing appeared for
curl "https://zernio.com/api/v1/analytics/googlebusiness/search-keywords?accountId=ACC&months=3" \
  -H "Authorization: Bearer YOUR_API_KEY"
```

## Best Time / Posting Frequency / Content Decay

Derived metrics across all your posts for a profile:

```bash
# When your audience is most likely to engage
curl "https://zernio.com/api/v1/analytics/best-time?profileId=PROFILE_ID&platform=instagram" \
  -H "Authorization: Bearer YOUR_API_KEY"

# Posting frequency vs. platform-recommended cadence
curl "https://zernio.com/api/v1/analytics/posting-frequency?profileId=PROFILE_ID" \
  -H "Authorization: Bearer YOUR_API_KEY"

# How long posts keep driving impressions after publishing
curl "https://zernio.com/api/v1/analytics/content-decay?profileId=PROFILE_ID&platform=twitter" \
  -H "Authorization: Bearer YOUR_API_KEY"
```

## Post Timeline / Daily Metrics

```bash
# Per-post performance across platforms
curl "https://zernio.com/api/v1/analytics/post-timeline?profileId=PROFILE_ID&fromDate=2026-04-01&toDate=2026-04-30" \
  -H "Authorization: Bearer YOUR_API_KEY"

# Daily timeline of aggregated engagement/impressions
curl "https://zernio.com/api/v1/analytics/daily-metrics?profileId=PROFILE_ID&platform=linkedin&granularity=daily" \
  -H "Authorization: Bearer YOUR_API_KEY"
```
