# Daily Tech Digest — AI-Powered WhatsApp News Bot

An automated workflow that scrapes multiple tech RSS feeds daily, uses **Groq LLaMA 3.3-70b** to curate and summarize the top stories like a senior tech journalist, and delivers a clean digest to your **WhatsApp** every morning at 8AM.

---

## What It Does

- Pulls articles from 4 RSS sources (TechCrunch, Hacker News, Dev.to, MIT Technology Review)
- Preprocesses and strips raw XML down to clean structured data
- Feeds it to an **AI Agent** (Groq LLaMA 3.3-70b) with a journalist-style system prompt
- AI deduplicates stories across sources, merges multi-source coverage, scores by relevance, and writes 3-line summaries
- Formats and delivers a **WhatsApp digest** via Twilio every day at 8AM

---

## Architecture

```
Schedule Trigger (8AM daily)
    ↓
5x HTTP Request nodes (RSS feeds)
    ↓
Merge (append all feeds)
    ↓
Limit (20 items)
    ↓
Code node (XML preprocessor — strips to title + url + description)
    ↓
AI Agent (Groq LLaMA 3.3-70b — curates, scores, summarizes)
    ↓
Code node (formats WhatsApp message)
    ↓
HTTP Request → Twilio WhatsApp API
```

---

## Tech Stack

| Tool | Purpose |
|------|---------|
| N8N | Workflow automation |
| Groq API | LLM inference (LLaMA 3.3-70b-versatile) |
| Twilio | WhatsApp delivery |
| JavaScript | XML preprocessing + message formatting |
| RSS Feeds | TechCrunch, HN Jobs, Dev.to, MIT Tech Review, Wired |

---

## RSS Sources

| Source | URL |
|--------|-----|
| TechCrunch | `https://techcrunch.com/feed/` |
| Hacker News Jobs | `https://hnrss.org/jobs` |
| Dev.to | `https://dev.to/feed` |
| MIT Technology Review | `https://www.technologyreview.com/feed/` |
| Wired | `https://www.wired.com/feed/latest/rss` |

---

## Agent System Prompt

The AI Agent is instructed to act as a **senior tech journalist**:

- Read ALL articles across all sources
- Group and merge articles covering the same event from different sources
- Cover: AI/ML, new models, tools, startups, funding, layoffs, hiring, backend, devops, web dev, data science, software engineering, big tech moves
- Score each story 1–10 (10 = industry-changing)
- Pick top 7 stories only
- Write a 3-line journalist-style summary: what happened, why it matters, what's next
- Return clean JSON only — no extra text

---

## Setup

### Prerequisites
- [N8N](https://n8n.io/) (Cloud or self-hosted)
- [Groq API key](https://console.groq.com/) (free tier available)
- [Twilio account](https://console.twilio.com/) (free sandbox)

### Steps

**1. Clone and import workflow**
```bash
git clone https://github.com/Krishna-721/daily-tech-digest-n8n
```
Import `workflow.json` into your N8N instance via **Settings → Import Workflow**.

**2. Set up Groq credential**
- In N8N → Credentials → New → Search "Groq"
- Paste your API key from `console.groq.com`

**3. Set up Twilio WhatsApp sandbox**
- Go to `console.twilio.com` → Messaging → Try it out → Send a WhatsApp message
- Send `join <your-sandbox-word>` to `+14155238886` from your WhatsApp
- Note your Account SID and Auth Token

**4. Configure HTTP Request node (Twilio)**

Update the Twilio HTTP Request node with:
```
URL: https://api.twilio.com/2010-04-01/Accounts/YOUR_ACCOUNT_SID/Messages.json
Auth: Basic Auth (SID as username, Auth Token as password)

Body fields:
  From=whatsapp:+14155238886
  To=whatsapp:+91XXXXXXXXXX
  Body={{ $json.message }}
```

**5. Activate workflow**
- Click **Publish** in N8N
- Workflow runs automatically every day at 8AM

---

## Groq Model Config

```
Model: llama-3.3-70b-versatile
Max Tokens: 4096
Temperature: 0.2
```

> Temperature set to 0.2 for consistent, structured JSON output.

---

## Notes

- Groq free tier has a **100k token/day limit** — the XML preprocessor node is essential to stay within limits
- Twilio sandbox sessions expire every **72 hours** — re-send the join message to re-activate
- The workflow fetches fresh RSS data on every run — no database needed

---

## Project Structure

```
daily-tech-digest-n8n/
├── workflow.json       # N8N workflow export (import directly)
└── README.md
```

---

## Future Improvements

- [ ] Add PostgreSQL layer for deduplication across days
- [ ] Text-to-speech via ElevenLabs → audio digest on Telegram
- [ ] Category-based filtering (e.g. only AI news)
- [ ] Multi-recipient support
- [ ] Slack integration as alternative delivery

---

## Author

**Vamshi Krishna**
[GitHub](https://github.com/Krishna-721) · [Portfolio](https://your-portfolio-url.vercel.app)

> Built as part of exploring AI agent workflows and automation pipelines.
