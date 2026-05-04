# 🤖 AI-Infused SDLC Customer Support Agent (CSAgent)

> A functional prototype showcasing an AI-powered support pipeline that replaces manual workflows with intelligent automation, audit tracking, and human-in-the-loop supervision.
---

## Overview

The **AI-Infused SDLC Customer Support Agent (CSAgent)** is a fully autonomous **17-agent AI pipeline** that acts as the _"Customer-Facing Brain"_ within an Agentic SDLC ecosystem. It transforms how customer support is delivered across ticket handling, communication, escalation, and resolution workflows.

CSAgent receives tickets, categorizes them, enforces Service Level Agreements (SLAs), drafts resolutions, and directly coordinates with backend engineering bots (DevOps, QA, SRE, BA, PM) to deploy fixes — all while keeping a human in control of sensitive decisions through mandatory **Human-in-the-Loop (HIL)** gates.

---

## Key Features

| Feature | Description |
|---|---|
| 🌐 **Omnichannel Intake** | Ingests tickets from email, chat, portal, Teams, Slack, WhatsApp, phone transcripts, and API feeds |
| 🧠 **Intelligent Triage** | NLP + sentiment analysis classifies type, severity, urgency, and priority (P1–P4) with ≥85% accuracy |
| 📋 **Full Lifecycle Management** | Manages tickets from `New` → `Acknowledged` → `In Triage` → ... → `Customer Confirmed` → `Closed` |
| ⏱️ **SLA Governance** | Tracks first response time, resolution time, fires warnings at 75% and critical alerts at 90% of SLA window |
| 🔍 **Semantic Deduplication** | Qdrant-powered duplicate detection with ≥90% accuracy; links duplicates to a Master Ticket |
| 🔗 **Incident Linkage** | Auto-links tickets matching active SRE incidents and informs customers of known incident status |
| 🧬 **RAG-Powered Resolution** | Retrieves KB articles and historical resolutions to provide first-contact resolution ≥35% by month 6 |
| 🛑 **HIL Enforcement** | Physically blocks financial commitments, legal correspondence, and VIP interactions without human approval |
| 🔒 **PII Redaction** | Auto-detects and redacts sensitive credentials (payment cards, passwords) from all customer messages |
| 📊 **Analytics & Reporting** | Generates 5 automated reports: Daily Digest, SLA Compliance, CSAT, Ticket Ageing, Voice of Customer |
| 🗄️ **Zero-Bleed Multi-Tenancy** | Strict project-level data isolation across Qdrant, Redis, and PostgreSQL |

---

## Tech Stack

| Layer | Technology |
|---|---|
| **Frontend** | React, Vite |
| **Backend** | Python 3.11+, FastAPI |
| **AI Orchestration** | LangGraph (Stateful Agent Pipeline, 17 nodes) |
| **LLM** | Gemini / GPT (configurable) |
| **Vector Database** | Qdrant (Semantic Search, KB RAG, Deduplication) |
| **State & Caching** | Redis (LangGraph state checkpointing, rate limiting, SQL cache) |
| **Relational Database** | PostgreSQL (with Row-Level Security for project isolation) |
| **ITSM Integration** | Jira Service Management / ServiceNow / Zendesk / Freshdesk |
| **CRM Integration** | Salesforce / HubSpot |

---

## Architecture

```
Customer Input (Web Portal / Phase 4: Email, Slack, Teams, WhatsApp)
         │
         ▼
  [FastAPI Backend]  ──────  [Redis Cache & State Store]
         │
         ▼
  [LangGraph StateGraph - 17 Agent Nodes]
   │
   ├─ AG-13: PII Redaction
   ├─ AG-16: Config Validation      ◄─── Qdrant (SLA Rules & KB)
   ├─ AG-01: Intake & Language
   ├─ AG-02: Triage & Classification
   ├─ AG-06: SLA Clocks
   ├─ AG-11: Ticket Splitting
   ├─ AG-03: Master Ticket Linkage  ◄─── Qdrant (Semantic Dedup)
   ├─ AG-08: Escalation Guard       ◄─── HIL Gate (pauses graph)
   ├─ AG-04: Routing
   ├─ AG-09: SDLC Gate              ◄─── Waits for DevOps + QA Webhooks
   ├─ AG-05: KB Resolution          ◄─── Qdrant RAG
   ├─ AG-07: Communication
   ├─ AG-10: KB Generation          ◄─── HIL-5 Gate
   ├─ AG-12: Analytics & Reporting
   ├─ AG-14: Loop Detection
   ├─ AG-15: Cold Start
   └─ AG-17: Data Consent & Retention

         │
         ▼
  [ITSM / CRM / Channel APIs]  ──  [Audit Trail / Context Store]
```

### Data Isolation (Zero-Bleed Architecture)

- **Qdrant:** Separate collections per project (`project_alpha_kb`, `project_beta_kb`) with strict `project_id` payload filtering.
- **Redis:** Key prefixes (`proj_{id}:state:{ticket_id}`) isolate LangGraph state, rate limits, and caches.
- **PostgreSQL:** Row-Level Security (RLS) tied to the authenticated user's `project_id` JWT claim.

