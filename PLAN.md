# Awesome Repository Plan

## Purpose

Yao Agents ecosystem index center, serving:
- **Website Agents Page** — List display, search, category filter
- **Website Detail Page** — Show complete intro for single Agent
- **Yao CLI** — `yao agent add <id>` queries this list
- **README.md** — Readable index on GitHub

---

## Directory Structure

```
Awesome/
├── README.md                    # GitHub homepage
├── agents.json                  # All agents index (website/CLI reads this)
├── categories.json              # Category definitions
├── CONTRIBUTING.md              # Contribution guide
├── AGENT.md                     # Agent data schema
├── PLAN.md                      # This file
├── scripts/                     # Automation scripts
│   ├── build-readme.js          # Generate README.md list section
│   └── validate-pr.js           # PR validation script
├── assistants/                  # Assistants (reactive, conversational)
│   ├── README.md                # Type introduction
│   └── <owner>/
│       ├── README.md            # Organization introduction
│       └── <name>/
│           ├── README.md
│           ├── meta.json
│           └── cover.jpg
├── robots/                      # Autonomous Agents (proactive, triggered)
│   ├── README.md
│   └── <owner>/
│       └── ...
└── mcps/                        # MCP Tools (capability modules)
    ├── README.md
    └── <owner>/
        └── ...
```

---

## Website Pages

| Page | Data Source | Features |
|------|-------------|----------|
| `/agents` list page | `agents.json` | Recommended display, search, category filter |
| `/agents/<id>` detail page | `<type>/<owner>/<name>/README.md` + `meta.json` | Full intro, screenshots, install command |

### List Page Features
- **Recommended Section**: Agents with `featured: true`, after manual review
- **Search**: By name, description, tags
- **Filter**: By type (robot/assistant/mcp), tags

---

## Update Flow

### Internal Auto-Publish

```
Yao Autonomous Agent executes task
    ↓
Publish to GitHub (YaoAgents org)
    ↓
Trigger Webhook → Auto-create PR to Awesome
    ↓
CI validation → Auto-merge
    ↓
Update agents.json + README.md
```

### External Community Contribution

```
Author submits PR
    ↓
Add <type>/<owner>/<name>/ directory
    ├── meta.json
    ├── README.md
    └── cover.jpg
    ↓
CI auto-validation
    ↓
Manual review (required for featured)
    ↓
Auto-update agents.json + README.md after merge
```

---

## TODO

- [x] Create initial `agents.json` data
- [x] Create `categories.json`
- [x] Create `AGENT.md` data schema
- [x] Add meta.json and README for existing Assistants
- [x] Organize by type: assistants/, robots/, mcps/
- [x] Add type and organization README files
- [ ] Create `CONTRIBUTING.md` contribution guide
- [ ] Implement `scripts/validate-pr.js` PR validation
- [ ] Implement `scripts/build-readme.js` auto-generation
- [ ] Website `/agents` list page
- [ ] Website `/agents/<id>` detail page
