# Game Crafter - AI Web Game Development Agent

![Game Crafter Cover](cover.jpg)

An intelligent sandbox-based agent that generates complete, playable HTML5 games with cyberpunk-style visuals.

> **📚 Demo Focus**: This example demonstrates how to use **Hooks** to process **Sandbox output**, creating a complete workflow from AI-generated code to automatically published web pages.

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
# Create a simple game
yao agent test -n yao.craft -i "Create a Snake game"

# Chinese
yao agent test -n yao.craft -i "做一个贪吃蛇游戏"
```

## About

Game Crafter uses **Sandbox Mode** to run Claude CLI in a Docker container. Sandbox mode provides:

- Full file system access to create and modify files
- Command execution capability in an isolated environment
- Persistent workspace for building complete applications

## Hooks and Sandbox Output Integration

The core of this example demonstrates how to use **Hooks** to process **Sandbox output**, enabling automated post-processing workflows.

### Why Hooks?

In Sandbox mode, the AI generates files (like `game.zip`) inside a container, but these files need further processing to become accessible applications:

| Phase | Executor | Input | Output |
|-------|----------|-------|--------|
| Code Generation | Sandbox (Claude) | User request | `game.html`, `game.css`, `game.js`, `game.zip` |
| Post-processing | **Next Hook** | `game.zip` | SUI page source |
| Publishing | **Next Hook** | SUI source | Accessible `/ai/{chat_id}` page |

### How Hooks Work

This example uses two Hooks:

#### 1. Create Hook - Initialization

```typescript
// src/index.ts
export function Create(ctx: agent.Context, messages: agent.Message[]): agent.Create {
  log.Info("Game Crafter: Create hook started");

  // Store chat_id and start_time in memory for later use
  ctx.memory.context.Set("chat_id", ctx.chat_id);
  ctx.memory.context.Set("start_time", Date.now());

  return { messages };
}
```

#### 2. Next Hook - Process Sandbox Output ⭐

This is the **core demonstration** of this example:

```typescript
// src/index.ts
export function Next(ctx: agent.Context, payload: agent.Payload): agent.Next {
  const { error } = payload;

  // Handle errors
  if (error) {
    log.Error(`Sandbox execution error: ${error}`);
    return { data: { status: "error", message: error } };
  }

  // Check if sandbox is available
  if (!ctx.sandbox) {
    return null;
  }

  // 1. Read game files from sandbox workspace
  const gameFiles = readGameFiles(ctx);
  if (!gameFiles) {
    return null;
  }

  // 2. Get chat_id for unique page route
  const chatId = ctx.memory.context.Get("chat_id") || ctx.chat_id;

  // 3. Create SUI page from game files
  const pageResult = createGamePage(chatId, gameFiles);
  if (!pageResult.success) {
    ctx.Send(`❌ Failed to create game page: ${pageResult.error}`);
    return { data: { status: "error", message: pageResult.error } };
  }

  // 4. Build/compile the SUI page
  const buildResult = buildGamePage(pageResult.route);

  // 5. Navigate user to game URL
  ctx.Send({
    type: "action",
    props: { name: "navigate", payload: { route: pageResult.url } },
  });

  ctx.Send(`\n\n🎮 **Game Ready!** [Open Game](${pageResult.url})`);

  return { data: { status: "success", game_url: pageResult.url } };
}
```

### Key Technical Points

| Technique | Description | Code Location |
|-----------|-------------|---------------|
| Read Sandbox Files | Read files from host workspace | `utils.ts: readGameFiles()` |
| JS Code Wrapping | Wrap code in `init()` function to ensure DOM ready | `sui.ts: wrapJsForSui()` |
| Font Extraction | Extract Google Fonts from HTML head | `sui.ts: extractFontImports()` |
| SUI Page Creation | Call Yao SUI processors | `sui.ts: createGamePage()` |
| Page Compilation | Generate accessible static pages | `sui.ts: buildGamePage()` |

### What is Sandbox Mode?

Sandbox mode is a special execution mode in Yao Agent that runs an AI coding assistant (like Claude CLI) inside a Docker container. The container follows a **stateless container + persistent workspace** model:

| Component | Lifecycle | Storage |
|-----------|-----------|---------|
| **Container** | Per-request, disposable | None (stateless) |
| **Workspace** | Persistent across requests | `{YAO_DATA_ROOT}/sandbox/workspace/{user}/chat-{chat_id}/` |
| **Session** | Managed by Claude CLI | `/workspace/.claude/` |
| **History** | Managed by Yao | Yao's session store |

The workspace directory is **mounted from the host**, so:
- Files persist across requests and container restarts
- Generated games can be accessed directly from the host
- Container can be recreated anytime without losing state

### Multi-turn Conversation Support

Sandbox mode supports seamless multi-turn conversations through Claude CLI's native session management:

| Request Type | Session Detection | Message Handling | Claude CLI Flag |
|--------------|-------------------|------------------|-----------------|
| **First Request** | No `.claude/projects/` | Send all messages | (none) |
| **Continuation** | `.claude/projects/` exists | Send only last user message | `--continue` |

**How it works:**

1. **First request**: All messages are sent to Claude CLI, which creates session data in `/workspace/.claude/`
2. **Continuation**: Yao detects existing session files and uses `--continue` flag, sending only the latest user message
3. **Session storage**: By setting `HOME=/workspace`, Claude CLI stores session data in the workspace, persisting across container restarts

```
First Request:                          Continuation:
┌─────────────────┐                     ┌─────────────────┐
│  All Messages   │                     │ Last User Msg   │
│  + System Prompt│                     │ + --continue    │
└────────┬────────┘                     └────────┬────────┘
         │                                       │
         ▼                                       ▼
