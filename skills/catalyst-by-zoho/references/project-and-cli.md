# Project Structure & CLI Reference

## Table of Contents
1. [Project Directory Structure](#project-directory-structure)
2. [catalyst.json Configuration](#catalystjson)
3. [.catalystrc — Project Identity File](#catalystrc--project-identity-file)
4. [catalyst-config.json for Functions](#catalyst-configjson-for-functions)
5. [catalyst-config.json for Client](#catalyst-configjson-for-client)
6. [CLI Installation & Login](#cli-installation--login)
7. [CLI Commands Reference](#cli-commands-reference)
8. [Local Testing](#local-testing)
9. [Deployment](#deployment)
10. [Environments](#environments)

---

## Project Directory Structure

When `catalyst init` is run, a standard project layout is created. Claude must always produce code that
matches this structure exactly, or deployment will fail.

```
my-catalyst-project/              # Project root (home directory)
├── catalyst.json                 # Auto-generated project config
├── functions/                    # All server-side functions
│   ├── function_name_1/          # Each function is its own directory
│   │   ├── index.js              # Entry point (Node.js)
│   │   ├── catalyst-config.json  # Function-specific config
│   │   ├── package.json          # NPM dependencies
│   │   └── node_modules/         # Dependencies (auto-installed)
│   └── function_name_2/
│       ├── main.py               # Entry point (Python)
│       ├── catalyst-config.json
│       └── requirements.txt
├── client/                       # Web client (frontend)
│   ├── index.html                # Entry HTML file
│   ├── css/
│   ├── js/
│   ├── images/
│   └── catalyst-config.json      # Client config
└── appsail/                      # AppSail services (optional)
    ├── app.js                    # Express/Hapi/Koa etc.
    ├── catalyst-config.json
    └── package.json
```

Key rules:
- The `functions/` directory name is fixed and cannot be renamed
- The `client/` directory name is fixed and cannot be renamed
- Each function must be in its own subdirectory under `functions/`
- Each function directory must contain its own `catalyst-config.json`
- The web client must have `index.html` at its root level
- `catalyst.json` at the project root is auto-generated — never create it manually

---

## catalyst.json

`catalyst.json` at the project root holds **deployment configuration** — which functions, AppSail
services, and Slate apps to deploy. It is created and updated by the CLI; do not create it manually.

```json
{
  "functions": {
    "targets": ["myFunction1", "myFunction2"],
    "ignore": [],
    "source": "functions"
  },
  "appsail": {
    "targets": ["my-appsail-service"],
    "source": "appsail"
  },
  "slate": [
    { "name": "my-frontend", "source": "/absolute/path/to/client" }
  ]
}
```

Key rules:
- The `functions` block **must** include `targets`, `ignore`, and `source` — omitting any field
  causes the CLI error `"targets was found to be empty"`.
- Slate `source` **must be an absolute path**. A relative path causes:
  `"You are not in a catalyst app directory"`.
- `catalyst.json` is for deployment configuration only. Project identity (project ID, env ID,
  timezone) lives in `.catalystrc` — see the section below.

---

## .catalystrc — Project Identity File

Created automatically by `catalyst init` at the project root. Contains the actual project metadata
required for `catalyst deploy` and `catalyst push` to work.

```json
{
  "project_id": "4074000000163001",
  "project_domain": "myapp-60019947973.development",
  "env_id": "60019947973",
  "timezone": "Asia/Kolkata"
}
```

Key points:
- Created automatically by `catalyst init` — do not create manually.
- Contains `project_id`, `env_id`, `project_domain`, and `timezone`.
- If this file is missing or deleted, `catalyst deploy` will fail.
- Do not confuse with `catalyst.json` which holds deployment targets, not project identity.

---

## catalyst-config.json for Functions

Every function directory must contain this file. It tells Catalyst how to deploy and execute the function.

⚠️ **Format:** The config must use a nested `function` key. A flat top-level structure (`name`/`type`/`stack` at root) will fail.

⚠️ **First deploy prerequisite:** Having a directory + `catalyst-config.json` is NOT enough for the first deploy. Run `catalyst functions:add` from the project root first (interactive — enter name, type, stack when prompted). This registers the function in `catalyst.json`. Only then will `catalyst deploy --only functions` work. Subsequent deploys do not need this step again.

### Node.js Advanced I/O Function:
```json
{
  "function": {
    "name": "my_advanced_io_function",
    "type": "advancedio",
    "stack": "node20",
    "entry_point": "index.js"
  }
}
```

### Node.js Basic I/O Function:
```json
{
  "function": {
    "name": "my_basic_io_function",
    "type": "basicio",
    "stack": "node20",
    "entry_point": "index.js"
  }
}
```

### Event Function:
```json
{
  "function": {
    "name": "my_event_function",
    "type": "event",
    "stack": "node20",
    "entry_point": "index.js"
  }
}
```

### Job Function:
```json
{
  "function": {
    "name": "my_job_function",
    "type": "job",
    "stack": "node20",
    "entry_point": "index.js"
  }
}
```

### Python Function:
```json
{
  "function": {
    "name": "my_python_function",
    "type": "basicio",
    "stack": "python39",
    "entry_point": "main.py"
  }
}
```

### Java Function:
```json
{
  "function": {
    "name": "my_java_function",
    "type": "basicio",
    "stack": "java17",
    "entry_point": "com.example.Main"
  }
}
```

**Supported stacks:**
- Node.js: `node20` *(recommended — actively supported)*, `node14`, `node16`, `node18` *(legacy — no upstream security patches)*
- Java: `java8`, `java11`, `java17`
- Python: `python39`

**Memory options:** 128, 256, 384, 512, 640, 768, 896, 1024 (in MB) — configured in the `deployment` block. (Applies to Functions only; AppSail memory is configured in the console: 256–2048 MB.)

---

## catalyst-config.json for Client

```json
{
  "name": "my_web_client"
}
```

---

## CLI Installation & Login

```bash
# Install CLI globally
npm install -g zcatalyst-cli

# Verify installation
catalyst --version

# Login to Zoho account (opens browser for OAuth)
catalyst login

# Check logged-in user
catalyst whoami

# Logout
catalyst logout
```

Prerequisites: Node.js v20 (LTS) and NPM.

---

## CLI Commands Reference

### Project Management
```bash
catalyst init                        # Initialize project in current directory
catalyst init --project "MyProject"  # Init with specific project name
catalyst projects:list               # List all projects in account
catalyst use                         # Switch active project
catalyst reset                       # Reset the project association
```

### Function Management
```bash
catalyst functions:setup             # Set up functions after init
catalyst functions:add               # Add a new function
catalyst functions:shell             # Launch Node.js shell for testing
catalyst functions:delete            # Delete a function
catalyst functions:configure-memory  # Configure function memory
```

### Client Management
```bash
catalyst client:setup                # Set up web client
catalyst client:delete               # Delete web client
```

### AppSail Management
```bash
catalyst appsail:add                 # Add AppSail service
```

### Data Store
```bash
catalyst data:import                 # Import data into Data Store
catalyst data:export                 # Export data from Data Store
catalyst data:status                 # Check import/export status
```

### Slate
```bash
catalyst slate:init                  # Initialize Slate app
catalyst slate:create                # Create new Slate app
catalyst slate:link                  # Link existing Slate app
catalyst slate:unlink                # Unlink Slate app
```

### Pull & Export/Import
```bash
catalyst pull                        # Pull resources from remote
catalyst export                      # Export project as ZIP
catalyst import                      # Import project from ZIP
```

---

## Local Testing

```bash
# Serve all resources locally (functions + client)
catalyst serve

# Serve only functions
catalyst serve --only functions

# Serve only client
catalyst serve --only client

# Specify custom port
catalyst serve --port 3000

# Launch function shell for testing
catalyst functions:shell
```

When serving locally:
- Functions are available at `http://localhost:<port>/server/<function_name>/execute`
- Client is available at `http://localhost:<port>/app/index.html`
- The serve command connects to the remote Catalyst console for Data Store, File Store, etc.

### What works locally vs what doesn't

| Feature | Works with `catalyst serve`? | Notes |
|---------|------------------------------|-------|
| Basic I/O functions | Yes | Full local execution |
| Advanced I/O functions | Yes | Full local execution |
| Event functions | No | Triggered by platform events only |
| Cron functions | No | Triggered by scheduler only |
| Job functions | No | Triggered by Job Scheduling only |
| Data Store / ZCQL | Yes | Connects to remote Development environment |
| Cache | Yes | Connects to remote Development environment |
| Stratus | Yes | Connects to remote Development environment |
| Web Client | Yes | Served on localhost |
| AppSail | Partial | Use `node server.js` locally; no Catalyst auth layer |

For functions that can't be tested locally, deploy to Development and test there.
Use Tunneling (`catalyst tunnel`) to expose your local server to Catalyst for
webhook/event testing.

---

## Deployment

```bash
# Deploy all resources (functions + client + appsail)
catalyst deploy

# Deploy only functions — correct flag: --only functions (space, not --only-functions)
catalyst deploy --only functions

# Deploy only a specific function
catalyst deploy --only functions:my_function

# Deploy only client
catalyst deploy --only client

# Deploy only AppSail
catalyst deploy --only appsail

# Deploy to Slate
catalyst slate:deploy
```

⚠️ **Flag syntax (CLI v1.23.0+):** Use `--only <target>` with a space. The hyphenated `--only-functions` / `--only-client` forms do NOT exist and will throw "unknown option".

### Deployment package limits

| Component | Max package size | What's counted |
|-----------|-----------------|----------------|
| Function (zip) | 100 MB | Code + node_modules + assets |
| AppSail | 250 MB | Full build directory including node_modules |
| Slate (frontend) | 400 MB | Frontend build output (HTML/CSS/JS/assets) |

**To reduce function package size:**
- Use `npm install --production` to exclude devDependencies
- Add unnecessary files to `.catalystignore`
- Consider splitting large functions into smaller, focused ones
- Remove test files, documentation, and examples from node_modules

### Rolling back a deployment

Catalyst does not have a one-click rollback. To revert a bad deployment:

1. **From git:** Check out the previous working commit and redeploy:
```bash
   git checkout <previous-commit>
   catalyst deploy
```
2. **From console:** For AppSail, the console shows deployment history — you can
   redeploy a previous revision
3. **Blue/green via API Gateway:** Route traffic between two function versions
   by updating API Gateway routes

**Best practice:** Tag your git commits before each deployment so you can quickly
identify which version to roll back to.

### Slate — Manual Setup (without interactive slate:link)

`catalyst slate:link` is interactive-only and cannot be scripted or piped. To set up Slate
in automated or non-interactive environments:

**Step 1** — Create `.catalyst/slate-config.toml` inside the client directory:

```toml
framework = "static"
deployment_name = "default"
```

**Step 2** — Add the slate entry to `catalyst.json` (absolute source path required):

```json
"slate": [
  { "name": "my-frontend", "source": "/absolute/path/to/client" }
]
```

**Step 3** — Deploy:

```bash
catalyst deploy slate -m "initial deploy"
```

Slate URL format: `https://<project-domain>.onslate.in`
Example: `https://myapp-60019947973.development.onslate.in`

---

## Environments

Catalyst has two environments:

1. **Development (sandbox)**: Where CLI deploys go. Free to use within limits. Used for testing.
2. **Production**: Requires billing setup. Serves live traffic. Deployed separately from the console.

To deploy to production:
1. Set up billing in the Catalyst console (Settings → Billing)
2. Deploy to production from the console (not from CLI)
3. Production gets its own URL and domain mapping

The CLI always works with the Development environment. Production deployment is done through the web console.

### Dev-to-Prod promotion checklist

1. **Verify in Development** — all functions, AppSail services, and frontend working correctly
2. **Set up billing** — Catalyst Console → Settings → Billing (required for Production)
3. **Deploy to Production** — Console → Deploy → select Production environment
4. **Update environment variables** — Production uses separate env vars. Set them in the
   Console for each function and AppSail service
5. **Swap ZAIDs** — Production has different ZAIDs than Development. Update any hardcoded
   ZAID references in your code or use environment variables instead
6. **Reconfigure social login** — OAuth redirect URLs must point to the Production domain
7. **Map custom domain** — Console → Domain Mapping → add your production domain
8. **Test the full flow** — auth, data, file uploads, email — all on the Production URL
9. **Monitor** — enable APM and Application Alerts for Production

**Common pitfall:** Using Development ZAIDs in Production is the #1 cause of auth failures
after deployment. Always use environment variables for ZAIDs, never hardcode them.
