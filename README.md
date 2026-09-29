<p align="center">
  <img src="https://img.shields.io/badge/SilentSLA-Autonomous%20SLA%20Agent-6366f1?style=for-the-badge&labelColor=000" alt="SilentSLA" />
</p>

<h1 align="center">SilentSLA</h1>

<p align="center">
  <b>The autonomous agent that turns forgotten micro-outages into recovered revenue.</b>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/TypeScript-007ACC?style=flat-square&logo=typescript&logoColor=white" />
  <img src="https://img.shields.io/badge/Express.js-v5-000?style=flat-square&logo=express&logoColor=white" />
  <img src="https://img.shields.io/badge/Prisma-2D3748?style=flat-square&logo=prisma&logoColor=white" />
  <img src="https://img.shields.io/badge/Google%20Gemini-886FBF?style=flat-square&logo=google&logoColor=white" />
  <img src="https://img.shields.io/badge/Zod-3E67B1?style=flat-square&logo=zod&logoColor=white" />
  <img src="https://img.shields.io/badge/Tailwind%20CSS-06B6D4?style=flat-square&logo=tailwindcss&logoColor=white" />
</p>

---

## 🧠 The Problem

When a cloud service fails, engineers fix it. But the **SLA contract** — which entitles you to service credits — lives in a completely different world: Finance and Procurement.

Small "micro-outages" (5 min here, 10 min there) **accumulate silently**, breach contractual thresholds, and are **forgotten** because proving them requires tedious, cross-team, manual effort.

> **Result:** Companies leave thousands of dollars in service credits unclaimed every year.

## 💡 The Solution

**SilentSLA** is a memory-driven autonomous agent that sits between your observability tools (Datadog, PagerDuty) and your vendor SLA contracts. It:

1. **Watches** — Ingests incident events via webhooks silently
2. **Remembers** — Accumulates downtime facts in a relational database across billing windows
3. **Evaluates** — Uses deterministic math (never AI) to detect when cumulative downtime breaches the SLA threshold
4. **Acts** — Autonomously triggers an AI-powered claim generator the instant a breach is detected
5. **Recovers** — Produces a professional, evidence-backed service-credit claim ready for vendor submission

**No human intervention. No forgotten outages. No lost revenue.**

---

## 🏗️ Architecture

```
┌─────────────┐     ┌──────────────┐     ┌───────────────┐     ┌──────────────┐
│  Webhook /   │────▶│  Ingestion   │────▶│   Hindsight   │────▶│   Policy     │
│  Dashboard   │     │  API Layer   │     │   Memory DB   │     │  Evaluator   │
└─────────────┘     └──────────────┘     └───────────────┘     └──────┬───────┘
                                                                       │
                                                              ┌───────▼───────┐
                                                              │  Threshold    │
                                                              │  Breached?    │
                                                              └───┬───────┬───┘
                                                            No │       │ Yes
                                                               ▼       ▼
                                                          ┌────────┐ ┌──────────────┐
                                                          │ Store  │ │ AI Claim      │
                                                          │ & Wait │ │ Generator     │
                                                          └────────┘ │ (Gemini)      │
                                                                     └──────┬───────┘
                                                                            ▼
                                                                     ┌──────────────┐
                                                                     │ Claim Package │
                                                                     │ → Vendor API  │
                                                                     └──────────────┘
```

### Key Design Decisions

| Decision | Reasoning |
|---|---|
| **Deterministic math for breach detection** | LLMs are unreliable for arithmetic. All downtime aggregation and threshold checks are pure TypeScript. |
| **AI only for text synthesis** | Gemini writes the claim package and extracts contract terms — it never makes the breach decision. |
| **Relational DB over Vector DB** | SLA tracking needs exact time-window aggregations (`SUM(duration) WHERE month = X`), not semantic search. |
| **Event-driven, not conversational** | This is not a chatbot. Value comes from autonomous background state transitions. |

---

## 🛠️ Tech Stack

| Layer | Technology | Why |
|---|---|---|
| **Runtime** | Node.js + TypeScript (strict) | Type safety for financial calculations |
| **API** | Express.js v5 | Fast iteration, excellent JSON ecosystem |
| **Database** | Prisma ORM + SQLite | Strict typing for SLA/Incident models; relational queries |
| **AI** | Google Gemini (`@google/genai`) | Structured JSON generation for claim packages |
| **Validation** | Zod | Schema enforcement on all API and LLM boundaries |
| **Testing** | Vitest | Unit tests for deterministic math/time logic |
| **Frontend** | Tailwind CSS (dark glassmorphism) | Professional enterprise dashboard |

---

## 🚀 Quick Start

