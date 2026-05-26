# Contributing to Catalyst Agent Skills

Thanks for your interest in improving Catalyst agent skills! This guide covers how to contribute.

## How to contribute

### Reporting issues

If the skill produces incorrect code, recommends deprecated services, or misses a Catalyst feature:

1. Open an [Issue](../../issues)
2. Include the prompt you used and the response you got
3. Describe what was wrong and what the correct answer should be

### Improving the skill

1. Fork the repository
2. Edit the `SKILL.md` or reference files in `skills/references/`
3. Test the updated skill by importing it into your AI tool
4. Open a Pull Request with a clear description of what changed and why

### What makes a good contribution

- **Accuracy fixes**: Correcting wrong SDK patterns, handler signatures, or API details
- **New service coverage**: Adding documentation for newly launched Catalyst services
- **Better examples**: Adding real-world code patterns that work out of the box
- **Deprecation updates**: Updating deprecated service warnings as EOL dates pass
- **MCP tool coverage**: Adding new Zoho MCP tool patterns as they become available

### Skill folder structure

```
skills/
├── SKILL.md                              ← Main skill file (triggers, context, quick reference)
└── references/                           ← Detailed reference files
    ├── cloud-scale.md                    ← Data Store, Stratus, NoSQL, Cache, Auth, etc.
    ├── functions-and-sdk.md              ← Function types, Node.js/Java/Python SDK patterns
    ├── architecture-patterns.md              ← Use-case → service mapping, architecture blueprints
    ├── services.md                       ← AppSail, Circuits, Signals, Slate, Pipelines, etc.
    ├── equivalents-aws.md                ← AWS Lambda/S3/RDS → Catalyst mapping
    ├── equivalents-gcp.md                ← GCP Cloud Run/Pub-Sub → Catalyst mapping
    ├── equivalents-azure.md              ← Azure Functions/Blob → Catalyst mapping
    ├── equivalents-firebase.md           ← Firebase Auth/Firestore → Catalyst mapping
    ├── equivalents-heroku.md             ← Heroku/Railway/Render → Catalyst mapping
    ├── equivalents-supabase.md           ← Supabase full-stack BaaS → Catalyst mapping
    ├── equivalents-vercel-netlify.md     ← Vercel/Netlify serverless → Catalyst mapping
    ├── meta-ids.md                       ← Project ID, ZAID, Table ID — where to find them
    ├── pricing.md                        ← Unit prices, free tiers, cost estimation
    ├── project-and-cli.md                ← CLI commands, project structure
    ├── zoho-mcp-tools.md                 ← LLM-driven resource management via Zoho MCP
    ├── sdk-nodejs.md                     ← Complete Node.js SDK code examples
    ├── sdk-java.md                       ← Complete Java SDK code examples
    ├── sdk-python.md                     ← Complete Python SDK code examples
    ├── sdk-web.md                        ← Web SDK v4 client-side (auth, Data Store, Stratus)
    ├── sdk-mobile.md                     ← Android, iOS, Flutter SDK reference
    ├── signals-deep-dive.md              ← Signals event bus (publishers, rules, targets)
    ├── smartbrowz-deep-dive.md           ← SmartBrowz (headless, grid, templates, dataverse)
    ├── job-scheduling-deep-dive.md       ← Job pools, crons, SDK examples, REST APIs
    ├── devops-deep-dive.md               ← APM, logs, automation testing, alerts
    └── cli-reference.md                  ← Full CLI command map, Slate frameworks, safety
```

### Guidelines

- Keep the `SKILL.md` description field under 1024 characters
- Test your changes by importing the skill and running prompts against it
- Don't remove existing reference files without discussion — add new ones instead
- Keep code examples deployment-ready (correct handler signatures, proper config files)

## Code of conduct

Be respectful, constructive, and helpful. We're all here to make Catalyst development better.

## Plugin manifests — keep in sync

When updating the skill description, update ALL four manifest files to keep them identical:
- `.claude-plugin/plugin.json`
- `.claude-plugin/marketplace.json`
- `.cursor-plugin/plugin.json`
- `gemini-extension.json`
