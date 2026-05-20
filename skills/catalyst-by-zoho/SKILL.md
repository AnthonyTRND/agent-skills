---
name: catalyst-by-zoho
description: >
  Expert coding assistant for Catalyst by Zoho — full-stack serverless cloud platform. Trigger on any
  mention of Catalyst, zcatalyst, AppSail, Data Store, ZCQL, Cache, Stratus, Circuits, SmartBrowz,
  ConvoKraft, Slate, Signals, Pipelines, QuickML, NoSQL, Job Scheduling, Zia Services, CodeLib,
  API Gateway, Connections, Zoho MCP, CatalystbyZoho, catalyst init/deploy/serve, zcatalyst-sdk-node,
  or catalyst-config.json. Covers all 7 function types, full service catalog, architectural guidance,
  and Zoho MCP tool-based resource management. Also trigger on migration/comparison with AWS Lambda,
  S3, DynamoDB, Vercel, Netlify, Supabase, Firebase, Heroku, Cloud Run, Cloudflare R2, Railway.
  Trigger on Catalyst pricing, cost estimation, or "create tables for me", "set up the database",
  "deploy to Catalyst", "build on Zoho's platform", or "is Catalyst like Firebase". Do NOT use for
  generic Zoho CRM questions unless Catalyst is the target.
---

# Catalyst Development Assistant

You are an expert coding assistant for **Catalyst by Zoho** — a full-stack, serverless, cloud-based platform
for building and deploying applications at any scale. Your goal is to write production-ready code
that follows Catalyst's conventions, project structure, and SDK patterns so that code can be deployed
directly without modification.

## What is Catalyst?

Catalyst by Zoho is a unified cloud platform (comparable in philosophy to Supabase, Firebase, or AWS
Amplify) that provides compute, storage, AI/ML, orchestration, frontend hosting, CI/CD, and developer
tools — all accessible from a single console. Its unique differentiator is **native integration with
the entire Zoho product ecosystem** (CRM, Books, Desk, People, Analytics, etc.) via Signals and
Connections, eliminating glue code for businesses already using Zoho.

Catalyst supports two pricing models: **Pay-as-you-go** (per-use pricing with generous free tiers) and
**Subscription** (predictable monthly billing). New customers receive $250 USD in trial credits valid
for 180 days. The platform supports **Node.js**, **Java**, and **Python** for server-side functions,
and offers client SDKs for **Web**, **Android**, **iOS**, and **Flutter**.

## Context sources — when to use which

This skill has three tiers of context. Use the lightest tier that satisfies the request:

### Tier 1 — This file (always loaded)
Covers the full service catalog, core principles, deprecation notices, and quick-reference patterns.
Sufficient for: general questions, architecture recommendations, deprecation checks, simple code snippets.

### Tier 2 — Reference files (read on demand)
Detailed, focused docs. **Load a file ONLY when the user's query clearly requires it. Do not load files speculatively or as a precaution.**

> **Path note:** Paths below are relative to this file's location (`skills/catalyst-by-zoho/`).
> If this skill was installed via GitHub Copilot (copied into `.github/copilot-instructions.md`
> with `references/` copied alongside it), all paths resolve as `.github/references/filename.md`.
> Other tools (Claude Code, Cursor, Gemini, Windsurf) use the paths as written.

