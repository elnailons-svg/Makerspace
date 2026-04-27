# DMS Daily Social Poster
## Automated Facebook & Instagram Posting for Dallas Makerspace

---

## Project Summary

| Field | Value |
|-------|-------|
| **Purpose** | Automatically post daily events to Facebook and Instagram |
| **Data Source** | https://calendar.dallasmakerspace.org/events/feed/rss |
| **Platforms** | Facebook Page + Instagram Business |
| **Schedule** | Daily at 7:00 AM Central |
| **Hosting** | GitHub Actions (free) |
| **Language** | Node.js |
| **Cost** | $0/month |

---

## Visual Style

The social media posts use a clean, modern "Squarespace-style" design:

### Layout Structure

```
┌─────────────────────────────┐
│         [DMS LOGO]          │  ← Top: Logo centered
│                             │
│                             │
│      Today's Classes        │  ← Middle: Headline over
│    at Dallas Makerspace     │     faded background image
│      See what's happening   │     (underlined subtitle)
│                             │
├─────────────────────────────┤
│  ┌───────────────────────┐  │
│  │  Monday, April 28     │  │  ← Bottom: Clean white/cream
│  │                       │  │     card with schedule table
│  │  TIME    CLASS   COST │  │
│  │  10:00am Ceramics $25 │  │
│  │  2:00pm  Wood    Free │  │
│  │  6:00pm  Laser   Mbr  │  │
│  │                       │  │
│  │  dallasmakerspace.org │  │
│  └───────────────────────┘  │
└─────────────────────────────┘
```

### Design Specs

| Element | Value |
|---------|-------|
| **Background** | Real makerspace photos from `/images/`, dimmed to 50% brightness |
| **Card Color** | Cream/off-white (`#faf8f5`) |
| **Card Corners** | 12px border-radius |
| **Card Shadow** | Soft upward shadow |
| **Primary Font** | Inter (400-700 weights) |
| **Monospace Font** | JetBrains Mono (for times) |
| **Free Highlight** | Green (`#22a866`) |
| **Headline** | White text, centered, over background |
| **Subtitle** | Underlined, 85% opacity |

### Background Photo Options

| CSS Class | Photo | Best For |
|-----------|-------|----------|
| `.post-bg` (default) | MS360_Screenshot-35.jpg | Square posts - full workshop view |
| `.post-bg.welding` | IMG_1080.jpg | Stories/Reels - welding shop |
| `.post-bg.cnc` | IMG_1070.jpg | Tech posts - CNC router |
| `.post-bg.lab` | IMG_1040.jpg | Science posts - lab glassware |

### Post Formats

| Format | Dimensions | API Support |
|--------|------------|-------------|
| Feed Post | 1:1 (1080x1080) | Yes |
| Reels | 9:16 (1080x1920) | Yes |
| Stories | 9:16 (1080x1920) | **No** - Use Meta Business Suite |
| Carousels | 1:1 (up to 10 images) | Yes |

---

## Event Display Rules

### Maximum Events Per Post

| Rule | Value |
|------|-------|
| **Max events per post** | 10 |
| **If more than 10** | Create additional post(s) |
| **Carousel option** | Split across slides (10 per slide) |

### Multi-Post Logic

```
Events Today: 15

Post 1: Events 1-10
Post 2: Events 11-15 + "See all at calendar.dallasmakerspace.org"
```

### Implementation

```javascript
const MAX_EVENTS_PER_POST = 10;

function splitIntoChunks(events, chunkSize = MAX_EVENTS_PER_POST) {
  const chunks = [];
  for (let i = 0; i < events.length; i += chunkSize) {
    chunks.push(events.slice(i, i + chunkSize));
  }
  return chunks;
}

// Usage
const todayEvents = filterToday(allEvents);
const eventChunks = splitIntoChunks(todayEvents);

for (let i = 0; i < eventChunks.length; i++) {
  const isLastPost = i === eventChunks.length - 1;
  const postNumber = eventChunks.length > 1 ? ` (${i + 1}/${eventChunks.length})` : '';

  const { text, imageUrl } = formatPost(eventChunks[i], {
    postNumber,
    showFullCalendarLink: isLastPost
  });

  await postToFacebook(text, imageUrl);
  await postToInstagram(text, imageUrl);
}
```

### Display Priority

When limiting to 10 events, prioritize by:

1. **Classes/Workshops** (paid events first - revenue generating)
2. **Certifications/Training** (member value)
3. **Open Events** (community engagement)
4. **Meetings** (lower priority)

### Edge Cases

