# MCP Tools

Reusable capability modules based on Model Context Protocol. MCPs provide tools, resources, and prompts that can be used by Assistants and Robots.

## Characteristics

- **Modular**: Self-contained capability packages
- **Reusable**: Shared across multiple agents
- **Standardized**: Follows MCP specification
- **Flexible transport**: Supports process, HTTP, SSE, stdio

## MCP Components

| Component | Description |
|-----------|-------------|
| **Tools** | Callable functions for specific tasks |
| **Resources** | Data sources and configurations |
| **Prompts** | Reusable prompt templates |

## Installation

```bash
yao agent add <owner>/<name>
```

## Directory Structure

```
mcps/
└── <owner>/              # Organization or user
    ├── README.md         # Organization introduction
    └── <name>/           # Individual MCP package
        ├── README.md     # MCP documentation
        ├── meta.json     # Metadata
        └── cover.jpg     # Cover image (optional)
```

## Available MCP Tools

See subdirectories for available MCP tools organized by owner.