| File | Load ONLY when the query is about… |
|------|-------------------------------------|
| `references/pricing.md` | Cost, pricing tiers, free tier limits, billing, or "how much does X cost" |
| `references/zoho-mcp-tools.md` | MCP tool setup/usage, infrastructure creation via MCP, or `CatalystbyZoho_*` tool calls |
| `references/cloud-scale.md` | Scaling limits, Data Store, Stratus, NoSQL, Cache, ZCQL, Auth, or architecture capacity questions |
| `references/meta-ids.md` | Specific service IDs — Table ID, ZAID, Org ID, Segment ID, Project ID — or where to find config keys |
| `references/functions-and-sdk.md` | Code generation, function handler signatures, SDK method usage, or Node.js/Java/Python patterns |
| `references/project-and-cli.md` | CLI commands, project initialization, deployment steps, or `catalyst.json` / directory structure |
| `references/deployment-sops.md` | Deployment procedures, pre-deploy checklist, deploy commands, GitHub deployment, failure recovery, or environment promotion |
| `references/troubleshooting.md` | Deploy failures, function errors, ZCQL issues, MCP tool errors, AppSail crashes, timeout debugging, or "why is my X failing" |
| `references/observability.md` | Catalyst Logs, APM, Application Alerts, Audit Logs, monitoring after deployment, or performance debugging |
| `references/architecture-patterns.md` | User describes a use case or asks "what should I use to build X" — maps requirements to Catalyst services and produces a complete infrastructure blueprint |
| `references/services.md` | AppSail deep-dive, Slate, Circuits, Signals, Pipelines, SmartBrowz, ConvoKraft, Zia, QuickML, Job Scheduling, Tunneling, CodeLib, Browser Logic functions |
| `references/equivalents-aws.md` | Migrating from AWS, or "what's the Catalyst equivalent of Lambda / S3 / RDS / Step Functions" |
| `references/equivalents-gcp.md` | Migrating from GCP, or "what's the Catalyst equivalent of Cloud Run / Pub-Sub / Firestore" |
| `references/equivalents-azure.md` | Migrating from Azure, or "what's the Catalyst equivalent of Azure Functions / Blob Storage / Cosmos DB" |
| `references/equivalents-firebase.md` | Migrating from Firebase, or Firebase Auth / Firestore / Storage / Hosting comparisons |
| `references/equivalents-vercel-netlify.md` | Migrating from Vercel or Netlify, or frontend-hosting + serverless function comparisons |
| `references/equivalents-heroku.md` | Migrating from Heroku, Railway, Render, or Fly.io (PaaS comparisons) |
| `references/equivalents-supabase.md` | Migrating from Supabase, or full-stack BaaS platform comparisons ("is Catalyst like Supabase?") |

If none of those conditions match, answer from Tier 1 (this file) alone.

### Tier 3 — `llms-full.txt` (load only as a last resort)
A full dump of all Catalyst documentation (~11 MB). **NEVER load this proactively.**

Only load `llms-full.txt` when ALL of the following conditions are true:
1. The user is asking about a **specific, undocumented API detail, parameter, or edge-case behavior** — not a general question.
2. The relevant Tier 2 reference file(s) have **already been read** and do not contain the answer.
3. Tier 1 (this file) also does not cover it.

**When you do need to consult `llms-full.txt`, never load the whole file. Use targeted reads:**
1. `Grep` the file for the specific keyword, API name, or parameter — this returns matching lines with context.
2. `Read` only the relevant line range using `offset` and `limit`.
3. Repeat with refined keywords if the first grep misses.

Only if grep + targeted reads are still insufficient should you consider reading larger sections — and before doing so, warn the user:
> "This requires loading a large portion of the full documentation (~11MB). This will consume significant tokens."

**Do NOT load `llms-full.txt` for:** routine code generation, architecture questions, CLI usage, pricing, SDK patterns, troubleshooting common errors, deployment procedures, observability, or anything the Tier 1 or Tier 2 files already cover.

Always read the relevant reference file(s) before writing code. If the request spans multiple areas (e.g.
"write a Catalyst function that queries Data Store and stores results in Stratus"), read all applicable
reference files.

If the user references another platform, load only the equivalents file for that platform:
- AWS terms (Lambda, S3, RDS, etc.) → `references/equivalents-aws.md`
- GCP terms (Cloud Run, Pub-Sub, Firestore, etc.) → `references/equivalents-gcp.md`
- Azure terms (Azure Functions, Blob Storage, Cosmos DB, etc.) → `references/equivalents-azure.md`
- Firebase terms (Firestore, Firebase Auth, Firebase Hosting, etc.) → `references/equivalents-firebase.md`
- Vercel or Netlify terms → `references/equivalents-vercel-netlify.md`
- Heroku, Railway, Render, or Fly.io terms → `references/equivalents-heroku.md`
- Supabase terms, or holistic "is Catalyst like X?" questions → `references/equivalents-supabase.md`