| Scenario | Action |
|----------|--------|
| 0 events | Skip posting (no empty posts) |
| 1-10 events | Single post |
| 11-20 events | 2 posts |
| 21-30 events | 3 posts (or use carousel) |
| 30+ events | Consider carousel format or summary post |

---

## Photos Library

The `/images/` folder contains 90 real makerspace photos for use in social media posts.

### Photo Inventory

| Type | Count | Filename Pattern |
|------|-------|------------------|
| Workshop shots | 44 | `IMG_1030.jpg` - `IMG_1083.jpg` |
| 360° panoramas | 46 | `MS360_Screenshot-1.jpg` - `MS360_Screenshot-45.jpg` |

### Recommended Photos by Content Type

| Content | Recommended Photo | Why |
|---------|-------------------|-----|
| General/Daily posts | `MS360_Screenshot-35.jpg` | Shows full workshop with activity |
| Welding classes | `IMG_1080.jpg` | Welding shop with tools on wall |
| CNC/Laser classes | `IMG_1070.jpg` | CNC router bed in action |
| Science/Chemistry | `IMG_1040.jpg` | Lab glassware on shelves |
| Darkroom/Photo | `IMG_1055.jpg` | Photo enlarger equipment |
| Art/Gallery | `MS360_Screenshot-20.jpg` | Hallway with artwork displayed |

### Using Photos in Posts

**GitHub Raw URL:**
```
https://raw.githubusercontent.com/olsonj-wps/Makerspace/main/images/FILENAME.jpg
```

**In automation code:**
```javascript
const PHOTO_OPTIONS = {
  default: 'MS360_Screenshot-35.jpg',
  welding: 'IMG_1080.jpg',
  cnc: 'IMG_1070.jpg',
  lab: 'IMG_1040.jpg'
};

const imageUrl = `https://raw.githubusercontent.com/olsonj-wps/Makerspace/main/images/${PHOTO_OPTIONS.default}`;
```

---

## Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                     GitHub Actions                               │
│                  (Runs daily @ 7 AM CT)                          │
└─────────────────────┬───────────────────────────────────────────┘
                      │
                      ▼
┌─────────────────────────────────────────────────────────────────┐
│                     Node.js Script                               │
│                                                                  │
│   ┌─────────────┐   ┌─────────────┐   ┌─────────────────────┐   │
│   │ RSS Fetcher │ → │ Event Filter│ → │ Post Formatter      │   │
│   │             │   │ (today only)│   │ (text + image)      │   │
│   └─────────────┘   └─────────────┘   └──────────┬──────────┘   │
│                                                  │              │
└──────────────────────────────────────────────────┼──────────────┘
                                                   │
                        ┌──────────────────────────┴───────┐
                        ▼                                  ▼
              ┌─────────────────┐                ┌─────────────────┐
              │  Facebook Page  │                │    Instagram    │
              │   Graph API     │                │   Graph API     │
              └─────────────────┘                └─────────────────┘
```

---

## Sample Post Output

```
📅 TODAY at Dallas Makerspace (Mon, Apr 28)

🔧 Intro to Ceramics — 10:00 AM
   📍 Ceramics Room | $25

🎨 Woodworking 101 — 2:00 PM
   📍 Wood Shop | Free

⚡ Laser Cutter Training — 6:00 PM
   📍 Laser Lab | Members

🗓️ Full calendar: calendar.dallasmakerspace.org
```

---

## Project Structure

### Documentation Site (GitHub Pages)

**Repo:** `olsonj-wps/Makerspace`
**URL:** https://olsonj-wps.github.io/Makerspace/

```
Makerspace/
├── index.html           # Interactive mockup site
├── images/              # 90 real makerspace photos
│   ├── IMG_1030.jpg     # Workshop shots (44 photos)
│   ├── IMG_1040.jpg     # Lab glassware
│   ├── IMG_1070.jpg     # CNC router
│   ├── IMG_1080.jpg     # Welding shop
│   ├── MS360_Screenshot-1.jpg   # 360° panoramas (46 photos)
│   └── MS360_Screenshot-35.jpg  # Full workshop overview
├── dallasmakerspace.png # DMS logo
├── DOCUMENTATION.md     # This file
└── README.md            # Repo overview
```

### Automation Code

**Location:** `~/Desktop/dms-social-poster/`

```
dms-social-poster/
├── .github/
│   └── workflows/
│       └── daily-post.yml      # GitHub Actions cron job
├── src/
│   ├── index.js                # Main entry point
│   ├── fetchEvents.js          # RSS fetcher & parser
│   ├── filterToday.js          # Filter to today's events
│   ├── formatPost.js           # Create post text
│   ├── postToFacebook.js       # Facebook Graph API
│   └── postToInstagram.js      # Instagram Graph API
├── .env.example                # Template for secrets
├── .gitignore
├── package.json
├── README.md
└── PROJECT-WRITEUP.md          # Detailed documentation
```

