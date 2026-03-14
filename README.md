# AI-Automation-Projects-Python-Django-AI-APIs
Portfolio of backend systems, automation workflows, and AI-powered applications deployed on Linux using Docker.
# OpenClaw — AI Lead Automation System

> Production-deployed backend automation platform that captures, classifies, and nurtures leads through AI-powered workflows — built and maintained solo.

[![Live](https://img.shields.io/badge/Live-rosseliodigital.online-brightgreen)](https://rosseliodigital.online)
[![Stack](https://img.shields.io/badge/Stack-Django%20%7C%20MongoDB%20%7C%20Docker-blue)]()
[![AI](https://img.shields.io/badge/AI-Claude%20Haiku%20%7C%20Llama%203.2-purple)]()
[![Uptime](https://img.shields.io/badge/Uptime-Monitored%2024%2F7-success)]()

---

## The Problem

Most small businesses lose leads because:
- Responses are too slow (hours, not seconds)
- Follow-up emails are manual and forgotten
- There's no system tracking where each lead is in the pipeline

## The Solution

OpenClaw is a backend automation system I built from scratch that:
- **Captures leads** from Telegram bot interactions and a website chat widget
- **Classifies messages** using Claude AI to determine intent and routing
- **Fires automated email sequences** (Day 0 → Day 10) tailored to lead type
- **Notifies the operator** via Telegram with real-time alerts
- **Tracks everything** in MongoDB with a Django admin dashboard

---

## Architecture

```
Telegram Bot (@RosselioDigitalBot)
        ↓
Cloudflare Tunnel  ←→  webhook.rosseliodigital.online
        ↓
Django (Dockerized) — webhook handler
  ├── /inquiry       → lead saved → admin notified
  ├── /live_agent    → AI chat activates (Claude Haiku → OpenRouter → Ollama)
  └── /start, /help  → instant responses
        ↓
MongoDB — Lead storage, conversation history, email logs
        ↓
Email Sequences (via Resend API)
  ├── job_hunt      — Day 0–10 outreach
  ├── general_va    — VA services pitch
  └── warm_outreach — warm contact nurture
        ↓
Telegram Operator Bot (@RosselioDailyBot)
  ├── 8AM digest — pipeline counts, job tracker, reminders
  └── Commands: /health /status /leads /quota /backup /restart
```

---

## Features

### Lead Capture & Classification
- Telegram bot with inline keyboard — service menu, inquiry flow, live agent mode
- AI chat powered by **Claude Haiku** (primary) → **OpenRouter llama-3.3-70b** (fallback) → **Ollama** (offline fallback)
- Exponential backoff retry logic — 3 attempts, 1s/2s/4s intervals
- Duplicate lead detection — email deduplication on every import

### Automated Email Sequences
- 3 active niche sequences (job_hunt, general_va, warm_outreach) — Day 0 to Day 10
- 5 seasonal campaigns (Women's Month, Graduation, Lenten, Mother's Day, Father's Day) — auto-fire on active dates
- Proper HTML email with responsive layout, social footer, unsubscribe link
- Resend API integration — SPF/DKIM/DMARC all passing, Gmail inbox delivery confirmed

### Operator Intelligence (Personal AI Assistant Bot)
- Daily 8AM digest — pipeline counts, job applications, custom reminders
- Real-time server commands via Telegram: disk, RAM, CPU, container status, logs
- Natural language routing — Filipino and English both understood
- Quota monitoring — alerts at 70% daily and 50 remaining monthly

### Infrastructure & Reliability
- Dockerized — 6 containers (Django, Django-cron, MongoDB, Ollama, gateway, Uptime Kuma)
- Cloudflare Tunnel — no exposed ports, HTTPS everywhere
- WSL failover system — laptop automatically takes over if VPS goes down, reverts when server recovers
- Automated backup every 2 days — MongoDB dump + .env + GitHub push
- Uptime Kuma monitoring — Telegram alerts on any service down
- Windows Task Scheduler watchdog — silent VBS script, restarts failover watcher if dead

### Local RAG Assistant (WSL)
- Offline AI assistant powered by **Ollama + llama3.2** — no API cost
- Fed with personal resume, system docs, and operations runbook
- `ops` command from PowerShell — suggests and auto-runs server commands
- `analyze` command — pastes a job description, returns fit score + gaps + cover letter

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Backend | Python 3.11, Django 4.x, Django REST Framework |
| Database | MongoDB 6 (via Djongo ORM) |
| AI — Primary | Anthropic Claude Haiku |
| AI — Fallback | OpenRouter (meta-llama/llama-3.3-70b-instruct) |
| AI — Offline | Ollama (llama3.2) — self-hosted |
| Email | Resend API |
| Messaging | Telegram Bot API |
| Containers | Docker, Docker Compose |
| Tunnel | Cloudflare Tunnel |
| Monitoring | Uptime Kuma |
| VPS | Ubuntu Server (Linux) |
| Local Dev | WSL2 AlmaLinux-9 |

---

## Deployment

```bash
# All services run via Docker Compose
docker compose -f ~/openclaw/docker-compose.yml up -d

# 6 containers:
# openclaw-django-1        → main API + webhooks (port 8000)
# openclaw-django-cron-1   → scheduled jobs (email sequences, campaigns)
# openclaw-mongodb-1        → database (port 27017)
# openclaw-ollama-1         → local AI (port 11434)
# openclaw-openclaw-gateway → service gateway
# openclaw-uptime-kuma-1    → monitoring dashboard (port 3001)
```

Cloudflare Tunnel routes `webhook.rosseliodigital.online` → Django — no port exposure, free SSL.

---

## Cron Schedule

```
8AM PHT daily    → Lead pipeline digest sent to operator Telegram
9AM PHT daily    → Email sequences fire (max 50/day — well under free tier limit)
9AM PHT daily    → Seasonal campaigns fire if active date window matches
Every hour       → Resend quota check — Telegram alert if approaching limit
Every 2 days 2AM → MongoDB backup + .env backup + GitHub push
```

---

## Lead Pipeline

Leads enter from:
1. **Telegram bot** `/inquiry` or `/live_agent` commands
2. **Excel import** — bulk import via `import_leads.sh` with dry-run confirmation
3. *(Planned)* Zoho CRM webhook → AI classification → auto-sequence

Each lead is tracked by: name, email, niche, sequence day, email status, company, role, industry, location, notes.

---

## Failover System

If the VPS goes down:

```
Watcher (WSL) detects 3 consecutive failures
        ↓
failover_start.sh activates
  → Syncs latest lead data to WSL
  → Starts Django + Ollama on WSL via Docker
  → Activates Cloudflare Tunnel from laptop
  → Redirects both Telegram bots to WSL
  → Sends Telegram alert: "OpenClaw Failover ACTIVE"
        ↓
Both bots continue responding from laptop
        ↓
Server recovers → 3/3 confirmations
        ↓
failover_stop.sh reverts everything back to server
  → Sends Telegram alert: "OpenClaw Back on Server — XX minutes"
```

Tested and confirmed working — 43-minute failover cycle, zero manual intervention.

---

## NDA Notice

This system was built for **Rosselio Digital** — a personal brand and freelance automation business. All client data, lead contact information, and business-specific configurations are excluded from this repository. Code shown represents system architecture, automation logic, and AI integration patterns only.

Some automation techniques and workflow designs were developed during client engagements under NDA. Implementation details shown here are original work built for this personal system.

---

## Screenshots

```
screenshots/
  chat_widget.png          ← Telegram bot in action
  lead_capture.png         ← Django admin lead view
  email_sequence.png       ← Email sequence in inbox
  docker_deployment.png    ← docker compose ps output
  uptime_kuma.png          ← monitoring dashboard
  daily_bot.png            ← 8AM digest in Telegram
  failover_active.png      ← WSL failover Telegram alert
```

*(Screenshots folder — add before presenting to recruiter)*

---

## Live Demo

**Bot:** Search `@RosselioDigitalBot` on Telegram → type `/start`

**Website:** [rosseliodigital.online](https://rosseliodigital.online)

**Monitoring:** Available on request

---

## About

Built, deployed, and maintained by **Rosselio M. Andres** — Python/Django developer and AI automation specialist based in the Philippines.

- Portfolio: [rosseliodigital.online](https://rosseliodigital.online)
- GitHub: [github.com/digitalrosselio-coder](https://github.com/digitalrosselio-coder)
- Email: contact@rosseliodigital.online
