# DealerScope AI 🚗💰

> AI-powered revenue recovery platform for car dealerships — finds missed leads, service upsells, and inventory gaps, then alerts the GM in real time.

**🔴 Live at [dealerscope.app](https://dealerscope.app)**

[![FastAPI](https://img.shields.io/badge/FastAPI-009688?style=flat&logo=fastapi&logoColor=white)](https://fastapi.tiangolo.com)
[![React](https://img.shields.io/badge/React-20232A?style=flat&logo=react&logoColor=61DAFB)](https://react.dev)
[![Railway](https://img.shields.io/badge/Railway-0B0D0E?style=flat&logo=railway)](https://railway.app)
[![Vercel](https://img.shields.io/badge/Vercel-000000?style=flat&logo=vercel)](https://vercel.com)
[![NVIDIA NIM](https://img.shields.io/badge/NVIDIA_NIM-76B900?style=flat&logo=nvidia&logoColor=white)](https://build.nvidia.com)

> **Note:** This is a public showcase repository. The full production codebase is private.  
> Try the live demo at [dealerscope.app](https://dealerscope.app) — login: `demo@dealership.com` / `demo123`

---

## The Problem

Most California car dealerships lose **$15,000–$30,000/month** in recoverable revenue:

- 🔴 **Lost leads** — 20–40% of leads never get a second follow-up call
- 🔴 **Missed service upsells** — repair orders closed without flagging additional needed work  
- 🔴 **Stale inventory** — vehicles sitting 60+ days with no targeted promotion
- 🔴 **CRM gaps** — nobody actively monitoring for dropped opportunities

Enterprise solutions (VinSolutions, DealerSocket) cost $10,000+/month. DealerScope works in 24 hours with a CSV export.

---

## How It Works

```
1. CONNECT    Upload DMS export (Tekion, DealerSocket, CDK, any format)
      │
      ▼
2. ANALYZE    AI scans every lead, RO, and inventory record overnight
      │
      ▼
3. ALERT      GM receives real-time notification with dollar amount + who to call
      │
      ▼
4. RECOVER    Dashboard shows exact recovery actions with one-click follow-up
```

---

## System Architecture

```
┌─────────────────────────────────────────────────────────────────┐
│                    PRODUCTION INFRASTRUCTURE                     │
│                                                                 │
│   User Browser                                                  │
│       │                                                         │
│       ▼                                                         │
│  ┌─────────────┐    HTTPS     ┌──────────────────┐             │
│  │ Vercel CDN  │ ◀──────────▶ │  FastAPI Backend  │             │
│  │ React/Vite  │              │  Railway Cloud    │             │
│  │ Edge Network│              │  Auto-deploy CI/CD│             │
│  └─────────────┘              └────────┬─────────┘             │
│                                        │                        │
│                               Tailscale VPN (encrypted)         │
│                                        │                        │
│                               ┌────────▼─────────┐             │
│                               │  Ubuntu Mini PC   │             │
│                               │  Edge Agent       │             │
│                               │  • Audit runner   │             │
│                               │  • Telegram bot   │             │
│                               │  • Cron pipelines │             │
│                               │  • 24/7 systemd   │             │
│                               └──────────────────┘             │
└─────────────────────────────────────────────────────────────────┘
```

**Key design decisions:**
- **Railway** for backend — zero-downtime auto-deploy on every `git push`
- **Vercel CDN edge** for frontend — served from 30+ global PoPs, sub-100ms load
- **Ubuntu mini PC as edge agent** — always-on via systemd lingering, handles all cron pipelines
- **Tailscale VPN mesh** — encrypted node-to-node communication, no open ports on home network

---

## Tech Stack

| Layer | Technology |
|-------|-----------|
| Backend | Python 3.11 · FastAPI · uvicorn |
| Frontend | React 18 · Vite · Tailwind CSS |
| AI/ML | NVIDIA NIM API · Kimi K2.5 LLM |
| Cloud | Railway (backend) · Vercel (frontend) |
| Networking | Tailscale VPN · GitHub CI/CD |
| Notifications | Telegram Bot API |
| Security | slowapi · JWT · bcrypt · CORS · input sanitization |

---

## Security Implementation

```python
# CORS locked to production domain only
app.add_middleware(
    CORSMiddleware,
    allow_origins=[
        "https://dealerscope.app",
        "https://www.dealerscope.app"
    ],
    allow_credentials=True,
    allow_methods=["GET", "POST", "PUT", "DELETE"],
    allow_headers=["Authorization", "Content-Type"],
)

# Rate limiting — blocks brute force and abuse
limiter = Limiter(key_func=get_remote_address)
app.state.limiter = limiter

# Security headers on every response
response.headers["X-Content-Type-Options"] = "nosniff"
response.headers["X-Frame-Options"] = "DENY"
response.headers["Strict-Transport-Security"] = "max-age=31536000"

# Input sanitization — strips XSS and injection patterns
def sanitize_string(value: str, max_length: int = 500) -> str:
    value = value.strip()[:max_length]
    value = re.sub(r'[<>"\']', '', value)
    return value
```

---

## Universal CSV Adapter

One of the core technical challenges — every dealership exports data differently. The adapter handles any format:

```python
# Simplified example of the column mapping logic
FIELD_MAPPINGS = {
    "name": ["customer_first_name", "first_name", "name", "customer"],
    "email": ["customer_email", "email", "contact_email"],
    "phone": ["customer_phone", "phone_number", "mobile", "cell"],
    "vehicle": ["vehicle_interest", "vehicle_of_interest", "car_wanted"],
    "status": ["lead_status", "opportunity_stage", "status"],
}

def detect_columns(headers: list) -> dict:
    """
    Map unknown CSV headers to canonical field names.
    Falls back to LLM inference for ambiguous columns.
    """
    mapping = {}
    for canonical, variants in FIELD_MAPPINGS.items():
        for header in headers:
            if any(v in header.lower() for v in variants):
                mapping[canonical] = header
                break
    return mapping
```

Supports: **Tekion · DealerSocket · CDK · Fullpath · Reynolds & Reynolds · Any custom format**

---

## Dashboard Features

| Feature | Description |
|---------|-------------|
| 💰 Revenue Recovery Feed | $18,770 total opportunity displayed in real time |
| 👥 Lost Leads | Add, recover, filter leads — duplicate detection built in |
| 🚗 Hot Inventory | Scan for stale vehicles, generate marketplace URLs |
| 🔧 Service Department | Flag missed upsells, KPI cards per department |
| 🔔 Alerts & Gaps | Resolve/reopen toggle, severity filtering |
| 📁 Integrations Hub | Drag-and-drop CSV import, universal adapter |
| 📋 Activity Log | Full audit trail with timestamp and user on every event |
| ⚙️ Settings | Dark/light mode, notification preferences, profile management |

---

## Business Model

| Tier | Monthly | Features |
|------|---------|---------|
| Starter | $999 | CSV import, weekly audits, Telegram alerts |
| Professional | $2,499 | Live DMS API, multiple logins, custom alerts |
| Enterprise | $4,999 | Multi-location, dealer groups |
| White Label | Custom | Resell under dealer's brand |

Free 30-day pilot for California dealerships — no credit card, no commitment.

---

## Try It Live

**[dealerscope.app](https://dealerscope.app)**

```
Email:    demo@dealership.com
Password: demo123
```

All 9 dashboard features fully unlocked. California-themed demo data preloaded.

---

## Screenshots

> *Coming soon — live demo available at [dealerscope.app](https://dealerscope.app)*

---

## Built By

**Rudra Patel** — 18-year-old Solo Founder · Eastvale, CA  
Data Science Student (Santiago Canyon College → UCSD/UCSB Fall 2026) · GPA 3.57

[🔗 LinkedIn](https://linkedin.com/in/rudra-patel-a0a115354) · 
[📧 rcppatel24@gmail.com](mailto:rcppatel24@gmail.com) · 
[📱 (657) 258-7212](tel:6572587212)

---

*Full production codebase is private. For technical deep-dives or partnership inquiries, reach out directly.*