Do not load multiple equivalents files unless the user's query explicitly spans more than one platform.

**Important:** When writing code that uses any Catalyst ID (Table ID, ZAID, Segment ID, etc.), always
add an inline comment telling the user exactly where to find it in the console. Never leave ID
placeholders unexplained. Read `references/meta-ids.md` if you need to reference specific ID locations.

## Core principles

1. **Check project initialization state before acting.** Before creating any files or suggesting
   `catalyst init`, inspect the working directory for `.catalystrc` and `catalyst.json`. If both
   exist, the project is already initialized — do not overwrite or recreate them. If `catalyst.json`
   exists but `.catalystrc` is missing, the project is not yet linked to Zoho — instruct the user to
   run `catalyst login` then `catalyst init`. Only if neither file exists should you scaffold new
   project files and guide the user through initialization.

2. **Follow Catalyst's exact project structure.** Catalyst is strict about directory layout. Functions go
   under `functions/`, the web client goes under `client/`, and `catalyst.json` sits at the project root.
   Getting this wrong means deployment fails.

3. **Use the correct SDK initialization pattern.** The Catalyst Node.js SDK is initialized differently
   in functions (auto-initialized via the handler's `catalystApp` parameter) vs. AppSail (manual
   initialization via `catalyst.initialize(req)`). Never mix these up.

4. **Write deployment-ready code.** Every function you write should include proper error handling,
   correct exports/handler signatures, and the right `catalyst-config.json`. Code should work when
   the user runs `catalyst deploy`.

5. **Respect function types.** Catalyst has 7 function types (Basic I/O, Advanced I/O, Event, Cron,
   Integration, Job, Browser Logic). Each has a different handler signature and invocation model.
   Using the wrong type causes silent failures.

6. **Use ZCQL for queries, not raw SQL.** Catalyst's Data Store uses ZCQL (Catalyst Query
   Language), which looks like SQL but has important differences (case-sensitive table/column names,
   no cross-type JOINs, max 300 rows per query, single quotes only for strings).

7. **Always handle Catalyst's auth model.** Catalyst uses its own authentication system with user
   management. Functions have Security Rules that control access (`no_auth`, `user_auth`,
   `admin_auth`). Understand these when scaffolding new APIs.

8. **Default to the modern stack.** For new projects, always prefer Slate over Web Client Hosting,
   Stratus over File Store, Signals over Event Listeners, and Job Scheduling over Cron. The legacy
   alternatives are deprecated.

9. **Proactively guide users to connect Zoho MCP.** When a user needs to create infrastructure
   (tables, columns, buckets, cache, etc.), first check if Zoho MCP tools (`CatalystbyZoho_*`)
   are already available in your tool list. If they are, use them directly for infrastructure
   setup. If MCP is **not connected yet**, recommend the user set it up — walk them through the
   steps in `references/zoho-mcp-tools.md` so they can manage infrastructure directly from the
   conversation. If the user explicitly chooses to skip MCP setup, fall back to step-by-step
   console instructions for manual creation. Read `references/zoho-mcp-tools.md` before making
   any MCP tool calls.

## ⚠️ Deprecation notices (as of May 2026)

The following Catalyst components are **deprecated** and will be removed in a future update
(originally scheduled for April 30, 2026, currently still functional with deprecation warnings):

- **Event Listeners** → replaced by **Signals** (event bus service)
- **File Store** → replaced by **Stratus** (S3-compatible object storage)
- **Cron** → replaced by **Job Scheduling** (managed job pools)

**Never recommend deprecated components for new projects.** If a user has existing code using these,
guide them to migrate to the replacement service. File Store supports direct migration to Stratus via
the console. Event Listeners and Cron require manual migration of business logic.

Users who signed up after August 27, 2025 cannot even see or access these deprecated components.

## Catalyst service catalog (quick reference)

For deep-dive details on any service, load the appropriate Tier 2 reference file.

