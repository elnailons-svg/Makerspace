# DMS Daily Social Poster - Documentation Site

Visual documentation and mockups for the Dallas Makerspace automated social media posting system.

## Live Site

**https://olsonj-wps.github.io/Makerspace/**

## What's Here

- `index.html` - Interactive documentation with:
  - System flow overview
  - Facebook post mockup (using real makerspace photos)
  - Instagram post mockup
  - Reels/Stories mockup (9:16 vertical)
  - Full setup documentation
- `images/` - 90 real makerspace photos for social media posts
- `dallasmakerspace.png` - DMS logo
- `DOCUMENTATION.md` - Full text documentation

## Photos Library

The `/images/` folder contains 90 photos from the makerspace:

| Type | Count | Examples |
|------|-------|----------|
| Workshop shots | 44 | IMG_1030.jpg - IMG_1083.jpg |
| 360° panoramas | 46 | MS360_Screenshot-1.jpg - MS360_Screenshot-45.jpg |

### Photo Highlights

| Photo | Description | Best For |
|-------|-------------|----------|
| `MS360_Screenshot-35.jpg` | Full workshop overview | Square posts (1:1) |
| `IMG_1080.jpg` | Welding shop with tools | Stories/Reels (9:16) |
| `IMG_1070.jpg` | CNC router in action | Tech-focused posts |
| `IMG_1040.jpg` | Lab glassware | Science/chemistry posts |

### Using Photos in Posts

**Raw GitHub URL pattern:**
```
https://raw.githubusercontent.com/olsonj-wps/Makerspace/main/images/FILENAME.jpg
```

**Example:**
```html
<img src="https://raw.githubusercontent.com/olsonj-wps/Makerspace/main/images/IMG_1070.jpg" alt="CNC Router">
```

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
| Photos | 90 images in `/images/` |

## Setup Summary

1. Create Meta Developer app at developers.facebook.com
2. Get Page Access Token with `pages_manage_posts` and `instagram_content_publish`
3. Add secrets to GitHub repo: `META_PAGE_ACCESS_TOKEN`, `META_PAGE_ID`, `META_IG_USER_ID`
4. Push code, GitHub Actions handles the rest

See the [live documentation](https://olsonj-wps.github.io/Makerspace/) for full details.
