# Agent Data Schema

Three types of agents have different metadata structures.

---

## Types

| Type | ID | Description | Runtime Data |
|------|-----|-------------|--------------|
| Autonomous Agent | `robot` | Proactive, scheduled tasks, event-triggered | `__yao.member` table (`member_type = "robot"`) |
| Assistant | `assistant` | Conversational, reactive | `package.yao` config file |
| MCP Tool | `mcp` | Capability modules for other agents | `.mcp.yao` config file |

---

## ID Format

Follows GitHub structure: `<owner>/<repo>`

- Official: `yaoagents/yao-craft`
- Community: `someuser/my-agent`

Install command:
```bash
yao agent add yaoagents/yao-craft
```

---

## Common Fields

Shared by all types:

| Field | Type | Required | Description |
|-------|------|:--------:|-------------|
| `id` | string | ✅ | Unique identifier, format `owner/repo` |
| `name` | i18n | ✅ | Display name |
| `description` | i18n | ✅ | One-line description |
| `type` | enum | ✅ | `robot` / `assistant` / `mcp` |
| `tags` | string[] | ✅ | Tags for categorization |
| `version` | string | ✅ | Semantic version |
| `github` | url | ✅ | GitHub repository URL |
| `author` | object | ✅ | Author info |
| `license` | string | ✅ | Open source license |
| `capabilities` | string[] | - | Capability list (for display) |
| `cover` | path | - | Cover image |
| `featured` | boolean | - | Whether recommended |
| `created_at` | datetime | ✅ | Creation time |
| `updated_at` | datetime | ✅ | Last update time |

### development — Development Status

| Field | Type | Description |
|-------|------|-------------|
| `ai_planning` | boolean | AI planning completed |
| `ai_development` | boolean | AI development completed |
| `ai_testing` | boolean | AI testing completed |
| `ai_review` | boolean | AI review completed |
| `manual_review` | boolean | Manual review completed |
| `manual_testing` | boolean | Manual testing completed |

### availability — Availability

| Field | Type | Description |
|-------|------|-------------|
| `downloadable` | boolean | Can be downloaded |
| `installable` | boolean | Can be installed (`yao agent add`) |
| `testable` | boolean | Can be tested (`yao agent test`) |

---

## Robot (Autonomous Agent)

Creates `__yao.member` record after installation, `member_type = "robot"`.

### Type-Specific Fields

| Field | Type | Description |
|-------|------|-------------|
| `triggers` | string[] | Trigger methods: `schedule` / `email` / `event` / `webhook` |
| `assistants` | string[] | Callable assistant list |
| `mcps` | string[] | Callable MCP tool list |
| `default_config` | object | Default config template (for installation reference) |

### meta.json Example

```json
{
  "id": "yaoagents/yao-robot-tasks",
  "name": {
    "en": "Task Executor",
    "zh-CN": "任务执行器"
  },
  "description": {
    "en": "Autonomous agent for executing scheduled tasks",
    "zh-CN": "用于执行定时任务的自主 Agent"
  },
  "type": "robot",
  "tags": ["automation", "tasks", "scheduling"],
  "version": "1.0.0",
  "development": {
    "ai_planning": true,
    "ai_development": true,
    "ai_testing": true,
    "ai_review": true,
    "manual_review": true,
    "manual_testing": true
  },
  "availability": {
    "downloadable": true,
    "installable": true,
    "testable": true
  },
  "github": "https://github.com/YaoAgents/yao-robot-tasks",
  "author": {
    "name": "Yao Team",
    "github": "YaoAgents"
  },
  "license": "Apache-2.0",
  "install": "yao agent add yaoagents/yao-robot-tasks",
  "capabilities": ["Task scheduling", "Email processing", "Event handling"],
  "robot": {
    "triggers": ["schedule", "email", "event"],
    "assistants": ["yaoagents/yao-scribe"],
    "mcps": ["yaoagents/mcp-email"],
    "default_config": {
      "autonomous_mode": true,
      "language_model": "deepseek.v3"
    }
  },
  "requirements": {
    "yao": ">=0.10.4"
  },
  "created_at": "2026-01-27T00:00:00Z",
  "updated_at": "2026-01-27T00:00:00Z"
}
```

---

## Assistant

Deploys `package.yao` + `prompts.yml` files after installation.

### Type-Specific Fields

| Field | Type | Description |
|-------|------|-------------|
| `modes` | string[] | Supported modes: `chat` / `task` |
| `sandbox` | object | Sandbox info (if Coding Agent) |
| `dependencies` | object | Dependencies on MCP or other Assistants |

### meta.json Example

```json
{
  "id": "yaoagents/yao-craft",
  "name": {
    "en": "Game Crafter",
    "zh-CN": "游戏工匠"
  },
  "description": {
    "en": "AI Web Game Development Agent - generates complete HTML5 games",
    "zh-CN": "AI 网页游戏开发助手 - 生成完整的 HTML5 游戏"
  },
  "type": "assistant",
  "tags": ["game", "development", "sandbox", "html5"],
  "version": "1.0.0",
  "development": {
    "ai_planning": true,
    "ai_development": true,
    "ai_testing": true,
    "ai_review": true,
    "manual_review": true,
    "manual_testing": true
  },
  "availability": {
    "downloadable": true,
    "installable": true,
    "testable": true
  },
  "github": "https://github.com/YaoAgents/yao-craft",
  "author": {
    "name": "Yao Team",
    "github": "YaoAgents"
  },
  "license": "Apache-2.0",
  "install": "yao agent add yaoagents/yao-craft",
  "test": "yao agent test -n yao.craft -i \"Create a Snake game\"",
  "capabilities": ["HTML5 game generation", "Interactive development", "SUI integration"],
  "assistant": {
    "modes": ["chat", "task"],
    "sandbox": {
      "type": "claude",
      "description": "Runs in isolated Docker container"
    }
  },
  "requirements": {
    "yao": ">=0.10.4"
  },
  "cover": "cover.jpg",
  "featured": true,
  "created_at": "2026-01-15T00:00:00Z",
  "updated_at": "2026-02-01T00:00:00Z"
}
```

