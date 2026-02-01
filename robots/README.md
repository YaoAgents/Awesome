# Autonomous Agents (Robots)

Proactive AI workers that operate independently. Robots are triggered by schedules, events, or external signals, executing tasks without continuous human supervision.

## Characteristics

- **Proactive**: Initiates actions based on triggers
- **Autonomous**: Works independently once configured
- **Trigger-driven**: Activated by schedule, email, event, or webhook
- **Pipeline-based**: Follows P0→P1→P2→P3→P4 execution phases

## Execution Phases

| Phase | Name | Description |
|-------|------|-------------|
| P0 | Inspiration | Analyze context, generate insights |
| P1 | Goals | Convert insights to prioritized goals |
| P2 | Tasks | Decompose goals into executable tasks |
| P3 | Run | Execute tasks, validate results |
| P4 | Deliver | Format and deliver outputs |

## Installation

```bash
yao agent add <owner>/<name>
```

## Directory Structure

```
robots/
└── <owner>/              # Organization or user
    ├── README.md         # Organization introduction
    └── <name>/           # Individual robot
        ├── README.md     # Robot documentation
        ├── meta.json     # Metadata
        └── cover.jpg     # Cover image
```

## Available Robots

See subdirectories for available robots organized by owner.
