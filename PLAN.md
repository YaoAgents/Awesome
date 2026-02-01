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
├── README.md                    # GitHub homepage (auto-generated)
├── agents.json                  # All agents index (website/CLI reads this)
├── categories.json              # Category definitions
├── CONTRIBUTING.md              # Contribution guide
├── scripts/                     # Automation scripts
│   ├── build-readme.js          # Generate README.md list section from agents.json
│   └── validate-pr.js           # PR validation script
└── agents/                      # Detail directories for each agent
    ├── yaoagents/
    │   ├── yao-craft/
    │   │   ├── meta.json
    │   │   ├── README.md
    │   │   ├── README.zh-CN.md
    │   │   └── assets/
    │   └── ...
    └── <owner>/
        └── <repo>/
            └── ...
```

---

## Website Pages

| Page | Data Source | Features |
|------|-------------|----------|
| `/agents` list page | `agents.json` | Recommended display, search, category filter |
| `/agents/<id>` detail page | `agents/<id>/README.md` + `meta.json` | Full intro, screenshots, install command |

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
Add agents/<owner>/<repo>/ directory
    ├── meta.json
    ├── README.md
    └── assets/
    ↓
CI auto-validation
    ↓
Manual review (required for featured)
    ↓
Auto-update agents.json + README.md after merge
```

---

## TODO

- [ ] Create initial `agents.json` data
- [ ] Create `categories.json`
- [ ] Create `CONTRIBUTING.md` contribution guide
- [ ] Implement `scripts/validate-pr.js` PR validation
- [ ] Implement `scripts/build-readme.js` auto-generation
- [ ] Add meta.json and README for existing Agents
- [ ] Website `/agents` list page
- [ ] Website `/agents/<id>` detail page