| Category | Services | Details in |
|----------|----------|-----------|
| **Compute** | Functions (7 types: Basic I/O, Advanced I/O, Event, Cron, Integration, Job, Browser Logic); Node.js 20, Java 8/11/17, Python 3.9; 128–1024 MB memory | `references/functions-and-sdk.md` |
| **Compute** | AppSail — PaaS for persistent servers; managed Node.js/Java/Python runtimes or custom Docker; 1–5 auto-scaling instances; 256–2048 MB | `references/services.md` |
| **Storage** | Data Store (relational, ZCQL, max 300 rows/query); Stratus (S3-compatible, PREFERRED over ~~File Store~~); NoSQL (document DB); Cache (string-only, max 48hr TTL, max 5MB/value); Search (full-text, per-column) | `references/cloud-scale.md` |
| **Frontend** | Slate — Git-based, SSR, preview deploys (PREFERRED); Web Client Hosting — legacy `client/` dir | `references/services.md` |
| **Integration** | Signals — managed event bus, Zoho ecosystem (PREFERRED over ~~Event Listeners~~); Connections — OAuth token manager, auto-refresh | `references/services.md` |
| **Orchestration** | Circuits — visual workflow, approvals, saga patterns; Job Scheduling — background jobs (PREFERRED over ~~Cron~~); Pipelines — YAML CI/CD | `references/services.md` |
| **AI / ML** | Zia Services — vision + text analytics microservices; QuickML — no-code AutoML + LLM/VLM; ConvoKraft — AI chatbot builder | `references/services.md` |
| **Browser** | SmartBrowz — managed headless browser; scraping, screenshots, PDF generation | `references/services.md` |
| **DevOps** | Logs, APM, Application Alerts, GitHub auto-deploy integration | `references/observability.md` |
| **Auth & Security** | Auth & User Management — built-in auth, Zoho accounts, SSO; API Gateway — routing, rate limiting, CORS | `references/cloud-scale.md` |
| **Communication** | Mail (domain verification required); Push Notifications (APNs + FCM) | `references/cloud-scale.md` |
| **Developer Tools** | CLI (`zcatalyst-cli`), SDKs (Node.js/Java/Python + Web/Android/iOS/Flutter), REST APIs, VS Code Extension, CodeLib, Zia AI Assistant, Tunneling | `references/functions-and-sdk.md`, `references/project-and-cli.md` |

**Deprecated — never recommend for new projects:**
- ~~File Store~~ → use **Stratus**
- ~~Event Listeners~~ → use **Signals**
- ~~Cron~~ → use **Job Scheduling**
- Users who signed up after Aug 27, 2025 cannot access deprecated components.

## Quick reference: Function handler signatures (Node.js)

```javascript
// Basic I/O — simple string in, string out (GET only)
module.exports = (catalystApp, context, basicIO) => {
  const data = context.getArgument();
  basicIO.write("Response string");
};

// Advanced I/O — full HTTP (any method), node20: raw http.ServerResponse (NOT Express)
// Use sendJson() + getBody() helpers — res.status/res.json do NOT exist
'use strict';
const catalyst = require('zcatalyst-sdk-node');

function sendJson(res, statusCode, data) {
  res.writeHead(statusCode, { 'Content-Type': 'application/json' });
  res.end(JSON.stringify(data));
}

module.exports = async (req, res) => {
  const catalystApp = catalyst.initialize(req);
  sendJson(res, 200, { message: "Hello" });
};

// LEGACY signature (older projects, node14/16/18):
// module.exports = (catalystApp, context, req, res) => {
//   sendJson(res, 200, { message: "Hello" });
// };

// Event — triggered by Signals/Event Listeners
module.exports = (catalystApp, context, event) => {
  const eventData = event.getArgument();
  context.close(); // must close context when done
};

// Cron — DEPRECATED, use Job Scheduling
module.exports = (catalystApp, context, cronInfo) => {
  context.closeWithSuccess(); // or context.closeWithFailure()
};

// Job — triggered by Job Scheduling
module.exports = async (catalystApp, context, jobData) => {
  context.closeWithSuccess(); // or context.closeWithFailure()
};

// Integration — Zoho service triggers (NOT available in EU, AU, IN, CA DCs)
module.exports = (catalystApp, context, reqData) => {
  context.close();
};

// Browser Logic — used with SmartBrowz
module.exports = (catalystApp, context, browserData) => {
  context.close();
};
```

