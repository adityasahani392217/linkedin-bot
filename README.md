# 🤖 AI LinkedIn Content Bot

> Fully automated daily LinkedIn post generator with Telegram approval — built with n8n, OpenAI, and zero paid APIs for fetching trends.

![Made with n8n](https://img.shields.io/badge/Built%20with-n8n-orange?style=flat-square)
![AI Powered](https://img.shields.io/badge/AI-GPT--4.1--nano-blue?style=flat-square)
![License](https://img.shields.io/badge/license-MIT-green?style=flat-square)
![Status](https://img.shields.io/badge/status-active-brightgreen?style=flat-square)

---

## 📌 What This Does

Every day at **10:00 AM IST**, this bot:

1. Scrapes **trending AI/ML topics** from Hacker News (free, no API key)
2. Sends the best topic to **GPT-4.1-nano** to write a viral LinkedIn post in your voice
3. Sends a **Telegram preview** with ✅ Approve / ⏸ Hold buttons
4. If approved → **waits until peak LinkedIn time** (set by GPT)
5. **Auto-posts to LinkedIn** via API
6. Sends you the **live post link** on Telegram
7. **Logs everything** to Google Sheets

---

## 🧠 Built By

**Aditya Sahani**
- AI Student (IBCP) · Python Developer
- Building real AI systems at 17: anomaly detection, automation pipelines, data scrapers

---

## 🗂️ Project Structure

```
linkedin-bot/
│
├── linkedin_bot_WORKING.json   ← n8n workflow (import this)
└── README.md                   ← this file
```

---

## ⚙️ Workflow Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                     DAILY AT 10:00 AM IST                       │
└──────────────────────────────┬──────────────────────────────────┘
                               │
                    ┌──────────▼──────────┐
                    │  Hacker News API    │  ← Free, no key needed
                    │  Fetch top AI posts │
                    └──────────┬──────────┘
                               │
                    ┌──────────▼──────────┐
                    │  Pick Best Topic    │  ← Filters by AI/ML keywords
                    │  Build GPT Prompt   │  ← Pre-builds JSON body
                    └──────────┬──────────┘
                               │
                    ┌──────────▼──────────┐
                    │  Wait 3s            │  ← Avoids rate limits
                    └──────────┬──────────┘
                               │
                    ┌──────────▼──────────┐
                    │  OpenAI GPT-4.1nano │  ← Writes post in your voice
                    │  Returns JSON post  │  ← Falls back if rate limited
                    └──────────┬──────────┘
                               │
                    ┌──────────▼──────────┐
                    │  Parse + Format     │  ← Converts IST→UTC for scheduling
                    │  Builds LI body     │  ← Pre-builds LinkedIn JSON
                    └──────────┬──────────┘
                               │
                    ┌──────────▼──────────┐
                    │  Telegram Preview   │  ← Sends post + 2 buttons
                    │  ✅ Approve  ⏸ Hold │
                    └──────────┬──────────┘
                               │
               ┌───────────────┴───────────────┐
               │                               │
    ┌──────────▼──────────┐        ┌──────────▼──────────┐
    │   ✅ APPROVED        │        │   ⏸ HELD             │
    │                     │        │                     │
    │  Wait until peak    │        │  "No post today"    │
    │  Post to LinkedIn   │        │  Ends. Restarts     │
    │  Send live link     │        │  tomorrow           │
    │  Log to Sheets      │        └─────────────────────┘
    └─────────────────────┘
```

---

## 🛠️ Setup Guide

### Prerequisites

- [n8n](https://n8n.io) self-hosted or cloud (free tier works)
- OpenAI account with **$5 credit** added
- Telegram account
- LinkedIn personal account (no company needed)
- Google account

---

### Step 1 — Import the Workflow

1. Open n8n
2. Click **+** → **Import from file**
3. Upload `linkedin_bot_WORKING.json`
4. You'll see 17 nodes connected — don't change any connections

---

### Step 2 — OpenAI API Key

1. Go to [platform.openai.com/api-keys](https://platform.openai.com/api-keys)
2. Click **Create new secret key** → copy it
3. Add $5 credit at [platform.openai.com/settings/billing](https://platform.openai.com/settings/billing) (lasts months for daily posts)
4. In the workflow → open `OpenAI Generate Post` node
5. Under Headers → replace `YOUR_OPENAI_API_KEY_HERE` with your key

> ⚠️ Never commit your API key to GitHub. Use n8n's credential store instead.

---

### Step 3 — Telegram Bot

**Create bot:**
1. Open Telegram → search **@BotFather** → start chat
2. Send `/newbot`
3. Choose a name (e.g. `Aditya LinkedIn Bot`)
4. Copy the **bot token** you receive

**Add to n8n:**
1. In n8n → top right menu → **Credentials**
2. Click **Add Credential** → search **Telegram**
3. Paste bot token → Save
4. Copy the credential ID shown in the URL or settings

**Update workflow:**
- Open each of the 4 Telegram nodes
- Under Credentials → select your saved Telegram credential

**Start the bot:**
- Open Telegram → search your bot name → click **Start**
- Your chat ID `XXXXXXXXXX` is already set in all nodes ✅

---

### Step 4 — LinkedIn App (No Company Needed)

#### 4a. Create the App

1. Go to [linkedin.com/developers/apps](https://linkedin.com/developers/apps)
2. Click **Create App**
3. **App name:** `My Content Bot`
4. **LinkedIn Page:** Click the field → type your own name → select your **personal profile** (not a company)
5. Upload any logo → check the legal box → **Create App**

#### 4b. Enable Permissions

1. Go to the **Products** tab
2. Request **Share on LinkedIn**
3. Request **Sign In with LinkedIn using OpenID Connect**
4. Both are usually approved within 1–2 minutes

#### 4c. Get Your Access Token

1. Go to **Auth** tab → copy your **Client ID** and **Client Secret**
2. Under OAuth 2.0 → add redirect URL: `https://oauth.pstmn.io/v1/callback`
3. Open this URL in your browser (replace `YOUR_CLIENT_ID`):

```
https://www.linkedin.com/oauth/v2/authorization?response_type=code&client_id=YOUR_CLIENT_ID&redirect_uri=https://oauth.pstmn.io/v1/callback&scope=w_member_social%20openid%20profile
```

4. Log in with LinkedIn → you get redirected
5. Copy the `code=XXXXXXXX` value from the redirect URL

6. Make this POST request (use Postman, Hoppscotch, or any API tool):

```
POST https://www.linkedin.com/oauth/v2/accessToken
Content-Type: application/x-www-form-urlencoded

grant_type=authorization_code
code=PASTE_CODE_HERE
client_id=YOUR_CLIENT_ID
client_secret=YOUR_CLIENT_SECRET
redirect_uri=https://oauth.pstmn.io/v1/callback
```

7. Copy the `access_token` from the response

#### 4d. Get Your Person URN

```
GET https://api.linkedin.com/v2/me
Authorization: Bearer YOUR_ACCESS_TOKEN
```

Copy the `id` value → format it as: `urn:li:person:THAT_ID`

#### 4e. Update the Workflow

- Open `Post to LinkedIn API` node → Authorization header → replace `YOUR_LINKEDIN_ACCESS_TOKEN_HERE`
- Open `Parse and Format Output` code node → find `LINKEDIN_PERSON_URN_HERE` → replace with your URN

---

### Step 5 — Google Sheets (for logging)

1. Create a new Google Sheet
2. Add a tab called `Posts Log`
3. Copy the Sheet ID from the URL:
   ```
   https://docs.google.com/spreadsheets/d/COPY_THIS_PART/edit
   ```
4. In n8n → open `Log to Google Sheets` node → paste your Sheet ID
5. Connect Google Sheets credential (n8n Credentials → Google Sheets OAuth2)

---

### Step 6 — Activate

1. Click **Activate** toggle in the top right of n8n
2. The workflow now runs automatically every day at 10:00 AM IST
3. Use the **Test Manually** node to do a test run right now

---

## 📱 Daily Flow (What You See)

```
10:00 AM IST
  Telegram message arrives with full post preview

  ┌────────────────────────────────────────┐
  │ 📝 LinkedIn Post Ready                 │
  │                                        │
  │ 📌 Topic: AI agents replacing SaaS     │
  │ ⏰ Post at: 11:00 IST tomorrow         │
  │ ──────────────────────────────         │
  │ [full post text here]                  │
  │                                        │
  │ #AI #BuildInPublic #Automation         │
  │ ──────────────────────────────         │
  │ ✅ Approve    ⏸ Hold                   │
  └────────────────────────────────────────┘

→ Tap ✅  →  "Scheduled for 11:00 IST tomorrow"
→ Tomorrow 11:00 AM  →  Posts to LinkedIn
→ "🚀 LIVE on LinkedIn! [link]"

→ Tap ⏸  →  "Held. Fresh content tomorrow."
→ Workflow ends. Restarts fresh next morning.
```

---

## 📊 Google Sheets Log

| Date | Topic | Post | Hashtags | Used Fallback | Posted At | Post URL | Status |
|------|-------|------|----------|---------------|-----------|----------|--------|
| 22/3/2026 | Deep Learning AI... | I built an... | #AI #ML... | false | 2026-03-23T... | linkedin.com/... | Published |

---

## 🔧 Configuration Reference

| Location | What to replace |
|---|---|
| `OpenAI Generate Post` → Auth header | `YOUR_OPENAI_API_KEY_HERE` |
| `Post to LinkedIn API` → Auth header | `YOUR_LINKEDIN_ACCESS_TOKEN_HERE` |
| `Parse and Format Output` → code line 4 | `LINKEDIN_PERSON_URN_HERE` |
| `Log to Google Sheets` → Document ID | `PASTE_YOUR_GOOGLE_SHEET_ID_HERE` |
| All Telegram nodes → Credentials | Select your saved Telegram credential |

---

## 🚨 Security Notes

- **Never commit API keys** to GitHub — use n8n's credential manager
- LinkedIn access tokens expire in **60 days** — re-run the OAuth flow to refresh
- OpenAI keys are single-use secrets — revoke immediately if accidentally exposed

---

## 🐛 Troubleshooting

| Error | Cause | Fix |
|---|---|---|
| `400 invalid JSON` | Expression in raw body field | Body must be `={{ $json.openai_body }}` not raw JSON with expressions |
| `429 rate limit` | Too many OpenAI calls | Wait node + fallback handles this automatically |
| Telegram buttons not working | Missing `inline_keyboard` in reply_markup | Already fixed in this version |
| LinkedIn `401 unauthorized` | Token expired or wrong URN | Re-run OAuth flow, check URN format |
| No posts extracted | HN returned no AI topics | Fallback post activates automatically |

---

## 📦 Tech Stack

| Tool | Purpose | Cost |
|---|---|---|
| **n8n** | Workflow automation | Free (self-hosted) |
| **Hacker News API** | Trending topics | Free, no key |
| **OpenAI GPT-4.1-nano** | Post generation | ~$0.01/day |
| **Telegram Bot API** | Approval interface | Free |
| **LinkedIn API** | Publishing | Free |
| **Google Sheets** | Logging | Free |

---

## 📄 License

MIT — use it, modify it, ship it.

---

*Built by [Aditya Sahani](https://linkedin.com/in/adityasahani) · AI Student · IIT Bombay Eureka Junior Top 100*
