# Catalyst Architecture Patterns

> **⚠️ PRE-FLIGHT CHECK:** Before building anything from these patterns, confirm `.catalystrc` and `catalyst.json` exist in the project directory. If not, STOP and tell the user to run `catalyst init` first. Do not scaffold the project yourself.

## Purpose

This file exists for one reason: **when a user describes a use case, map it to a complete Catalyst
infrastructure blueprint**. The user does not need to know what Catalyst services exist — read their
requirements, apply the decision matrix below, and produce a concrete architecture with named services,
how they connect, and what the folder structure looks like.

**Always use this file when:**
- A user says "I want to build X" or "help me architect Y"
- A user has a use case but no idea what infrastructure they need
- A user asks "what Catalyst services should I use for my project?"

---

## Step 1 — Requirement → Service Mapping

Read the user's requirements and map each capability to a Catalyst service using this table.

### Compute

| Requirement | Catalyst Service | Notes |
|-------------|-----------------|-------|
| REST API / HTTP endpoints | **Advanced I/O Function** | Stateless, per-request, node20 |
| Simple string-in / string-out function | **Basic I/O Function** | GET only, limited use |
| Background job / async processing | **Job Scheduling** | Replaces deprecated Cron |
| Scheduled recurring task | **Job Scheduling** | Configure interval in console |
| Persistent server / WebSockets / long-lived process | **AppSail** | Docker or managed runtime |
| Respond to Zoho product events (CRM, Desk, etc.) | **Event Function** via **Signals** | Event-driven |
| Multi-step business workflow with approvals | **Circuits** | Visual drag-and-drop |
| Web scraping / PDF generation / screenshots | **Browser Logic Function** + **SmartBrowz** | Managed headless browser |

### Storage

| Requirement | Catalyst Service | Notes |
|-------------|-----------------|-------|
| Structured/relational data with queries | **Data Store** + **ZCQL** | Max 300 rows/query, paginate |
| Flexible schema / document data | **NoSQL** | Collections + documents |
| File/object storage (images, docs, exports) | **Stratus** | S3-compatible, preferred |
| Session state / temporary data (< 48hr) | **Cache** | Strings only, serialize JSON |
| Full-text search across records | **Search** | Enable per-column in console |

### Frontend

| Requirement | Catalyst Service | Notes |
|-------------|-----------------|-------|
| React / Vue / Next.js / static site | **Slate** | Git-based, SSR support, preferred |
| Simple HTML/JS (no build step) | **Web Client Hosting** | Legacy, `client/` directory |

### Auth & Users

| Requirement | Catalyst Service | Notes |
|-------------|-----------------|-------|
| User signup, login, session management | **Auth & User Management** | Built-in, Zoho IAM |
| OAuth to external services (Google, GitHub, etc.) | **Connections** | Token auto-refresh |
| SSO / enterprise login | **Auth (Custom SSO)** | Configure in console |

### Integration

| Requirement | Catalyst Service | Notes |
|-------------|-----------------|-------|
| React to Zoho CRM / Desk / Books events | **Signals** | Subscribe to Zoho product signals |
| Call external APIs with OAuth | **Connections** | Pre-built Zoho connectors + custom |
| Webhook from external services | **Advanced I/O Function** | Expose as HTTP endpoint |

### AI / ML

| Requirement | Catalyst Service | Notes |
|-------------|-----------------|-------|
| OCR, face detection, image moderation | **Zia Services** | Vision microservices |
| Text classification, sentiment, NLP | **Zia Services** | Text analytics microservices |
| Custom ML model (no-code) | **QuickML** | AutoML pipeline builder |
| Chatbot / conversational UI | **ConvoKraft** | Embeddable AI bot |

### DevOps

| Requirement | Catalyst Service | Notes |
|-------------|-----------------|-------|
| CI/CD pipeline (GitHub/GitLab) | **Pipelines** | YAML-based |
| App monitoring, alerting, logs | **DevOps (Logs + APM + Alerts)** | Built into console |

---

## Step 2 — Architecture Blueprints