---

## RSS Feed Data

**URL:** `https://calendar.dallasmakerspace.org/events/feed/rss`

Each event contains:

| Field | Example |
|-------|---------|
| `title` | "Laser Cutter Training" |
| `when` | "Monday, April 28, 2026 6:00pm - 8:00pm CDT" |
| `where` | "Laser Lab" |
| `details` | Event description |
| `host` | "John Smith" |
| `cost` | "Free for members" |
| `image` | URL to event image (optional) |
| `category` | Class, Event, Workshop, etc. |

---

## Setup Instructions

### Step 1: Test Locally

```bash
cd ~/Desktop/dms-social-poster
npm install
npm test   # Dry run - shows what would post
```

### Step 2: Create Meta Developer App

1. Go to https://developers.facebook.com/
2. Click **"Create App"**
3. Select **"Business"** as app type
4. Name it: `DMS Social Poster`
5. Click **"Create App"**

### Step 3: Add API Products

In your app dashboard:
1. Click **"Add Product"**
2. Add **Facebook Login**
3. Add **Instagram Graph API**

### Step 4: Get Page Access Token

1. Go to https://developers.facebook.com/tools/explorer/
2. Select your app from dropdown
3. Click **"Get User Access Token"**
4. Select these permissions:
   - `pages_manage_posts`
   - `pages_read_engagement`
   - `instagram_basic`
   - `instagram_content_publish`
5. Click **"Generate Access Token"**
6. Authorize when prompted
7. Select your **Page** from "User or Page" dropdown
8. **Copy the Page Access Token**

### Step 5: Get Page ID

```bash
curl "https://graph.facebook.com/v18.0/me/accounts?access_token=YOUR_TOKEN"
```

Response:
```json
{
  "data": [{
    "id": "123456789",        ← PAGE_ID
    "name": "Dallas Makerspace"
  }]
}
```

### Step 6: Get Instagram Business Account ID

```bash
curl "https://graph.facebook.com/v18.0/YOUR_PAGE_ID?fields=instagram_business_account&access_token=YOUR_TOKEN"
```

Response:
```json
{
  "instagram_business_account": {
    "id": "987654321"         ← IG_USER_ID
  }
}
```

### Step 7: Configure Local Environment

```bash
cd ~/Desktop/dms-social-poster
cp .env.example .env
```

Edit `.env` with your values:
```
META_PAGE_ACCESS_TOKEN=EAAG...your_token
META_PAGE_ID=123456789
META_IG_USER_ID=987654321
DRY_RUN=false
```

### Step 8: Test Real Post

```bash
npm start
```

### Step 9: Push to GitHub

```bash
cd ~/Desktop/dms-social-poster
git init
git add .
git commit -m "Initial commit: DMS Daily Social Poster"
gh repo create dms-social-poster --private --source=. --push
```

### Step 10: Add GitHub Secrets

1. Go to your repo on GitHub
2. Settings → Secrets and variables → Actions
3. Click **"New repository secret"**
4. Add these three secrets:

| Secret Name | Value |
|-------------|-------|
| `META_PAGE_ACCESS_TOKEN` | Your Page Access Token |
| `META_PAGE_ID` | Your Facebook Page ID |
| `META_IG_USER_ID` | Your Instagram Business Account ID |

### Step 11: Test GitHub Action

1. Go to **Actions** tab in your repo
2. Click **"Daily Social Media Post"**
3. Click **"Run workflow"**
4. Check **"Run in test mode"** for dry run
5. Click **"Run workflow"**
6. Watch the logs to confirm it works

---

## Schedule

The automation runs daily via GitHub Actions cron:

| Time | Timezone |
|------|----------|
| 7:00 AM | Central Time (Chicago) |
| 12:00 PM | UTC (summer/CDT) |
| 1:00 PM | UTC (winter/CST) |

**Note:** The current cron is set for 12:00 UTC which equals 7 AM CDT (summer). During winter (CST), posts will go out at 6 AM Central. Consider adding a second cron entry for winter if exact timing matters.

**To change the time:** Edit `.github/workflows/daily-post.yml`:
```yaml
- cron: '0 12 * * *'  # 12:00 UTC = 7 AM CDT
```

---

## Manual Trigger

To post immediately (outside schedule):

1. GitHub repo → **Actions** tab
2. Select **"Daily Social Media Post"**
3. Click **"Run workflow"**
4. Uncheck "dry run" for real post
5. Click **"Run workflow"**

