# Mark - AI Visual Bookmark Agent

![Mark Cover](cover.jpg)

An intelligent sandbox-based agent that transforms URLs and text into beautiful 1920×1080 visual bookmark cards using HTML Canvas.

> **📚 Demo Focus**: This is a demo-level example that demonstrates how to combine **Sandbox + Chrome Browser + Visual Verification + Theme System** with Yao Hooks, showing a complete workflow from AI code generation → browser testing → vision-LLM quality gate → persistent storage → dynamic page display. It is meant as a developer reference for building Sandbox-powered agents, not a production-ready product.

**Development Status**

| AI Planning | AI Development | AI Testing | AI Review | Manual Review | Manual Testing |
|:-----------:|:--------------:|:----------:|:---------:|:-------------:|:--------------:|
| ✅ Done | ✅ Done | ✅ Done | ✅ Done | ✅ Done | ✅ Done |

**Availability**

| Downloadable | Installable | Manual Testable |
|:------------:|:-----------:|:---------------:|
| ✅ Yes | ✅ Yes | ✅ Yes |

## Quick Start

```bash
# Create a bookmark from a URL
yao agent test -n yao.mark -i "https://yaoagents.com/"

# Create a bookmark from a text snippet
yao agent test -n yao.mark -i "Rust's ownership system ensures memory safety without a garbage collector. Each value has exactly one owner, and when the owner goes out of scope, the value is dropped. Borrowing allows references without taking ownership, enforced at compile time."

# Chinese
yao agent test -n yao.mark -i "大语言模型（LLM）的核心是 Transformer 架构，通过自注意力机制捕捉长距离依赖关系。训练过程包括预训练和微调两个阶段，预训练使用海量文本数据学习语言规律，微调则针对特定任务进行优化。"
```

## About

Mark uses **Sandbox Mode** to run Claude CLI inside a Docker container equipped with Chrome, Xvfb, and Node.js. The sandbox provides:

- Chrome browser for real webpage browsing (CDP-based) and visual testing
- Full file system access to create, modify, and verify files
- Node.js scripts for screenshots (CDP), content extraction, and visual verification
- Persistent workspace across multi-turn conversations

## Hooks and Sandbox Output Integration

The core of this example demonstrates how to use **Hooks** to process **Sandbox output**, combining browser automation, visual verification, and persistent storage in a single workflow.

### Why Hooks?

In Sandbox mode, the AI generates files (`canvas.js`, `metadata.json`) inside a container, but these files need further processing to become accessible:

| Phase | Executor | Input | Output |
|-------|----------|-------|--------|
| Browse & Generate | Sandbox (Claude + Chrome) | URL or text | `canvas.js`, `metadata.json` |
| Visual Verify | Sandbox (verify.js) | Screenshot + theme rules | Pass/Fail verdict |
| Post-processing | **Next Hook** | `canvas.js`, `metadata.json` | Attachment + DB record |
| Display | **SUI Page** | DB query + attachment | Gallery with iframe preview |

### How Hooks Work

This example uses two Hooks:

#### 1. Create Hook - Initialization & Mode Detection

```typescript
// src/index.ts
export function Create(ctx: agent.Context, messages: agent.Message[]): agent.Create {
  // Store context for Next hook
  ctx.memory.context.Set("chat_id", ctx.chat_id);
  ctx.memory.context.Set("start_time", Date.now());
  ctx.memory.context.Set("user_input", lastMsg.content);

  // Write template, design.md, themes, scripts to sandbox
  writeAssetsToSandbox(ctx);

  // Open VNC preview for real-time monitoring
  openVncPreview(ctx);

  // Detect edit vs. create mode (checks URL for ?canvas=XXX)
  const existing = detectAndLoadExisting(ctx);
  if (existing) {
    return { messages, prompt_preset: "edit" }; // Use edit prompt
  }
  return { messages }; // Use default create prompt
}
```

#### 2. Next Hook - Process Sandbox Output ⭐