Pre-composed architectures for common use cases. Use these as the starting point and adapt for the
user's specific requirements.

---

### Blueprint 1: Standard Web App (SaaS / CRUD App)

**Use case examples:** Task manager, CRM, dashboard, booking system, internal tool, notes app.

**Requirements typically include:** User auth, relational data, REST API, web frontend.

```
┌─────────────────────────────────────────────────────┐
│                   User's Browser                    │
└────────────────────┬────────────────────────────────┘
                     │ HTTPS
┌────────────────────▼────────────────────────────────┐
│              Slate (Frontend)                       │
│         React / Vue / Next.js app                   │
│   fetch('/server/api/execute', { credentials:       │
│   'include', ... }) for all API calls               │
└────────────────────┬────────────────────────────────┘
                     │ /server/<fn>/execute
┌────────────────────▼────────────────────────────────┐
│         Advanced I/O Function (REST API)            │
│   catalyst.initialize(req) → user-scoped auth       │
│   catalyst.initialize(req, {type: admin}) → DS ops  │
└──────────┬─────────────────┬───────────────────────┘
           │                 │
┌──────────▼──────┐  ┌───────▼────────┐
│   Data Store    │  │    Stratus     │
│  (user records, │  │ (file uploads, │
│   app data)     │  │  media, docs)  │
└─────────────────┘  └────────────────┘

Auth: Catalyst Auth & User Management
      (signup → email verification → login → session cookie)
```

**Services:** Slate · Advanced I/O Function · Data Store · Stratus · Auth

**Key implementation notes:**
- Always use `credentials: 'include'` in `fetch()` calls from Slate to the function
- Enable CRUD permissions for App User role on each Data Store table
- Use admin-scoped SDK for all DataStore operations in the function
- Use user-scoped SDK only to call `getCurrentUser()` for auth verification

**Project structure:**
```
project-root/
├── catalyst.json
├── client/                       # or use Slate (separate Git repo)
│   └── build/
├── functions/
│   └── api/
│       ├── index.js              # Advanced I/O handler
│       ├── package.json
│       └── catalyst-config.json
```

---

### Blueprint 2: Mobile App Backend (API-only)

**Use case examples:** iOS/Android app backend, Flutter app, React Native app.

**Requirements typically include:** REST API, user auth, data storage, push notifications.

```
┌─────────────────────────────────────────────────────┐
│              Mobile App (iOS/Android/Flutter)       │
│       Uses Catalyst Mobile SDK or REST API          │
└────────────────────┬────────────────────────────────┘
                     │ HTTPS + Catalyst Auth SDK
┌────────────────────▼────────────────────────────────┐
│         Advanced I/O Function (REST API)            │
│              + API Gateway (optional)               │
│      Rate limiting, path routing, auth rules        │
└──────────┬─────────────────┬───────────────────────┘
           │                 │
┌──────────▼──────┐  ┌───────▼────────┐  ┌──────────────────┐
│   Data Store    │  │    Stratus     │  │  Push Notification│
│  (user data,    │  │ (media, files) │  │  (APNs + FCM)     │
│   app records)  │  └────────────────┘  └──────────────────┘
└─────────────────┘

Optional background processing:
┌──────────────────────────────────────┐
│  Job Scheduling (async tasks,        │
│  notifications, cleanup, exports)    │
└──────────────────────────────────────┘
```

**Services:** Advanced I/O Function · API Gateway · Data Store · Stratus · Push Notifications · Job Scheduling · Auth

---

### Blueprint 3: Event-Driven / Automation App

**Use case examples:** Zoho CRM → send onboarding email when lead converts, sync data between Zoho products, trigger workflows on Zoho Desk ticket creation.

**Requirements typically include:** React to Zoho product events, run business logic, update data.