---

## Troubleshooting

| Issue | Cause | Solution |
|-------|-------|----------|
| "Invalid OAuth access token" | Token expired | Generate new token (Step 4) |
| "Pages API access denied" | Missing permissions | Re-authorize with correct permissions |
| "Instagram container error" | Image URL not accessible | Ensure image is publicly hosted |
| No events posting | No events today | Check RSS feed has today's events |
| Wrong timezone | Server timezone | Code uses `America/Chicago` |

### Debug Commands

```bash
# Test RSS feed
curl -s "https://calendar.dallasmakerspace.org/events/feed/rss" | head -50

# Test Facebook API connection
curl "https://graph.facebook.com/v18.0/me?access_token=YOUR_TOKEN"

# Check token permissions
curl "https://graph.facebook.com/v18.0/me/permissions?access_token=YOUR_TOKEN"
```

---

## Token Management

**Important:** Page Access Tokens from Graph API Explorer expire after ~60 days.

### Options for Long-Term Use:

**Option 1: Calendar Reminder**
- Set reminder to refresh token every 60 days
- Regenerate token using Step 4
- Update GitHub secret

**Option 2: System User Token (Never Expires)**
1. Go to Business Manager → Business Settings
2. Users → System Users → Add
3. Generate token for System User
4. Use this permanent token

**Option 3: Long-Lived Token Exchange**
```bash
# Exchange short token for 60-day token
curl "https://graph.facebook.com/v18.0/oauth/access_token?\
grant_type=fb_exchange_token&\
client_id=YOUR_APP_ID&\
client_secret=YOUR_APP_SECRET&\
fb_exchange_token=YOUR_SHORT_TOKEN"
```

---

## Posting Formats Reference

### Feed Posts (1:1)
- Supported via API
- Image required for Instagram
- Recommended: 1080x1080px

### Reels (9:16)
- Supported via API
- Video: 3-90 seconds
- Max file size: 1GB
- Recommended: 1080x1920px

### Stories (9:16)
- **NOT supported via API**
- Use Meta Business Suite (free) to schedule
- URL: https://business.facebook.com

### Carousels
- Supported via API
- Up to 10 images
- All images same aspect ratio

---

## Maintenance Checklist

### Weekly
- [ ] Check GitHub Actions logs for failures
- [ ] Verify posts appearing on FB/IG

### Monthly
- [ ] Update npm dependencies: `npm update`
- [ ] Check for security advisories: `npm audit`

### Every 60 Days
- [ ] Refresh Page Access Token
- [ ] Update GitHub secret

---

## Cost Breakdown

| Component | Cost |
|-----------|------|
| GitHub Actions | Free (2,000 min/month) |
| Meta Graph API | Free |
| Hosting | None required |
| **Total** | **$0/month** |

---

## Dependencies

```json
{
  "cheerio": "^1.0.0-rc.12",
  "date-fns": "^2.30.0",
  "date-fns-tz": "^2.0.0",
  "dotenv": "^16.3.1",
  "node-fetch": "^2.7.0",
  "rss-parser": "^3.13.0"
}
```

---

## Future Enhancements

- [ ] Generate image with schedule overlay (matching mockup style)
- [ ] Carousel posts with multiple event images
- [ ] Include registration/RSVP links
- [ ] Weekly digest summary
- [ ] Twitter/X integration
- [ ] Slack notifications on post success/failure
- [ ] Analytics tracking
- [ ] Retry logic for failed API calls
- [ ] DST-aware scheduling (dual cron entries)

---

## Quick Reference

### Local Commands
```bash
cd ~/Desktop/dms-social-poster
npm install          # Install dependencies
npm test             # Dry run
npm start            # Real post
```

### Key URLs
- RSS Feed: https://calendar.dallasmakerspace.org/events/feed/rss
- Meta Developer: https://developers.facebook.com/
- Graph Explorer: https://developers.facebook.com/tools/explorer/
- API Docs: https://developers.facebook.com/docs/graph-api/
- Business Suite: https://business.facebook.com
- Mockup Site: https://olsonj-wps.github.io/Makerspace/

### Environment Variables
```
META_PAGE_ACCESS_TOKEN   # Page Access Token
META_PAGE_ID             # Facebook Page ID
META_IG_USER_ID          # Instagram Business Account ID
DRY_RUN                  # true/false
```

### GitHub Repos
| Repo | Purpose |
|------|---------|
| `olsonj-wps/Makerspace` | Documentation site (GitHub Pages) |
| `dms-social-poster` | Automation code (to be created) |

---

*Documentation for Dallas Makerspace Social Media Automation*
*Last Updated: April 2026*