```typescript
// src/index.ts
export function Next(ctx: agent.Context, payload: agent.Payload): agent.Next {
  // 1. Read canvas.js and metadata.json from sandbox workspace
  const canvasFiles = readCanvasFiles(ctx);

  // 2. Save canvas.js as Yao attachment → file_id
  const fileId = saveJSAttachment(canvasFiles.js, `canvas_${Date.now()}.js`);

  // 3. Create or update DB record with metadata
  if (mode === "edit") {
    updateCanvasRecord(recordId, fileId, metadata, auth);
  } else {
    createCanvasRecord(canvasId, fileId, chatId, metadata, auth, userInput);
  }

  // 4. Navigate user to bookmark gallery
  ctx.Send({
    type: "action",
    props: { name: "navigate", payload: { route: listUrl } },
  });

  return { data: { status: "success", canvas_id: canvasId, file_id: fileId } };
}
```

### Key Technical Points

| Technique | Description | Code Location |
|-----------|-------------|---------------|
| CDP Content Extraction | Browse URL with Chrome DevTools Protocol | `assets/browse.js` |
| CDP Screenshots | Pixel-perfect screenshots via CDP | `assets/screenshot.js` |
| Visual Verification | Vision LLM (gpt-4o-mini) inspects screenshot against theme rules | `assets/verify.js` |
| Theme System | 10 content-domain themes with color, layout, typography specs | `assets/themes/*.md` |
| Attachment Storage | Save JS via `attachment.Save` processor | `src/store.ts` |
| Edit Mode Detection | Parse URL query for existing canvas_id | `src/utils.ts` |
| Robust JSON Parsing | `Process("json.Parse")` with auto-repair for LLM output | `src/utils.ts` |

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                      Yao Agent System                        │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐  │
│  │   Create     │───▶│   Sandbox    │───▶│    Next      │  │
│  │    Hook      │    │  (Claude)    │    │    Hook      │  │
│  └──────────────┘    └──────────────┘    └──────────────┘  │
│       │                     │                    │          │
│  Write assets        Generate code         Save output     │
│  Detect mode         Browse URL            Create record   │
│  Open VNC            Test in Chrome        Navigate user   │
│                      Verify visually                        │
└─────────────────────────────┼───────────────────────────────┘
                              │ mount
                              ▼
┌─────────────────────────────────────────────────────────────┐
│  Docker Container (yaoapp/sandbox-claude-chrome:latest)      │
│  ┌────────────────────────────────────────────────────────┐ │
│  │  /workspace (mounted from host)                         │ │
│  │  ├── template.html    (HTML shell)                      │ │
│  │  ├── design.md        (visual standards)                │ │
│  │  ├── CLAUDE.md        (workflow rules)                  │ │
│  │  ├── browse.js        (CDP content extraction)          │ │
│  │  ├── screenshot.js    (CDP screenshots)                 │ │
│  │  ├── verify.js        (vision LLM verification)         │ │
│  │  ├── themes/          (10 domain themes)                │ │
│  │  ├── canvas.js        ← generated output                │ │
│  │  └── metadata.json    ← generated output                │ │
│  └────────────────────────────────────────────────────────┘ │
│  Chrome + Xvfb (:99) + Node.js 22                           │
└─────────────────────────────────────────────────────────────┘
```

### Execution Flow

```
User Input (URL or text)
     │
     ▼
┌─────────────────────────────────────────────────────────────┐
│  1. Create Hook                                              │
│     - Write assets (template, design.md, themes, scripts)   │
│     - Detect create/edit mode                                │
│     - Open VNC preview                                       │
└─────────────────────────┬───────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│  2. Sandbox Execution (Claude CLI + Chrome)                  │
│     Phase 0: If URL → browse.js extracts content via CDP    │
│     Phase 1: Select theme, read design.md                    │
│     Phase 2: Generate canvas.js (Canvas 2D, 1920×1080)      │
│     Phase 3: Verify & Fix Loop (max 3 rounds)               │
│       ├── Syntax check (node --check)                       │
│       ├── Screenshot (screenshot.js via CDP)                 │
│       └── Visual verification (verify.js → vision LLM)      │
│     Phase 4: Write metadata.json                             │
└─────────────────────────┬───────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│  3. Next Hook                                                │
│     - Read canvas.js + metadata.json from sandbox           │
│     - Save canvas.js via attachment.Save → file_id          │
│     - Create/update DB record (title, tags, theme, etc.)    │
│     - Navigate user to gallery page                          │
└─────────────────────────┬───────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│  4. SUI Dynamic Page (/agents/yao.mark/index)                │
│     - Left panel: searchable bookmark list from DB          │
│     - Right panel: iframe preview with zoom & pan           │
│     - Links to source URL and user input                     │
└─────────────────────────────────────────────────────────────┘
```

## Visual Verification Pipeline

A key feature of this example is the **automated visual QA** pipeline:

```
canvas.js → Chrome → screenshot.png → verify.js → vision LLM → verdict
                                          │
                                 Reads design.md + theme.md
                                 Sends screenshot as base64
                                 Returns JSON: { pass, score, issues }