```
┌─────────────────────────────────────────────────────┐
│            Zoho Product (CRM, Desk, Books…)         │
└────────────────────┬────────────────────────────────┘
                     │ Zoho product event
┌────────────────────▼────────────────────────────────┐
│                  Signals (Event Bus)                │
│         Subscribe to CatalystbyZoho product events  │
└────────────────────┬────────────────────────────────┘
                     │ triggers
┌────────────────────▼────────────────────────────────┐
│               Event Function                        │
│   Business logic: transform data, call APIs,        │
│   send emails, update records                       │
└──────────┬───────────┬─────────────────────────────┘
           │           │
┌──────────▼──────┐ ┌──▼──────────────┐
│   Data Store    │ │   Connections   │
│  (store results)│ │ (OAuth to       │
└─────────────────┘ │  external APIs) │
                    └─────────────────┘

For complex multi-step logic:
┌──────────────────────────────────────┐
│  Circuits (multi-step workflow with  │
│  approvals, branching, retries)      │
└──────────────────────────────────────┘
```

**Services:** Signals · Event Function · Data Store · Connections · Circuits (optional)

---

### Blueprint 4: Scheduled Data Pipeline

**Use case examples:** Nightly report generation, daily data sync from external API, periodic cache refresh, scheduled email digests.

**Requirements typically include:** Run on schedule, fetch/process data, store results, send notifications.

```
┌────────────────────────────────────┐
│   Job Scheduling (trigger)         │
│   Configure: interval, target fn   │
└──────────────┬─────────────────────┘
               │ triggers
┌──────────────▼─────────────────────┐
│   Job Function (worker)            │
│   Fetch external data (via HTTP)   │
│   Transform / aggregate            │
│   Store results                    │
└──────┬──────────────┬──────────────┘
       │              │
┌──────▼──────┐  ┌────▼──────────┐
│  Data Store │  │    Cache      │
│ (persist    │  │ (hot results, │
│  results)   │  │  temp state)  │
└─────────────┘  └───────────────┘
                              │
                    ┌─────────▼──────────┐
                    │  Mail (send digest │
                    │  / alert email)    │
                    └────────────────────┘
```

**Services:** Job Scheduling · Job Function · Data Store · Cache · Mail · Connections (if fetching from OAuth-protected APIs)

---

### Blueprint 5: AI-Powered App

**Use case examples:** Document OCR, image classification, customer sentiment analysis, AI chatbot, smart search.

**Sub-pattern A — Vision / NLP pipeline:**
```
┌──────────────┐    ┌───────────────────────┐    ┌─────────────┐
│   Stratus    │───▶│  Advanced I/O or Job  │───▶│ Zia Services│
│ (input files)│    │  Function (orchestrate│    │ (OCR, vision│
└──────────────┘    │  processing)          │    │  NLP, etc.) │
                    └──────────────┬────────┘    └─────────────┘
                                   │
                          ┌────────▼──────────┐
                          │     Data Store    │
                          │ (store AI results)│
                          └───────────────────┘
```

**Sub-pattern B — Conversational chatbot:**
```
┌─────────────────────────────────────────────────────┐
│          Website / Web App (Slate)                  │
│     Embed ConvoKraft bot snippet                    │
└────────────────────┬────────────────────────────────┘
                     │
┌────────────────────▼────────────────────────────────┐
│                 ConvoKraft                          │
│   Configure intents, responses, LLM integration    │
└────────────────────┬────────────────────────────────┘
                     │ calls backend when needed
┌────────────────────▼────────────────────────────────┐
│         Advanced I/O Function (API)                 │
└────────────────────┬────────────────────────────────┘
                     │
                ┌────▼─────────┐
                │  Data Store  │
                │(conversation │
                │ history, etc)│
                └──────────────┘
```

**Services (Vision/NLP):** Stratus · Job/Advanced I/O Function · Zia Services · Data Store

**Services (Chatbot):** Slate · ConvoKraft · Advanced I/O Function · Data Store

---

### Blueprint 6: Persistent Server App (Long-running / WebSockets)

**Use case examples:** Real-time collaboration, WebSocket server, game server backend, long-running data processor, containerized microservice.