---

## The 17 AI Agents

| Agent ID | Name | Role |
|---|---|---|
| AG-01 | Intake & Language | Parses inbound messages, normalises to unified ticket schema, detects language |
| AG-02 | Triage & Classification | NLP + sentiment → priority (P1-P4), ticket type, product area |
| AG-03 | Master Ticket Linkage | Qdrant semantic dedup (≥90%); links to SRE incidents; flags conflicts for HIL |
| AG-04 | Routing | Directs bugs to SRE/DevOps, feature requests to BA/PM agents |
| AG-05 | KB Resolution | RAG pipeline for First-Contact Resolution using Admin-uploaded KB docs |
| AG-06 | SLA Clocks | Assigns and tracks SLA deadlines by customer tier and priority |
| AG-07 | Communication | Generates professional, jargon-free customer responses; formats per channel |
| AG-08 | Escalation Guard | Enforces HIL-3/4 gates for billing, legal, VIP, angry/at-risk customers |
| AG-09 | SDLC Gate | Pauses graph; awaits dual-confirmation from DevOps + QA webhooks before resolution |
| AG-10 | KB Generation | Auto-drafts new KB articles from resolved tickets; routes to HIL-5 for publication |
| AG-11 | Ticket Splitting | Splits complex multi-issue tickets into independent child sub-graphs |
| AG-12 | Analytics | Scheduled workers generating R-01 through R-05 reports |
| AG-13 | PII Redaction | Detects and redacts payment cards, passwords, sensitive credentials |
| AG-14 | Loop Detection | Prevents infinite routing loops between engineering bots |
| AG-15 | Cold Start | Initialises agent context for new project workspaces |
| AG-16 | Config Validation | Validates SLA configs against Qdrant-embedded Admin-uploaded reference docs |
| AG-17 | Data Consent & Retention | Enforces 7-year data archival/deletion policies and GDPR/CCPA compliance |

---

## User Roles & RBAC

| Role | Access Level | Key Permissions |
|---|---|---|
| **System Admin / Support Ops** | Full Admin | Initialize workspace, provision users, manage ITSM/CRM integrations, upload SLA docs, approve HIL-1 configs |
| **Support Agent / Support Lead** | Standard | View assigned queues (UI-01), ticket details, SLA dashboard (UI-02), monitor HIL escalation status, submit KB drafts |
| **Support Manager / CSM** | Elevated | All Agent permissions + approve HIL-3/4 escalations, manage agent configs, publish KB articles (HIL-5) |
| **VP Customer Success** | Executive + Override | Executive dashboard (UI-11), override escalation recommendations, approve SLA exceptions, manage VIP customers |
| **Legal / Compliance Team** | Read + Legal Override | View legal-flagged tickets, take ownership of legal correspondence, approve/block legal communications |

---

## Human-in-the-Loop (HIL) Checkpoints

CSAgent is physically blocked from bypassing these gates:

| Checkpoint | Trigger | Reviewer |
|---|---|---|
| **HIL-1** (Config) | New SLA config, routing rules, or communication template before going live | System Admin / Support Ops Manager |
| **HIL-3/4** (Critical Escalation) | Billing disputes, legal issues, VIP customers, angry/at-risk customers, P1/P2 closure | Support Manager / CSM |
| **HIL-5** (KB Publication) | AI-drafted KB articles before publishing to the customer-facing portal | Support Engineer + Product SME |
| **Dual-Confirmation Gate** | "Fix Deployed" message to customer requires webhooks from both DevOps Agent AND QA/SRE Agent | Automated (both signals required) |

---

## Target KPIs (Arjuna Metrics)

| Metric | Target |
|---|---|
| P1 First Response SLA | 15 minutes |
| P2 First Response SLA | 1 hour |
| P3 First Response SLA | 4 hours |
| P4 First Response SLA | 1 business day |
| Resolution SLA Compliance | ≥90% overall, ≥95% Enterprise |
| CSAT Score | ≥4.2 / 5.0 |
| First Contact Resolution (Month 6) | ≥35% |
| First Contact Resolution (Month 12) | ≥45% |
| Triage Classification Accuracy | ≥85% |
| Duplicate Detection Accuracy | ≥90% |
| Proactive Update Compliance | ≥95% |
| Platform Availability | 99.9% uptime |

---

## 4-Phase Implementation Plan

### Phase 1 — Foundation, Admin Control & Web-Centric Core
>
> Establish LangGraph scaffolding, user role security, Admin workspace, project isolation, and the core web ticket pipeline.

- Admin workspace initialization & user provisioning
- SLA & KB document ingestion into Qdrant
- Customer Portal with AI disclosure (UI-09)
- Live Ticket Queue (UI-01) & HIL Board (UI-06)
- **Agents Active:** AG-13, AG-01, AG-02, AG-16, AG-06, AG-08

### Phase 2 — Agentic SDLC Coordination & Complex Routing
>
> Implement ticket splitting, semantic deduplication, and engineering handshakes.