┌─────────────────┐                     ┌─────────────────┐
│   Claude CLI    │                     │   Claude CLI    │
│  (new session)  │                     │ (resume session)│
└────────┬────────┘                     └────────┬────────┘
         │                                       │
         ▼                                       ▼
┌─────────────────────────────────────────────────────────┐
│  /workspace/.claude/projects/-workspace/                 │
│  ├── sessions-index.json  (session metadata)            │
│  └── {session-id}.jsonl   (conversation history)        │
└─────────────────────────────────────────────────────────┘
```

This approach:
- **Reduces token usage**: Only the latest message is sent on continuation
- **Preserves full context**: Claude CLI manages the complete conversation history
- **Enables iterative development**: Users can request changes without re-explaining the project

### Sandbox Configuration

```jsonc
// package.yao
{
  "sandbox": {
    "command": "claude",           // Claude CLI executor
    "timeout": "10m",              // Max execution time
    "arguments": {
      "max_turns": 20,             // Max conversation turns
      "permission_mode": "bypassPermissions"  // Auto-approve file operations
    }
  }
}
```

### Sandbox API

Hooks can interact with the sandbox environment via `ctx.sandbox`:

```typescript
// src/index.ts
export function Next(ctx: agent.Context, payload: agent.Payload): agent.Next {
  if (ctx.sandbox) {
    // Read files from sandbox workspace
    const content = ctx.sandbox.ReadFile("game.html");
    
    // Write files to sandbox workspace
    ctx.sandbox.WriteFile("config.json", '{"version": 1}');
    
    // List directory contents
    const files = ctx.sandbox.ListDir(".");
    
    // Execute shell commands
    const output = ctx.sandbox.Exec(["unzip", "game.zip"]);
  }
  return null;
}
```

## Architecture

```
┌─────────────────────────────────────────────────────────────┐
│                      Yao Agent System                        │
├─────────────────────────────────────────────────────────────┤
│                                                              │
│  ┌──────────────┐    ┌──────────────┐    ┌──────────────┐  │
│  │   Create     │───▶│   Sandbox    │───▶│   Complete   │  │
│  │    Hook      │    │  (Claude)    │    │    Hook      │  │
│  └──────────────┘    └──────────────┘    └──────────────┘  │
│                             │                               │
└─────────────────────────────┼───────────────────────────────┘
                              │ mount
                              ▼
┌─────────────────────────────────────────────────────────────┐
│  Docker Container (Stateless)                                │
│  ┌────────────────────────────────────────────────────────┐ │
│  │  /workspace (mounted from host)                         │ │
│  │  ┌─────────┐  ┌─────────┐  ┌─────────────────┐         │ │
│  │  │game.html│  │ assets/ │  │  Other files... │         │ │
│  │  └─────────┘  └─────────┘  └─────────────────┘         │ │
│  └────────────────────────────────────────────────────────┘ │
└─────────────────────────────────────────────────────────────┘
                              │
                              ▼
┌─────────────────────────────────────────────────────────────┐
│  Host Filesystem (Persistent)                                │
│  {YAO_DATA_ROOT}/sandbox/workspace/{user}/chat-{chat_id}/   │
└─────────────────────────────────────────────────────────────┘
```

### Execution Flow

```
User Request
     │
     ▼
┌─────────────────────────────────────────────────────────────┐
│  1. Create Hook                                              │
│     - Store chat_id, start_time                             │
└─────────────────────────┬───────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│  2. Sandbox Execution (Claude CLI)                          │
│     - Create game.html, game.css, game.js                   │
│     - Compress into game.zip                                │
└─────────────────────────┬───────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│  3. Next Hook (src/index.ts)                                │
│     - Read game files from sandbox workspace                │
│     - Extract HTML, CSS, JS content                         │
│     - Create and save SUI page via sui.page.save            │
│     - Build page via sui.build.page                         │
│     - Navigate user to game URL                             │
└─────────────────────────┬───────────────────────────────────┘
                          │
                          ▼
