# PulseWatch

> **Your apps are alive. Know when they’re not.** Self-hosted uptime monitor with Telegram alerts and a public status page.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Docker](https://img.shields.io/badge/Docker-ready-blue)](#)

## Features
- ⏱️ 1-minute HTTP health checks
- 📨 Telegram + email instant alerts
- 📊 Response time tracking and incident history
- 🌐 Public status page (zero auth needed)
- 🐳 Docker Compose self-host in 10 minutes
- 🔒 SSL certificate expiry alerts

## Quick Start
```bash
git clone https://github.com/gokulsenthilkumar3/PulseWatch
cp .env.example .env  # fill in your Supabase + Telegram credentials
docker-compose up -d
```

## Docs
See [PRD.md](PRD.md) for full product requirements and architecture.

## License
MIT
