# DMS Daily Social Poster - Documentation Site

Visual documentation and mockups for the Dallas Makerspace automated social media posting system.

## Live Site

**https://olsonj-wps.github.io/Makerspace/**

## What's Here

- `index.html` - Interactive documentation with:
  - System flow overview
  - Facebook post mockup (grungy/industrial style)
  - Instagram post mockup
  - Reels/Stories mockup (9:16 vertical)
  - Full setup documentation
- `dallasmakerspace.png` - DMS logo
- `DOCUMENTATION.md` - Full text documentation

## The Actual Project

The automation code lives in a separate repo: `dms-social-poster`

```
~/Desktop/dms-social-poster/
├── .github/workflows/daily-post.yml   # GitHub Actions cron
├── src/
│   ├── index.js                       # Main entry
│   ├── fetchEvents.js                 # RSS parser
│   ├── filterToday.js                 # Date filter
│   ├── formatPost.js                  # Post formatter
│   ├── postToFacebook.js              # FB Graph API
│   └── postToInstagram.js             # IG Graph API
├── .env.example
└── package.json
```

## Quick Reference

| Item | Value |
|------|-------|
| RSS Feed | `calendar.dallasmakerspace.org/events/feed/rss` |
| Post Time | 7:00 AM Central (daily) |
| Platforms | Facebook + Instagram |
| Cost | $0/month |

## Setup Summary

1. Create Meta Developer app at developers.facebook.com
2. Get Page Access Token with `pages_manage_posts` and `instagram_content_publish`
3. Add secrets to GitHub repo: `META_PAGE_ACCESS_TOKEN`, `META_PAGE_ID`, `META_IG_USER_ID`
4. Push code, GitHub Actions handles the rest

See the [live documentation](https://olsonj-wps.github.io/Makerspace/) for full details.
