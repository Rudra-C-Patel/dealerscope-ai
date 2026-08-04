# DealerScope AI

> Revenue intelligence for car dealerships. Reads your DMS and CRM exports, finds the revenue already sitting in them, and tells you who to call.

**Live at [dealerscope.app](https://dealerscope.app)**

[![FastAPI](https://img.shields.io/badge/FastAPI-009688?style=flat&logo=fastapi&logoColor=white)](https://fastapi.tiangolo.com)
[![React](https://img.shields.io/badge/React_19-20232A?style=flat&logo=react&logoColor=61DAFB)](https://react.dev)
[![Railway](https://img.shields.io/badge/Railway-0B0D0E?style=flat&logo=railway)](https://railway.app)
[![Vercel](https://img.shields.io/badge/Vercel-000000?style=flat&logo=vercel)](https://vercel.com)
[![NVIDIA NIM](https://img.shields.io/badge/NVIDIA_NIM-76B900?style=flat&logo=nvidia&logoColor=white)](https://build.nvidia.com)

> **This is a public showcase repository.** The production codebase is private.
> Try the live demo at [dealerscope.app](https://dealerscope.app) with `demo@dealership.com` / `demo123`.

---

## The problem

A dealership's CRM and DMS already contain the revenue. Nobody has time to go find it:

- Leads that got one call and then nothing
- Service work a customer declined, with no follow-up ever scheduled
- Leases running to term with no outreach
- Customers who quietly stopped coming in for service and never complained

Enterprise tools in this category are priced for dealer groups and take weeks to integrate. DealerScope works off a CSV export.

## How it works

```
1. CONNECT   Upload a DMS or CRM export (Tekion, DealerSocket, CDK, or any format)
     |
     v
2. ANALYZE   Adapter normalizes the columns, then every lead, RO, and lease is scored
     |
     v
3. ALERT     Findings surface on the dashboard and via Telegram, each with a dollar figure
     |
     v
4. RECOVER   Each finding carries a generated call script and the record it came from
```

---

## Architecture

```
        User browser
             |
             v
   +------------------+        HTTPS       +----------------------+
   |   Vercel edge    | <----------------> |   FastAPI backend    |
   |  React 19 / Vite |                    |   Railway            |
   |  auto-deploy     |                    |   auto-deploy        |
   +------------------+                    +----------+-----------+
                                                      |
                                           +----------v-----------+
                                           |  NVIDIA NIM API      |
                                           |  z-ai/glm-5.2        |
                                           |  call script gen     |
                                           +----------------------+
```

Both deploys are git-connected and fire on merge to `main`. Storage is file-based JSON on a mounted Railway volume.

---

## Stack

| Layer | Technology |
|---|---|
| Backend | Python 3.11, FastAPI, uvicorn |
| Frontend | React 19.2, Vite 8, React Router v7 |
| AI | NVIDIA NIM API, `z-ai/glm-5.2` |
| Hosting | Railway (backend), Vercel (frontend) |
| Storage | File-based JSON on a persistent volume |
| Notifications | Telegram Bot API, SMTP |
| Auth | Bearer token, bcrypt via passlib |

---

## Security

What actually ships:

- **Password hashing** with bcrypt (`passlib`), including transparent rehash of legacy SHA-256 hashes on next successful login.
- **Custom rate limiter** on login, registration, password change, and the deep health probe. Per-IP, sliding window.
- **Security headers** on every response: `X-Content-Type-Options: nosniff`, `X-Frame-Options: DENY`, `Strict-Transport-Security` with `includeSubDomains`.
- **CORS** restricted to the production domains plus local dev origins.
- **Admin and internal endpoints fail closed.** Every `/admin/*` and `/internal/*` route gates on an admin token and returns 403 when that token is unset, rather than falling back to a default.
- **500 responses never include exception text.** Caught exceptions log server-side with context and return a generic message.
- **No secrets in source.** Credentials come from environment variables only.

---

## The universal CSV adapter

The single hardest product problem. Every dealership exports data differently, and no two DMS platforms agree on a column name.

```python
# Simplified illustration of the mapping layer
FIELD_MAPPINGS = {
    "name":    ["customer_first_name", "first_name", "name", "customer"],
    "email":   ["customer_email", "email", "contact_email"],
    "phone":   ["customer_phone", "phone_number", "mobile", "cell"],
    "vehicle": ["vehicle_interest", "vehicle_of_interest", "car_wanted"],
    "status":  ["lead_status", "opportunity_stage", "status"],
}

def detect_columns(headers: list) -> dict:
    """Map unknown CSV headers to canonical field names.
    Ambiguous columns fall through to LLM inference."""
    mapping = {}
    for canonical, variants in FIELD_MAPPINGS.items():
        for header in headers:
            if any(v in header.lower() for v in variants):
                mapping[canonical] = header
                break
    return mapping
```

Uploads are validated before they are stored. A malformed file returns a 400 naming the missing columns and leaves the previous run untouched, rather than silently producing a zero-finding result.

---

## Not inventing numbers

An analytics product that fabricates figures is worse than no product. Two design rules follow from that:

- **The close-probability model refuses to guess.** It is Naive Bayes trained on the store's own closed-versus-lost leads. Below 20 closed deals it returns stated industry priors with an explicit disclaimer instead of a fabricated per-lead score.
- **A test enforces it.** `test_no_fabricated_data.py` fails the build if an endpoint starts returning figures that no underlying data supports. Modules that are not built report themselves as not built.

---

## Dashboard

| Feature | Description |
|---|---|
| Revenue recovery feed | Findings ranked by dollar value, each traceable to source records |
| Lost leads | Filter, action, and recover leads with duplicate detection |
| Close probability | Per-lead score with the top contributing factors in plain English |
| Service department | Declined work and missed upsells, with per-advisor KPIs |
| Lease and defection | Lease-end timing and customers who quietly stopped coming in |
| Alerts and gaps | Severity filtering with resolve and reopen |
| Integrations hub | Drag-and-drop CSV import through the universal adapter |
| Activity log | Full audit trail, timestamped per event |

---

## Try it

**[dealerscope.app](https://dealerscope.app)**

```
Email:    demo@dealership.com
Password: demo123
```

Read-only demo account, preloaded with a fully worked example dealership.

---

## Pricing

Contact for pricing. Pilot programs available for California dealerships.

---

## Built by

**Rudra Patel**, solo founder, Eastvale CA.
Transferring to UC San Diego, Fall 2026, Cognitive Science with an ML / Neural Computation focus.

[LinkedIn](https://linkedin.com/in/rudra-patel-a0a115354) · [rcppatel24@gmail.com](mailto:rcppatel24@gmail.com)

*Production codebase is private. For technical deep-dives or partnership inquiries, reach out directly.*
