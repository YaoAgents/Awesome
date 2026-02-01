# Robot Inspiration Agent - P0 Phase

An intelligent agent that analyzes context, time markers, and available resources to generate actionable inspiration reports for autonomous Robot execution systems.

![Robot Inspiration Agent](cover.jpg)

> **Agent Type**: `robot-inspiration`
>
> This agent can be selected in **Mission Control → Agent → Advanced Settings → Developer Options → Inspiration** dropdown. The `type` field in `package.yao` determines which execution phase this agent is available for.

**Development Status**

| AI Planning | AI Development | AI Testing | AI Review | Manual Review | Manual Testing |
|:-----------:|:--------------:|:----------:|:---------:|:-------------:|:--------------:|
| ✅ Done | ✅ Done | ✅ Done | ✅ Done | ✅ Done | ✅ Done |

**Availability**

| Downloadable | Installable | Manual Testable |
|:------------:|:-----------:|:---------------:|
| ✅ Yes | ✅ Yes | ✅ Yes |

**Agent Type**

| Type | Selectable In |
|:----:|:-------------:|
| `robot-inspiration` | Mission Control → Agent → Advanced Settings → Developer Options → Inspiration |

## Quick Start

```bash
# Test with a sales analyst scenario (Chinese)
yao agent test -n yao.robot-inspiration -i tests/inputs.jsonl -v

# Run a single test case
yao agent test -n yao.robot-inspiration -i tests/inputs.jsonl --run "T001" -v

# Extract results for review
yao agent extract tests/output-*.jsonl
```

## About This Agent

This agent is the **P0 (Inspiration) Phase** of the Robot execution pipeline. It receives structured context about time, robot identity, and available resources, then generates a focused inspiration report that guides subsequent phases (P1: Goals → P2: Tasks → P3: Run → P4: Deliver).

### Key Features

| Feature | Description |
|---------|-------------|
| **Time-Aware** | Understands time markers (weekend, month-end, quarter-end) for priority setting |
| **Duty-Constrained** | Only recommends actions within the Robot's defined duties and rules |
| **Resource-Mapped** | Maps recommendations to specific available Agents and MCP tools |
| **Bilingual** | Supports both English and Chinese input/output |

### Input Structure

The agent expects structured markdown input with:

1. **Time Context** - Current datetime, timezone, time markers
2. **Robot Identity** - Role, duties, rules
3. **Available Resources** - Agents, MCP Tools, Knowledge Base, Database Models

### Output Structure

```markdown
## Summary
[Situation assessment and primary focus]

## Key Insights
- **[High]** [Urgent item]
- **[Medium]** [Important item]
- **[Low]** [Optional item]

## Recommended Actions
1. **[Action]**: [Description]
   - Why: [Reason]
   - Using: [Agent/MCP tool]

## Resource Mapping
| Action | Executor | Type | Notes |
|--------|----------|------|-------|
| ... | ... | ... | ... |

## Risks & Considerations
- [Potential issues]
```

## Use Cases

| Scenario | Time Marker | Example Output |
|----------|-------------|----------------|
| Sales Analyst | Friday + Month End | Monthly report generation, KPI verification |
| SEO Specialist | Monday Morning | Weekly keyword research, content planning |
| Customer Service | Weekend | Urgent ticket prioritization, escalation |
| Research Analyst | Quarter End | Quarterly report completion, deadline focus |
| Financial Analyst | Year End | Annual report, reconciliation |

## Configuration

### Agent Type

The `type` field in `package.yao` determines which Robot execution phase this agent can be used for:

| Type | Phase | Description |
|------|-------|-------------|
| `robot-inspiration` | P0 | Discover insights and generate inspiration report |
| `robot-goals` | P1 | Generate goals from inspiration |
| `robot-tasks` | P2 | Split goals into executable tasks |
| `robot-delivery` | P4 | Format and deliver results |
| `robot-learning` | P5 | Extract insights from execution |

When you set `"type": "robot-inspiration"`, this agent will appear in **Mission Control → Agent → Advanced Settings → Developer Options → Inspiration** dropdown, allowing users to select it as a custom P0 phase agent.

### Connector Options

```json
{
  "connector": "deepseek.v3",
  "connector_options": {
    "optional": true,
    "connectors": ["deepseek.v3", "openai.gpt-4o"],
    "filters": ["tool_calls"]
  }
}
```

### Model Settings

| Setting | Value | Description |
|---------|-------|-------------|
| `temperature` | 0.5 | Balanced creativity vs consistency |
| `max_tokens` | 4096 | Sufficient for detailed reports |
| `search` | disabled | External info via MCP resources |

## File Structure

```
assistants/yao/robot-inspiration/
├── package.yao      # Agent configuration
├── prompts.yml      # System prompts
├── README.md        # This file
├── cover.jpg        # Cover image
└── tests/
    ├── inputs.jsonl # Test cases
    └── *.md         # Extracted results
```

## Test Cases

| ID | Scenario | Language | Key Assertions |
|----|----------|----------|----------------|
| T001 | Sales Analyst - Month End Friday | Chinese | Monthly report, data-analyst |
| T002 | SEO Specialist - Monday Morning | English | Keyword research, content-writer |
| T003 | Customer Service - Weekend | Chinese | Urgent tickets, helpdesk |
| T004 | Research Analyst - Quarter End | English | Quarterly report, arxiv |
| T005 | Data Entry - Minimal Resources | English | File processing only |
| T006 | Financial Analyst - Year End | Chinese | Annual report, forecaster |

## Integration

This agent is designed to be called by the Robot Executor during the P0 phase:

```go
// In robot/executor/standard/inspiration.go
report, err := executor.RunInspiration(ctx, robot, clock)
```

The output `InspirationReport` feeds into the P1 (Goals) phase where specific goals are extracted and prioritized.

## Related

- [Robot System Design](../../../docs/robot/DESIGN.md)
- [Robot Technical Reference](../../../docs/robot/TECHNICAL.md)
- [Agent Testing Guide](../../../ai-docs/testing.md)