---

## MCP (Tool)

Deploys `.mcp.yao` config file after installation.

### Type-Specific Fields

| Field | Type | Description |
|-------|------|-------------|
| `transport` | string | Transport type: `process` / `http` / `sse` / `stdio` |
| `tools` | string[] | Provided Tool list |
| `resources` | string[] | Provided Resource list |
| `prompts` | string[] | Provided Prompt list |

### meta.json Example

```json
{
  "id": "yaoagents/mcp-image-tools",
  "name": {
    "en": "Image Tools",
    "zh-CN": "图像工具包"
  },
  "description": {
    "en": "MCP tools for image generation and processing",
    "zh-CN": "用于图像生成和处理的 MCP 工具包"
  },
  "type": "mcp",
  "tags": ["image", "generation", "processing"],
  "version": "1.0.0",
  "development": {
    "ai_planning": true,
    "ai_development": true,
    "ai_testing": true,
    "ai_review": true,
    "manual_review": true,
    "manual_testing": true
  },
  "availability": {
    "downloadable": true,
    "installable": true,
    "testable": true
  },
  "github": "https://github.com/YaoAgents/mcp-image-tools",
  "author": {
    "name": "Yao Team",
    "github": "YaoAgents"
  },
  "license": "Apache-2.0",
  "install": "yao agent add yaoagents/mcp-image-tools",
  "capabilities": ["Text to image", "Image to image", "Upscaling"],
  "mcp": {
    "transport": "process",
    "tools": ["text2img", "img2img", "upscale"],
    "resources": ["models", "presets"],
    "prompts": []
  },
  "requirements": {
    "yao": ">=0.10.4"
  },
  "created_at": "2026-01-20T00:00:00Z",
  "updated_at": "2026-01-28T00:00:00Z"
}
```

---

## agents.json Example

Read by website list page and CLI.

```json
{
  "version": "1.0.0",
  "updated_at": "2026-02-01T12:00:00Z",
  "agents": [
    {
      "id": "yaoagents/yao-craft",
      "name": { "en": "Game Crafter", "zh-CN": "游戏工匠" },
      "description": { "en": "AI Web Game Development Agent", "zh-CN": "AI 网页游戏开发助手" },
      "type": "assistant",
      "tags": ["game", "development", "sandbox"],
      "cover": "agents/yaoagents/yao-craft/cover.jpg",
      "github": "https://github.com/YaoAgents/yao-craft",
      "version": "1.0.0",
      "development": {
        "ai_planning": true, "ai_development": true, "ai_testing": true,
        "ai_review": true, "manual_review": true, "manual_testing": true
      },
      "availability": { "downloadable": true, "installable": true, "testable": true },
      "author": { "name": "Yao Team", "github": "YaoAgents" },
      "featured": true,
      "created_at": "2026-01-15T00:00:00Z",
      "updated_at": "2026-02-01T00:00:00Z"
    },
    {
      "id": "yaoagents/yao-scribe",
      "name": { "en": "Scribe", "zh-CN": "Scribe" },
      "description": { "en": "AI Article Writing Agent", "zh-CN": "AI 文章写作助手" },
      "type": "assistant",
      "tags": ["writing", "content", "article"],
      "cover": "agents/yaoagents/yao-scribe/cover.jpg",
      "github": "https://github.com/YaoAgents/yao-scribe",
      "version": "1.0.0",
      "development": {
        "ai_planning": true, "ai_development": true, "ai_testing": true,
        "ai_review": true, "manual_review": true, "manual_testing": true
      },
      "availability": { "downloadable": true, "installable": true, "testable": true },
      "author": { "name": "Yao Team", "github": "YaoAgents" },
      "featured": true,
      "created_at": "2026-01-20T00:00:00Z",
      "updated_at": "2026-01-28T00:00:00Z"
    }
  ]
}
```

---

## categories.json Example

```json
{
  "types": [
    {
      "id": "robot",
      "name": { "en": "Autonomous Agents", "zh-CN": "自主 Agent" },
      "description": { "en": "Proactive AI workers with triggers", "zh-CN": "支持触发器的主动 AI 员工" },
      "icon": "smart_toy"
    },
    {
      "id": "assistant",
      "name": { "en": "Assistants", "zh-CN": "助手" },
      "description": { "en": "Conversational AI helpers", "zh-CN": "对话式 AI 助手" },
      "icon": "support_agent"
    },
    {
      "id": "mcp",
      "name": { "en": "MCP Tools", "zh-CN": "MCP 工具包" },
      "description": { "en": "Reusable capability modules", "zh-CN": "可复用的能力模块" },
      "icon": "extension"
    }
  ],
  "tags": [
    { "id": "game", "name": { "en": "Game", "zh-CN": "游戏" } },
    { "id": "development", "name": { "en": "Development", "zh-CN": "开发" } },
    { "id": "writing", "name": { "en": "Writing", "zh-CN": "写作" } },
    { "id": "automation", "name": { "en": "Automation", "zh-CN": "自动化" } },
    { "id": "image", "name": { "en": "Image", "zh-CN": "图像" } },
    { "id": "sandbox", "name": { "en": "Sandbox", "zh-CN": "沙箱" } }
  ]
}
```
