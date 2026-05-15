# PulseWatch — Product Requirements Document (PRD)

## 1. Overview

**Product Name:** PulseWatch  
**Tagline:** Your apps are alive. Know when they’re not.  
**Type:** Self-Hosted Uptime Monitoring Tool  
**Stack:** Node.js · TypeScript · Supabase · Next.js · Telegram Bot API  
**Target Users:** Solo developers, indie hackers, small dev teams with deployed apps  

---

## 2. Problem Statement

Developers deploy apps on Vercel and Render free tiers — but these sleep, crash, or timeout silently. Paid uptime services like Better Uptime or UptimeRobot charge $20+/mo. PulseWatch is a free, self-hostable alternative that any developer can deploy in 10 minutes.

---

## 3. Features

### 3.1 Core (MVP)
- [ ] Add/remove monitored endpoints (URL + name + check interval)
- [ ] HTTP health check with cron (1-min intervals)
- [ ] Response time tracking
- [ ] Telegram + email alerts on down/up events
- [ ] Incident log (when down, how long, resolved at)
- [ ] Public status page (no auth needed)
- [ ] Docker Compose for self-host

### 3.2 Growth
- [ ] SSL certificate expiry alerts
- [ ] Keyword presence check (assert text in response)
- [ ] Status page custom domain
- [ ] Multi-region checks (simulate from Chennai, Mumbai, US)
- [ ] Slack + Discord alert channels
- [ ] Managed cloud option (deploy once, monitor many)

---

## 4. Architecture

```
┌────────────────────────────────────────────────┐
│              Cron Engine (node-cron)             │
│   runs every 1 min → polls all active endpoints  │
└─────────────────┬────────────────────────────────┘
                 │
      ┌─────────┴───────────┐
      │ HTTP GET each URL        │
      │ record: status, latency  │
      └────────┬─────────────┘
               │
       ┌──────┴──────┐
       │              │
    UP ▼           DOWN ▼
  Write log     Write incident
  to Supabase   + fire alert
                   │
         ┌───────┴────────┐
         │               │
   Telegram Bot      Nodemailer
   Alert              Alert

         Supabase DB
  ────────────────────────
  endpoints | checks | incidents

  Next.js Dashboard + Status Page
  ────────────────────────
  /dashboard  (auth, manage endpoints)
  /status     (public, no auth)
```

---

## 5. Database Schema

```sql
endpoints (
  id uuid PK,
  user_id uuid FK,
  name varchar(100),
  url varchar(500),
  method varchar(10),        -- GET | POST
  expected_status integer,   -- 200
  check_interval_min integer, -- 1 | 5 | 10
  is_active boolean,
  created_at timestamptz
)

checks (
  id uuid PK,
  endpoint_id uuid FK,
  status_code integer,
  latency_ms integer,
  is_up boolean,
  checked_at timestamptz
)

incidents (
  id uuid PK,
  endpoint_id uuid FK,
  started_at timestamptz,
  resolved_at timestamptz,
  duration_seconds integer,
  cause varchar(255)
)

alert_channels (
  id uuid PK,
  user_id uuid FK,
  type varchar(20),           -- telegram | email | slack
  config jsonb                -- {"chat_id": "...", "token": "..."}
)
```

---

## 6. Docker Compose (Self-Host)

```yaml
version: '3.8'
services:
  app:
    build: .
    ports: ['3000:3000']
    environment:
      - SUPABASE_URL=${SUPABASE_URL}
      - SUPABASE_KEY=${SUPABASE_KEY}
      - TELEGRAM_BOT_TOKEN=${TELEGRAM_BOT_TOKEN}
      - SMTP_HOST=${SMTP_HOST}
    restart: always
  worker:
    build: ./worker
    environment:
      - SUPABASE_URL=${SUPABASE_URL}
      - SUPABASE_KEY=${SUPABASE_KEY}
    restart: always
```

---

## 7. Tech Stack

| Layer | Technology |
|-------|------------|
| Cron Worker | Node.js + node-cron |
| HTTP Client | axios |
| Alerts | Telegram Bot API + Nodemailer |
| Database | Supabase PostgreSQL |
| Dashboard | Next.js 14 + TypeScript + Tailwind |
| Self-Host | Docker + Docker Compose |
| Deployment | Render.com (managed) |

---

## 8. Monetization

| Tier | Price | Limits |
|------|-------|--------|
| Self-Hosted | Free | Unlimited (you run it) |
| Cloud Free | $0 | 5 endpoints, 5-min checks |
| Cloud Pro | $5/mo | 50 endpoints, 1-min checks, custom status page |
| Cloud Team | $15/mo | 200 endpoints, multi-user, multi-region |

---

## 9. Milestones

| Week | Deliverable |
|------|-------------|
| 1 | Cron poller + Supabase logging |
| 2 | Telegram + email alerts |
| 3 | Next.js dashboard (add/remove endpoints) |
| 4 | Public status page |
| 5 | Docker Compose + README deploy guide |
| 6 | SSL expiry checks + keyword assertions |