┌─────────────────────────────────────────────────────────────┐
│  4. User plays game at /ai/{chat_id}                        │
└─────────────────────────────────────────────────────────────┘
```

## Output Format

Games are created as **three separate files**:

| File | Content |
|------|---------|
| `game.html` | HTML structure, links to CSS/JS |
| `game.css` | All styles (cyberpunk theme) |
| `game.js` | Game logic and rendering |

These are then:
1. Created by Claude CLI in sandbox workspace
2. Read and processed by Next Hook
3. Published as SUI page at `/ai/{chat_id}`

## Design System

### Cyberpunk Color Palette

| Color | Hex | Usage |
|-------|-----|-------|
| Primary | `#00F0FF` | Main UI, borders, text |
| Secondary | `#FF00E5` | Accents, highlights |
| Accent | `#00FF88` | Success, power-ups |
| Warning | `#FFE500` | Alerts, special items |
| Background | `#0A0A0F` | Main background |
| Surface | `#12121A` | Elevated elements |

### Visual Effects

- **Neon Glow**: `box-shadow: 0 0 10px #00F0FF, 0 0 20px #00F0FF40`
- **Scan Lines**: CSS overlay for retro-tech feel
- **Glitch Effects**: CSS animations for titles
- **Sharp Corners**: 2-4px border-radius

### Typography

- **Title Font**: Orbitron (bold, 2.5rem)
- **UI Font**: Share Tech Mono (1.5rem)
- **Effects**: Letter-spacing, text-shadow glow

## Required Game Features

Every generated game includes:

| Feature | Description |
|---------|-------------|
| Start Screen | Title, instructions, start button |
| Game Over | Final score, restart button |
| Score Display | Real-time score counter |
| Pause | Press 'P' or 'Escape' |
| Controls | Arrow keys or WASD |
| Responsive | Centered, handles resize |

## File Structure

```
assistants/yao/craft/
├── package.yao          # Agent configuration (sandbox settings)
├── prompts.yml          # System prompt with design guidelines
├── README.md            # This file
├── cover.jpg            # Cover image
├── src/
│   ├── index.ts         # Create/Next hooks
│   ├── utils.ts         # Utility functions (file reading, language detection)
│   ├── utils_test.ts    # Unit tests for utils
│   ├── sui.ts           # SUI page creation and building
│   └── sui_test.ts      # Unit tests for SUI functions
├── tests/
│   └── inputs.jsonl     # Agent test cases
├── pages/
│   └── README.md        # Pages documentation
└── locales/
    ├── en-us.yml        # English translations
    └── zh-cn.yml        # Chinese translations
```

## SUI Integration

The Next hook uses these SUI processes to create game pages:

| Process | Purpose |
|---------|---------|
| `sui.page.save` | Create and save page source (HTML, CSS, JS) |
| `sui.build.page` | Build/compile the page to static files |

Generated games are accessible at:
```
/ai/{chat_id}
```

The `chat_id` ensures each conversation has its own unique game page.

## Testing

```bash
# Run unit tests for utils
yao agent test -i scripts.yao.craft.utils

# Run unit tests for SUI functions
yao agent test -i scripts.yao.craft.sui

# Test with direct message
yao agent test -n yao.craft -i "Create a Snake game"
```

## Example Games

Ask Game Crafter to create:

- **Classic Games**: Snake, Tetris, Pong, Breakout, Flappy Bird
- **Arcade Games**: Space Invaders, Asteroids, Pac-Man style
- **Puzzle Games**: Memory match, Sliding puzzle, Minesweeper
- **Casual Games**: Clicker games, Endless runners

## Workflow

Game Crafter follows a structured workflow:

1. **Understand** → Analyze game requirements
2. **Design** → Plan architecture and visuals
3. **Implement** → Write HTML/CSS/JS code
4. **Package** → Create game files in workspace
5. **Publish** → Next Hook creates SUI page
6. **Play** → User navigates to game URL

The AI explains each step as it works, keeping users informed.

## Configuration Reference

### package.yao Options

| Option | Description |
|--------|-------------|
| `connector` | LLM connector (volcengine.glm-4_7) |
| `sandbox.command` | Executor type (claude) |
| `sandbox.timeout` | Max execution time |
| `sandbox.arguments.max_turns` | Max conversation turns |
| `sandbox.arguments.permission_mode` | File permission handling |

### Permission Modes

| Mode | Description |
|------|-------------|
| `bypassPermissions` | Auto-approve all operations |
| `acceptEdits` | Auto-approve edits, prompt for others |
| `default` | Prompt for all operations |

## See Also

- [Agent Context API](../../ai-docs/agent-context-api.md) - Sandbox API reference
- [Scribe Agent](../scribe/README.md) - Multi-agent workflow example