### Prerequisites
- **Node.js** ≥ 18
- **Google Gemini API Key** ([Get one here](https://aistudio.google.com/apikey))

### Setup

```bash
# Clone the repository
git clone https://github.com/mohdaliazam/silentSLA.git
cd silentSLA

# Install dependencies
npm install

# Configure environment
cp .env.example .env
# Edit .env and add your GEMINI_API_KEY

# Initialize the database
npx prisma db push

# Start the development server
npm run dev
```

Open **http://localhost:3000** and you're live!

---

## 🎮 Demo Walkthrough

### Step 1 — Extract an SLA Contract
Go to the **Claim Simulator** tab and paste raw contract text:

```
AWS agrees to provide 99.9% monthly uptime for the Relational Database 
Service (RDS). Downtime exceeding the allowed threshold entitles the 
customer to proportional service credits.
```

Click **"Process Contract"** — Gemini extracts the structured SLA policy automatically.

### Step 2 — Inject Micro-Outages
Switch to the **Telemetry Vault** tab and click **"Inject 15m Anomaly"** multiple times. Watch the downtime accumulate against the threshold.

### Step 3 — Watch the Autonomous Breach Detection
After ~43 minutes of cumulative downtime (the threshold for 99.9% on a 30-day month), the system **autonomously generates an AI-drafted claim package**.

### Step 4 — Submit to Vendor
Go to the **Claim Simulator** tab and click **"Execute Autonomous Vendor Negotiation"** to simulate vendor submission.

---

## 📡 API Reference

| Method | Endpoint | Description |
|---|---|---|
| `POST` | `/api/policies` | Register an SLA policy |
| `GET` | `/api/policies` | List all SLA policies |
| `POST` | `/api/extract-policy` | AI-extract SLA from raw contract text |
| `POST` | `/api/incidents` | Ingest incident → auto-evaluate → auto-claim |
| `GET` | `/api/incidents` | List all incidents |
| `GET` | `/api/claims` | List all claims with related policies |
| `POST` | `/api/vendor/submit-claim` | Simulate vendor claim submission |
| `POST` | `/api/reset` | Reset all data (demo) |

<details>
<summary><b>Example: Ingest an Incident</b></summary>

```bash
curl -X POST http://localhost:3000/api/incidents \
  -H "Content-Type: application/json" \
  -d '{
    "serviceName": "Core Database",
    "startTime": "2026-09-29T10:00:00Z",
    "endTime": "2026-09-29T10:15:00Z",
    "evidenceSource": "Datadog Monitor #1123"
  }'
```

**Response (breach detected):**
```json
{
  "incident": { "id": "...", "durationMinutes": 15 },
  "claim": {
    "id": "...",
    "claimPackage": "SERVICE CREDIT CLAIM: Based on SLA target of 99.9%...",
    "status": "GENERATED"
  },
  "breach": true
}
```
</details>

---

## 🗄️ Database Schema

```
┌─────────────────┐       ┌─────────────────┐       ┌─────────────────┐
│    SLAPolicy     │       │    Incident      │       │     Claim        │
├─────────────────┤       ├─────────────────┤       ├─────────────────┤
│ id          (PK)│◀──┐   │ id          (PK)│       │ id          (PK)│
│ serviceName (UK)│   ├──▶│ slaPolicyId (FK)│   ┌──▶│ slaPolicyId (FK)│
│ vendor          │   │   │ startTime       │   │   │ totalDowntime   │
│ availTarget     │   │   │ endTime         │   │   │ claimPackage    │
│ measurePeriod   │   │   │ durationMinutes │   │   │ status          │
│                 │───┘   │ evidenceSource  │   │   │ createdAt       │
│                 │───────│ isExcluded      │   │   │                 │
│                 │       └─────────────────┘   │   └─────────────────┘
│                 │─────────────────────────────┘
└─────────────────┘
```

---

## 🧪 Running Tests

```bash
# Unit tests (evaluator math)
npx vitest run

# End-to-end simulation (requires server running)
npx tsx scripts/test-e2e.ts
```

---

## 📂 Project Structure

```
src/
├── index.ts              # Express server + REST API endpoints
├── evaluator.ts           # Deterministic breach detection engine
├── evaluator.test.ts      # Unit tests for math/time logic
└── claimGenerator.ts      # Gemini-powered claim text generation
prisma/
├── schema.prisma          # Database models (SLAPolicy, Incident, Claim)
└── dev.db                 # SQLite database
public/
├── index.html             # Dashboard UI (Tailwind CSS)
└── app.js                 # Frontend interaction logic
scripts/
└── test-e2e.ts            # E2E integration simulation
```

---

## 🤝 Team

Built with ❤️ for the hackathon.

---

## 📄 License

ISC