```

`verify.js` is a **zero-dependency** Node.js script that:

1. Reads the screenshot PNG and encodes it as base64
2. Loads `design.md` (general standards) and the selected `theme.md` (domain-specific rules)
3. Sends both to a vision-capable LLM (e.g. `gpt-4o-mini`) via OpenAI-compatible API
4. Returns a structured verdict: pass/fail, score (1-10), list of issues with severity

LLM credentials are injected via **sandbox secrets** (not hardcoded):

```jsonc
// package.yao
"secrets": {
  "LLM_API_KEY": "$ENV.OPENAI_API_KEY",
  "LLM_API_BASE": "https://api.openai.com/v1/",
  "LLM_MODEL": "gpt-4o-mini"
}
```

## Theme System

Mark automatically selects one of 10 content-domain themes based on the input:

| Theme | Domains | Key Visual Elements |
|-------|---------|---------------------|
| `tech` | Programming, AI/ML, DevOps | Circuit patterns, monospace fonts, neon accents |
| `science` | Physics, Chemistry, Biology | Molecular structures, data visualizations |
| `history` | History, Philosophy, Civilizations | Aged textures, serif fonts, sepia tones |
| `art` | Design, Architecture, Photography | Gallery layouts, creative typography |
| `business` | Finance, Startups, Marketing | Clean charts, professional palette |
| `nature` | Environment, Wildlife, Climate | Organic shapes, earth tones, leaf motifs |
| `health` | Medicine, Fitness, Nutrition | Soft gradients, calming colors, pulse icons |
| `culture` | Music, Film, Gaming, Sports | Bold colors, dynamic layouts |
| `education` | Tutorials, Courses, Academic | Structured layouts, notebook style |
| `lifestyle` | Food, Travel, Home, Hobbies | Warm tones, photo-card layouts |

Each theme file (`assets/themes/*.md`) defines: color palette, layout structure, typography, decorative elements, and content organization rules. Both `canvas.js` generation and `verify.js` verification use the selected theme.

## Deliverables

Claude produces exactly **2 files** in the sandbox:

| File | Content |
|------|---------|
| `canvas.js` | Pure vanilla JS using Canvas 2D API, 1920×1080 fixed viewport |
| `metadata.json` | `{ "title", "description", "tags", "theme", "source_url" }` |

## Storage

| Data | Storage Method |
|------|----------------|
| `canvas.js` content | `attachment.Save` → `file_id` |
| Canvas metadata | `models/canvas.mod.yao` database table |
| User input & source URL | Stored in DB record for reference |

## Sandbox API

Hooks interact with the sandbox environment via `ctx.sandbox`:

```typescript
// Read files from sandbox workspace
const content = ctx.sandbox.ReadFile("canvas.js");

// Write files to sandbox workspace
ctx.sandbox.WriteFile("template.html", htmlContent);

// List directory contents
const files = ctx.sandbox.ListDir(".");

// Make directory
ctx.sandbox.MakeDir("themes");
```

## What is Sandbox Mode?

Sandbox mode runs an AI coding assistant (Claude CLI) inside a Docker container. The container follows a **stateless container + persistent workspace** model:

| Component | Lifecycle | Storage |
|-----------|-----------|---------|
| **Container** | Per-request, disposable | None (stateless) |
| **Workspace** | Persistent across requests | `{YAO_DATA_ROOT}/sandbox/workspace/{user}/chat-{chat_id}/` |
| **Session** | Managed by Claude CLI | `/workspace/.claude/` |
| **History** | Managed by Yao | Yao's session store |

### Multi-turn Conversation Support

| Request Type | Session Detection | Message Handling | Claude CLI Flag |
|--------------|-------------------|------------------|-----------------|
| **First Request** | No `.claude/projects/` | Send all messages | (none) |
| **Continuation** | `.claude/projects/` exists | Send only last user message | `--continue` |

This enables iterative editing — users can say "change the color scheme" or "add more details" without re-explaining the content.

## File Structure

```
assistants/yao/mark/
├── package.yao              # Agent config (sandbox, secrets, connector)
├── prompts.yml              # System prompt (create mode)
├── README.md                # This file
├── cover.jpg                # Cover image
├── assets/
│   ├── template.html        # HTML shell (test harness + display shell)
│   ├── design.md            # Visual design standards (design bible)
│   ├── CLAUDE.md            # Workflow rules for Claude
│   ├── browse.js            # CDP-based webpage content extraction
│   ├── screenshot.js        # CDP-based screenshot capture
│   ├── verify.js            # Vision LLM visual verification (zero-dep)
│   └── themes/              # 10 content-domain themes
│       ├── tech.md
│       ├── science.md
│       ├── history.md
│       ├── art.md
│       ├── business.md
│       ├── nature.md
│       ├── health.md
│       ├── culture.md
│       ├── education.md
│       └── lifestyle.md
├── prompts/
│   └── edit.yml             # System prompt (edit mode)
├── models/
│   └── canvas.mod.yao       # Canvas data model
├── src/
│   ├── index.ts             # Create/Next hooks
│   ├── utils.ts             # Utility functions
│   └── store.ts             # DB and attachment operations
├── pages/
│   └── index/               # SUI gallery page
│       ├── index.html
│       ├── index.ts
│       ├── index.css
│       ├── index.json
│       ├── index.config
│       ├── index.backend.ts
│       └── __locales/
│           ├── en-us.yml
│           └── zh-cn.yml
├── locales/
│   ├── en-us.yml            # English translations
│   └── zh-cn.yml            # Chinese translations
└── tests/
    └── inputs.jsonl          # Test cases
```

## Configuration

### Sandbox Configuration

```jsonc
// package.yao
{
  "sandbox": {
    "image": "yaoapp/sandbox-claude-chrome:latest",
    "command": "claude",
    "timeout": "20m",
    "arguments": {
      "max_turns": 50,
      "permission_mode": "bypassPermissions"
    },
    "secrets": {
      "LLM_API_KEY": "$ENV.OPENAI_API_KEY",
      "LLM_API_BASE": "https://api.openai.com/v1/",
      "LLM_MODEL": "gpt-4o-mini"
    }
  }
}
```

| Option | Value | Description |
|--------|-------|-------------|
| `sandbox.image` | `yaoapp/sandbox-claude-chrome:latest` | Docker image with Chrome + Xvfb |
| `sandbox.command` | `claude` | Claude CLI executor |
| `sandbox.timeout` | `20m` | Max execution time |
| `sandbox.arguments.max_turns` | `50` | Max conversation turns |
| `sandbox.arguments.permission_mode` | `bypassPermissions` | Auto-approve file operations |
| `sandbox.secrets` | `LLM_*` | Vision LLM credentials for verify.js |

## Testing

```bash
# Test with URL input
yao agent test -n yao.mark -i "https://yaoagents.com/"

# Test with text snippet
yao agent test -n yao.mark -i "WebAssembly (Wasm) is a binary instruction format for a stack-based virtual machine. It enables deployment on the web for client and server applications, offering near-native performance and a compact binary format."

# Test with Chinese text snippet
yao agent test -n yao.mark -i "Go 语言的 goroutine 是轻量级线程，由 Go 运行时调度而非操作系统。通过 channel 实现 goroutine 间通信，遵循 CSP 并发模型，避免了共享内存带来的竞态问题。"
```

## See Also

- [Game Crafter Agent](../craft/README.md) - Sandbox game development example
- [Scribe Agent](../scribe/README.md) - Multi-agent workflow example