```
┌─────────────────────────────────────────────────────┐
│              Client (Browser / Mobile)              │
└────────────────────┬────────────────────────────────┘
                     │ HTTP / WebSocket
┌────────────────────▼────────────────────────────────┐
│                   AppSail                           │
│   Node.js (Express) / Java (Spring Boot) /          │
│   Python (FastAPI) / Custom Docker                  │
│   Auto-scales 1–5 instances                         │
│   Port: process.env.X_ZOHO_CATALYST_LISTEN_PORT     │
└──────────┬────────────────┬────────────────────────┘
           │                │
┌──────────▼──────┐  ┌───────▼──────┐  ┌──────────────┐
│   Data Store    │  │    Cache     │  │   Stratus    │
│  (persistent    │  │(session/room │  │(media, files)│
│   data)         │  │  state)      │  └──────────────┘
└─────────────────┘  └──────────────┘
```

**Services:** AppSail · Data Store · Cache · Stratus

**Key note:** Use admin-scoped SDK in AppSail: `catalyst.initialize(req, { scope: 'admin' })`.

---

### Blueprint 7: Static / Content Site

**Use case examples:** Marketing site, documentation site, landing page, blog.

```
┌─────────────────────────────────────────────────────┐
│              Slate (Frontend)                       │
│   Static site / SSR (Next.js, Astro, etc.)          │
│   Git-based deploy — push to branch → auto deploy   │
└────────────────────┬────────────────────────────────┘
                     │ optional dynamic data
┌────────────────────▼────────────────────────────────┐
│         Advanced I/O Function (optional)            │
│   Contact form, newsletter signup, search           │
└────────────────────┬────────────────────────────────┘
                     │
                ┌────▼─────────┐
                │  Data Store  │
                │(form submits,│
                │ subscribers) │
                └──────────────┘
```

**Services:** Slate · Advanced I/O Function (optional) · Data Store (optional)

---

### Blueprint 8: Web Scraping / Automation Pipeline

**Use case examples:** Price monitoring, data extraction from websites, automated PDF report generation, screenshot service.

```
┌────────────────────────────────────┐
│   Job Scheduling (trigger)         │
│   or Advanced I/O (on-demand)      │
└──────────────┬─────────────────────┘
               │
┌──────────────▼─────────────────────┐
│   Browser Logic Function           │
│   + SmartBrowz (headless browser)  │
│   Puppeteer-like API               │
│   Navigate → extract → screenshot  │
└──────────────┬─────────────────────┘
               │ results
┌──────────────▼─────────────────────┐
│   Stratus (store PDFs/screenshots) │
│   Data Store (store extracted data)│
└────────────────────────────────────┘
```

**Services:** Job Scheduling · Browser Logic Function · SmartBrowz · Stratus · Data Store

---

## Step 3 — How to Propose an Architecture (Workflow for the Model)

When a user describes their use case, follow this process:

1. **Identify the primary app pattern** — match to one of the 8 blueprints above (or combine multiple).

2. **Map individual requirements** — use the decision matrix in Step 1 for any requirements not covered by the blueprint.

3. **Produce the architecture proposal** in this format:

   ```
   ## Proposed Architecture for [User's App]

   ### Services you'll need
   | Service | Purpose |
   |---------|---------|
   | Slate   | Frontend hosting for your React app |
   | Advanced I/O Function | REST API for CRUD operations |
   | Data Store | Store [user's entities, e.g. medications, tasks, orders] |
   | Auth | User signup and login |
   | Stratus | File uploads (profile photos, documents) |

   ### How they connect
   [Describe the data flow — browser → frontend → function → DB → response]

   ### Project structure
   [Show the folder layout]

   ### First steps
   1. `catalyst init` — initialize the project
   2. Create Data Store tables: [list table names + columns]
   3. Set up Auth in the console (Authentication section)
   4. Build the Advanced I/O function at `functions/api/`
   5. Initialize Slate for the frontend
   ```

4. **Check for Zoho MCP** — if `CatalystbyZoho_*` tools are available, offer to create the Data Store tables and Stratus buckets directly from the conversation.

5. **Flag common gotchas** relevant to the architecture:
   - DataStore → remind to enable CRUD permissions for App User role
   - Web client → function calls → remind about `credentials: 'include'`
   - AppSail → remind about the correct port env variable
   - Job Scheduling → note that CLI always deploys to Development