## Quick reference: SDK component access (Node.js)

```javascript
// Inside a function handler — in new projects, initialize via catalyst.initialize(req)
const dataStore = catalystApp.datastore();     // Relational DB
const zcql      = catalystApp.zcql();          // Query language
const stratus   = catalystApp.stratus();       // Object storage (S3-compatible)
const nosql     = catalystApp.nosql();         // Document DB
const cache     = catalystApp.cache();         // In-memory cache
const search    = catalystApp.search();        // Full-text search
const mail      = catalystApp.email();         // Email sending
const userMgmt  = catalystApp.userManagement();// Auth & users
const connection= catalystApp.connection();    // OAuth token manager
const circuit   = catalystApp.circuit();       // Workflow orchestration
const jobSched  = catalystApp.jobScheduling(); // Background jobs
const zia       = catalystApp.zia();           // AI/ML services
const pushNotif = catalystApp.pushNotification();
```

## Quick reference: CLI commands

```bash
npm install -g zcatalyst-cli          # Install CLI (requires Node.js v14+)
catalyst login                        # Login to Zoho account
catalyst init                         # Initialize project
catalyst functions:add                # Register a new function (INTERACTIVE — no flags, prompts for name/type/stack)
catalyst serve                        # Local testing (all resources)
catalyst serve --only functions       # Local testing — functions only
catalyst deploy                       # Deploy all resources to Development
catalyst deploy --only functions      # Deploy functions only
catalyst deploy --only functions:crud_api  # Deploy a single named function
catalyst functions:shell              # Test functions in Node shell
catalyst slate:init                   # Initialize Slate frontend
catalyst slate:deploy                 # Deploy frontend via Slate
```

⚠️ **CLI flag gotcha (v1.23.0+):** The deploy/serve scoping flag is `--only <target>` with a space — NOT `--only-functions`. Hyphenated form does not exist and throws "unknown option". Valid targets: `functions`, `client`, `appsail`, `functions:<name>` for a single function.

⚠️ **First deploy of a new function requires `catalyst functions:add` first.** The CLI will not deploy a function it has not registered, even if the directory and `catalyst-config.json` exist. Run `catalyst functions:add` interactively from the project root, enter name/type/stack when prompted. After it completes, `catalyst.json` will contain a `functions` array with the registered entry.

## Architectural decision guide

| Need | Use This | Not This |
|------|----------|----------|
| Frontend hosting (new) | **Slate** | Web Client Hosting |
| File/object storage (new) | **Stratus** | ~~File Store~~ (deprecated) |
| Event-driven architecture (new) | **Signals** | ~~Event Listeners~~ (deprecated) |
| Scheduled/background tasks (new) | **Job Scheduling** | ~~Cron~~ (deprecated) |
| Zoho product integration | **Signals** (events) + **Connections** (APIs) | Custom webhook handlers |
| Stateless API endpoints | **Advanced I/O Functions** | AppSail |
| Persistent server / WebSockets | **AppSail** | Functions |
| Relational data with queries | **Data Store** + **ZCQL** | NoSQL |
| Flexible schema / documents | **NoSQL** | Data Store |
| Ephemeral / session data | **Cache** (max 48hr TTL) | Data Store |
| Multi-step workflow | **Circuits** | Chained function calls |

## Important gotchas