- Master Ticket linkage with Qdrant semantic search
- Complex ticket splitting into independent sub-graphs
- Routing to SRE/DevOps/QA agents; dual-confirmation gate
- **Agents Active:** AG-11, AG-03, AG-04, AG-09, AG-14

### Phase 3 — Analytics, Self-Healing KB & Compliance
>
> Enable RAG-based First-Contact Resolution, automated reporting, and 7-year data lifecycle enforcement.

- Qdrant RAG pipeline for KB resolution
- Auto-generated reports (R-01 to R-05)
- Dashboards: SLA Compliance (UI-02), Sentiment (UI-03), VoC (UI-08), Executive (UI-11)
- **Agents Active:** AG-15, AG-05, AG-10, AG-07, AG-12, AG-17

### Phase 4 — Omnichannel Integration & Interactive Avatar _(Future)_
>
> Break out of web-only, add Email/Slack/Teams/WhatsApp channels, and introduce a 3D voice-controlled avatar for internal users.

- Full omnichannel webhook integration
- 3D WebGL Voice Avatar with TTS + lip-sync (Three.js)
- Channel Admin UI (UI-04)
- **Agents Active:** AG-01 (Full Mode), AG-07 (Full Mode)

---

## Setup & Installation

### Prerequisites

- Python 3.11+
- Node.js 18+
- Docker (for Redis, PostgreSQL, Qdrant)
- A configured `.env` file in both `backend/` and `frontend/` (see `.env.example` in each directory)

### 1. Clone the Repository

```bash
git clone https://github.com/your-org/Customer-Support-Agent.git
cd Customer-Support-Agent
```

### 2. Configure Environment Variables

```bash
# Backend
cp backend/.env.example backend/.env
# Edit backend/.env with your API keys, DB URLs, Redis URL, Qdrant URL

# Frontend
cp frontend/.env.example frontend/.env
# Edit frontend/.env with your API base URL
```

### 3. Setup Backend (Python / FastAPI)

```bash
cd backend
python -m venv venv

# Linux / macOS
source venv/bin/activate

# Windows
.\venv\Scripts\activate

pip install -r requirements.txt
```

### 4. Setup Frontend (React / Vite)

```bash
cd ../frontend
npm install
```

### 5. Run the Application

**Backend (FastAPI + LangGraph):**

```bash
cd backend
source venv/bin/activate   # or .\venv\Scripts\activate on Windows
python run.py
```

**Frontend (Vite Dev Server):**

```bash
cd frontend
npm run dev
```

The backend API will be available at `http://localhost:8000` and the frontend at `http://localhost:5173`.

---

## Project Structure

```
Customer-Support-Agent/
├── backend/
│   ├── app/
│   │   ├── agents/          # 17 LangGraph agent node definitions
│   │   ├── api/             # FastAPI route handlers
│   │   ├── core/            # Config, RBAC, security middleware
│   │   ├── models/          # SQLAlchemy DB models
│   │   ├── services/        # Qdrant, Redis, ITSM integrations
│   │   └── graph/           # LangGraph StateGraph definition
│   ├── requirements.txt
│   ├── run.py
│   └── .env.example
├── frontend/
│   ├── src/
│   │   ├── components/      # Shared UI components
│   │   ├── pages/           # Role-based dashboard pages
│   │   ├── store/           # Redux state management
│   │   └── api/             # Axios API client
│   ├── package.json
│   └── .env.example
├── brd.md                   # Business Requirements Document
├── plan.md                  # Master Architecture & Implementation Blueprint
└── README.md
```

---

## Core Business Rules

| Rule ID | Name | Description |
|---|---|---|
| BR-021-001 | Omnichannel Intake | All channels must be processed within 5 minutes of receipt |
| BR-021-002 | Concurrent Processing | Unlimited concurrent tickets without performance degradation |
| BR-021-003 | SLA Governance & ITSM Integration | SLA clocks activated on ticket creation; all tickets synced to ITSM |
| BR-021-004 | HIL for Critical Actions | Billing, legal, VIP, and angry customer tickets require mandatory human review |
| BR-021-005 | AI Disclosure | Customers must acknowledge AI involvement before any support interaction begins |
| BR-021-006 | Data Privacy & PII Compliance | GDPR/CCPA-compliant; PII redacted; 7-year enterprise retention enforced |

---

## Non-Functional Requirements

| NFR | Category | Target |
|---|---|---|
| NFR-01 | Performance | Daily Digest delivered within 30 minutes of 08:00 schedule |
| NFR-02 | Scalability | Unlimited concurrent tickets with horizontal scaling |
| NFR-03 | Availability | 99.9% uptime SLA |
| NFR-04 | Reliability | Zero loss of customer messages during processing |
| NFR-05 | Security | TLS 1.3 encryption for all data in transit and at rest |
| NFR-06 | Security | RBAC with strict authentication on all interfaces |
| NFR-11 | Accuracy | ≥85% triage classification accuracy vs. human reviewer benchmarks |
| NFR-12 | Auditability | Every triage, SLA, and routing decision traceable to source ticket |
| NFR-13 | Data Integrity | Daily backups; recovery within 24 hours |
