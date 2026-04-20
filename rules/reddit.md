# Reddit Search & Feed

Read-side Reddit endpoints. Complements the posting-side Reddit features in `platforms.md` (subreddits, flairs, native video).

| Method | Path | Purpose |
|--------|------|---------|
| GET    | `/v1/reddit/search` | Search posts (optionally scoped to one subreddit) |
| GET    | `/v1/reddit/feed` | Fetch a subreddit feed (hot/new/top/rising) |

## Search

```bash
curl "https://zernio.com/api/v1/reddit/search?accountId=ACC&q=social%20media&subreddit=marketing&sort=top&limit=25" \
  -H "Authorization: Bearer YOUR_API_KEY"
```

**Required:** `accountId`, `q`. **Optional:** `subreddit`, `restrict_sr` (`0`/`1`), `sort` (`relevance`, `hot`, `top`, `new`, `comments`; default `new`), `limit` (max 100), `after` (cursor).

## Feed

```bash
curl "https://zernio.com/api/v1/reddit/feed?accountId=ACC&subreddit=socialmedia&sort=hot&limit=25" \
  -H "Authorization: Bearer YOUR_API_KEY"
```

**Required:** `accountId`. **Optional:** `subreddit` (omit for the user's home feed), `sort` (`hot`, `new`, `top`, `rising`; default `hot`), `t` (time window when `sort=top`: `hour`, `day`, `week`, `month`, `year`, `all`), `limit` (max 100), `after`.

## Response shape

Both endpoints return `{ items, after, before }` where each item is a `RedditPost`:

```json
{
  "id": "1abc234",
  "fullname": "t3_1abc234",
  "title": "How to grow on social media",
  "selftext": "Here are my tips...",
  "author": "marketingpro",
  "subreddit": "socialmedia",
  "url": "https://www.reddit.com/r/socialmedia/comments/1abc234/",
  "permalink": "https://...",
  "score": 156,
  "numComments": 42,
  "createdUtc": 1730000000,
  "over18": false,
  "stickied": false,
  "flairText": null,
  "isGallery": false,
  "galleryImages": ["https://i.redd.it/abc123.jpg", "..."]
}
```

`galleryImages[]` is only populated when `isGallery: true`. Use `after` as the next page cursor.