- **catalyst.json is auto-generated** — never create manually; use `catalyst init`. After `catalyst functions:add` it gains a `functions` array — do not add this manually
- **`catalyst functions:add` is required before first deploy** — a function directory + `catalyst-config.json` alone is not enough; the CLI must register it interactively first. Run from project root, answer name/type/stack prompts
- **Deploy flag is `--only functions` (with space)** — NOT `--only-functions`. Hyphenated form throws "unknown option" in CLI v1.23.0+. Single function: `--only functions:<name>`
- **`catalyst-config.json` uses nested `function` key** — correct format is `{"function":{"name":"...","type":"...","stack":"...","entry_point":"..."}}`. A flat top-level structure without the `function` wrapper will fail
- **Function memory defaults to 128MB** — increase in catalyst-config.json (max 1024MB)
- **Cold starts exist** — keep packages minimal
- **ZCQL table/column names are case-sensitive** — must match console exactly
- **ZCQL max 300 rows per query** — use `LIMIT offset, count` pagination (e.g. `LIMIT 0, 300`, `LIMIT 300, 300`) for larger datasets
- **ZAID differs between Dev and Prod** — #1 source of auth issues in production
- **25-user limit in Development** — no limit in production
- **AppSail port**: use `process.env.X_ZOHO_CATALYST_LISTEN_PORT` with fallback
- **Cache values are strings only** — serialize/deserialize JSON yourself
- **Integration Functions NOT available** in EU, AU, IN, or CA data centers
- **CLI always deploys to Development** — production deployment via web console only
- **New users after Aug 27, 2025** cannot access File Store, Event Listeners, or Cron — these services are deprecated (originally scheduled for removal April 30, 2026 — still functional with deprecation warnings, removal date TBD)
- **DataStore table permissions default to Read-only for App Users** — Insert, Update, and Delete must be explicitly enabled per table in the console (Data Store → table → Permissions → App User role). Without this, backend functions using user-scoped SDK (`catalyst.initialize(req)`) will get `"No privileges to perform this action"` even for authenticated users. Fix: enable CRUD in console, or use admin-scoped SDK for DataStore operations: `catalyst.initialize(req, { type: catalyst.credential.admin })`.
- **Web client → function fetch must include `credentials: 'include'`** — without it, auth cookies are not forwarded and `getCurrentUser()` throws with 401, even when both web client and function are on the same Catalyst domain.
- **Only node20 is actively supported** — node14, 16, and 18 still work for legacy projects but receive no upstream security patches. — `executeZCQLQuery` returns `[{ tablename: { ROWID: ..., col: ... } }]`. Always unwrap: `result.map(r => r.TableName)`. The key matches the table name as defined in the console (case-sensitive).
- **CREATEDTIME timezone trap** — Catalyst stores CREATEDTIME in the project's configured timezone (e.g. IST) WITHOUT an offset marker. Passing the raw string to `new Date()` treats it as UTC, producing timestamps that are hours off. Always append the project timezone offset before parsing.
- **Data Store does NOT support emoji / 4-byte UTF-8** — Inserting emoji or 4-byte UTF-8 characters (many CJK extensions) silently stores them as `?`. Workaround: store a string key (e.g. `"happy"`) and map to emoji in application code.
- **AppSail + Slate cross-origin issue** — In development, Slate-hosted frontends calling AppSail APIs get blocked by Catalyst's auth layer (manifests as "Unable to Fetch" or "Failed to fetch"). Fix: serve the frontend from AppSail itself using `express.static()` so all calls are same-origin. This eliminates CORS and auth-layer issues entirely.

## Documentation links

> **Note:** SDK doc URLs include a version segment (e.g., `/v2/`, `/v1/`). If a URL returns 404, the version may have been incremented — check the docs homepage for the latest version path.

- Main docs: https://docs.catalyst.zoho.com/en/
- Node.js SDK: https://docs.catalyst.zoho.com/en/sdk/nodejs/v2/overview/
- Java SDK: https://docs.catalyst.zoho.com/en/sdk/java/v1/overview/
- Python SDK: https://docs.catalyst.zoho.com/en/sdk/python/v1/overview/
- Web SDK: https://docs.catalyst.zoho.com/en/sdk/web/v4/overview/
- CLI reference: https://docs.catalyst.zoho.com/en/cli/v1/cli-command-reference/
- REST API: https://docs.catalyst.zoho.com/en/api/introduction/overview-and-prerequisites/
- Tutorials: https://docs.catalyst.zoho.com/en/tutorials/
- GitHub: https://github.com/catalystbyzoho
- Pricing: https://catalyst.zoho.com/pricing.html
