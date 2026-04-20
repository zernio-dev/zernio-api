# Twitter / X Engagement

Actions beyond posting: retweet, bookmark, follow.

| Method | Path | Purpose |
|--------|------|---------|
| POST   | `/v1/twitter/retweet` | Retweet a tweet |
| DELETE | `/v1/twitter/retweet?accountId=...&tweetId=...` | Undo retweet |
| POST   | `/v1/twitter/bookmark` | Bookmark a tweet |
| DELETE | `/v1/twitter/bookmark?accountId=...&tweetId=...` | Remove bookmark |
| POST   | `/v1/twitter/follow` | Follow a user (by numeric Twitter ID) |
| DELETE | `/v1/twitter/follow?accountId=...&targetUserId=...` | Unfollow |

## Examples

```bash
# Retweet
curl -X POST https://zernio.com/api/v1/twitter/retweet \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{"accountId": "ACCOUNT_ID", "tweetId": "1234567890"}'

# Bookmark
curl -X POST https://zernio.com/api/v1/twitter/bookmark \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{"accountId": "ACCOUNT_ID", "tweetId": "1234567890"}'

# Follow
curl -X POST https://zernio.com/api/v1/twitter/follow \
  -H "Authorization: Bearer YOUR_API_KEY" \
  -d '{"accountId": "ACCOUNT_ID", "targetUserId": "44196397"}'
```

## Notes

- **Rate limits:** Retweet and bookmark each allow **50 / 15 min**. Retweeting also shares the **300 / 3h** tweet-creation limit.
- **Scopes:** bookmark requires `bookmark.write`; follow requires `follows.write`. If missing, the endpoint returns `400`.
- **Protected accounts:** `/v1/twitter/follow` against a protected user returns `200` with `pending_follow: true` — a follow request was sent but not yet accepted.
- `targetUserId` must be the **numeric** Twitter user ID (not username). Resolve usernames with the Twitter API or via inbox conversation lookups.
