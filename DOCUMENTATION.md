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

🔧 Purple Wall Paint Day! — 10:00 AM
   📍 Main Hall | Free

🎨 Intro to Ceramics — 2:00 PM
   📍 Ceramics Room | $25

⚡ Laser Cutter Training — 6:00 PM
   📍 Laser Lab | Members

🗓️ Full calendar: calendar.dallasmakerspace.org
```

---

## Project Files

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

Run this command (replace YOUR_TOKEN):

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
| 12:00 PM | UTC |

**To change the time:** Edit `.github/workflows/daily-post.yml` line:
```yaml
- cron: '0 12 * * *'  # 12:00 UTC = 7 AM Central
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

## Future Enhancements

- [ ] Carousel posts with multiple event images
- [ ] Include registration/RSVP links
- [ ] Weekly digest summary
- [ ] Twitter/X integration
- [ ] Slack notifications on post success/failure
- [ ] Analytics tracking

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

### Environment Variables
```
META_PAGE_ACCESS_TOKEN   # Page Access Token
META_PAGE_ID             # Facebook Page ID
META_IG_USER_ID          # Instagram Business Account ID
DRY_RUN                  # true/false
```

---

*Documentation for Dallas Makerspace Social Media Automation*
*Project Location: ~/Desktop/dms-social-poster/*
