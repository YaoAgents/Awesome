# Robot Goals Agent - P1 Phase

An intelligent agent that converts inspiration reports or trigger inputs into prioritized, actionable goals with clear validation criteria for autonomous Robot execution systems.

![Robot Goals Agent](cover.jpg)

> **Agent Type**: `robot-goals`
>
> This agent can be selected in **Mission Control → Agent → Advanced Settings → Developer Options → Goals** dropdown. The `type` field in `package.yao` determines which execution phase this agent is available for.

**Development Status**

| AI Planning | AI Development | AI Testing | AI Review | Manual Review | Manual Testing |
|:-----------:|:--------------:|:----------:|:---------:|:-------------:|:--------------:|
| ✅ Done | ✅ Done | ✅ Done | ✅ Done | ⏳ Pending | ⏳ Pending |

**Availability**

| Downloadable | Installable | Manual Testable |
|:------------:|:-----------:|:---------------:|
| ✅ Yes | ✅ Yes | ✅ Yes |

**Agent Type**

| Type | Selectable In |
|:----:|:-------------:|
| `robot-goals` | Mission Control → Agent → Advanced Settings → Developer Options → Goals |

## Quick Start

```bash
# Test with all scenarios
yao agent test -n yao.robot-goals -i tests/inputs.jsonl -v

# Run a single test case
yao agent test -n yao.robot-goals -i tests/inputs.jsonl --run "T001" -v

# Extract results for review
yao agent extract tests/output-*.jsonl
```

## About This Agent

This agent is the **P1 (Goals) Phase** of the Robot execution pipeline. It receives input from various trigger types and generates structured, prioritized goals that guide subsequent phases (P2: Tasks → P3: Run → P4: Deliver).

### Key Features

| Feature | Description |
|---------|-------------|
| **Strategic Focus** | Generates high-level GOALS, not tasks (P2 handles decomposition) |
| **Deliverable-Driven** | Every goal specifies a concrete, tangible output |
| **Multi-Trigger Support** | Handles Clock (P0 output), Human, and Event triggers |
| **Priority-Aware** | Assigns [High], [Normal], [Low] based on urgency and context |
| **Decomposable** | Goals designed to be split into executable tasks by P2 |
| **Smart Delivery** | Extracts delivery targets from Human requests only |
| **Bilingual** | Supports both English and Chinese input/output |

### Input Types

| Trigger Type | Source | Contains |
|--------------|--------|----------|
| **Clock** | P0 Inspiration Phase | Time context, Inspiration report, Robot identity |
| **Human** | User Intervention | Action, Messages, Robot identity |
| **Event** | External Systems | Source, Event type, Data, Robot identity |

### Output Structure

```json
{
  "content": "Brief summary (UI title)\n\n### 1. [High] Goal Title\n**Objective**: Business purpose - why this matters\n**Deliverable**: Concrete output (PDF report, data package, etc.)\n**Success Criteria**: Measurable completion criteria\n**Resources**: Which agents/tools can be used"
}
```

**Key Fields**:
- **Objective**: Business purpose - why this goal matters
- **Deliverable**: Concrete, tangible output that will be produced
- **Success Criteria**: Measurable criteria to verify completion
- **Resources**: Available agents/MCP tools (reference for P2 decomposition)

**Note**: `delivery` is only included for Human triggers when user explicitly specifies recipients.

## Delivery Logic

| Trigger Type | Include `delivery`? | Recipients Source |
|--------------|---------------------|-------------------|
| Clock Trigger | ❌ Omit | Robot's default config |
| Human Trigger | ✅ If user specifies | Extract from message |
| Event Trigger | ❌ Omit | Robot's default config |

**Example Human Request**:
> "Analyze Q4 sales and send the report to Zhang Manager"

**Extracted Delivery**:
```json
"delivery": {
  "type": "email",
  "recipients": ["zhang.manager"],
  "format": "markdown"
}
```

## Goals vs Tasks

Understanding the difference is critical:

| Aspect | Goal (P1 - This Agent) | Task (P2 - Next Phase) |
|--------|------------------------|------------------------|
| Level | Strategic (WHAT to achieve) | Tactical (HOW to achieve) |
| Focus | Outcome & Deliverable | Executor & Instructions |
| Example | "Produce monthly sales report with trend analysis" | "Call data-analyst to query January sales data" |

**Bad Goal** (too task-like):
> Query sales database and aggregate January data using data-analyst

**Good Goal** (strategic with deliverable):
> Produce monthly sales performance report with trend analysis and KPI status

## Use Cases

| Scenario | Trigger | Key Deliverable |
|----------|---------|-----------------|
| Sales Analyst - Month End | Clock | PDF report with sales summary and KPI anomaly list |
| SEO Specialist - Monday | Clock | Keyword list and content plan for the week |
| Customer Service - Weekend | Clock | Urgent ticket resolution summary |
| User Task Request | Human | Specific report delivered to requested recipients |
| New Lead Created | Event | Lead qualification assessment |
| Financial Analyst - Year End | Clock | Annual financial report for approval |

## Configuration

### Agent Type

The `type` field in `package.yao` determines which Robot execution phase this agent can be used for:

| Type | Phase | Description |
|------|-------|-------------|
| `robot-inspiration` | P0 | Discover insights and generate inspiration report |
| `robot-goals` | P1 | Convert insights into prioritized goals |
| `robot-tasks` | P2 | Split goals into executable tasks |
| `robot-delivery` | P4 | Format and deliver results |
| `robot-learning` | P5 | Extract insights from execution |

When you set `"type": "robot-goals"`, this agent will appear in **Mission Control → Agent → Advanced Settings → Developer Options → Goals** dropdown.

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
| `temperature` | 0.4 | Lower for consistent structured output |
| `max_tokens` | 4096 | Sufficient for detailed goals |
| `search` | disabled | External info via MCP resources |

## File Structure

```
assistants/yao/robot-goals/
├── package.yao      # Agent configuration
├── prompts.yml      # System prompts
├── README.md        # This file
├── cover.jpg        # Cover image
└── tests/
    ├── inputs.jsonl # Test cases
    └── *.md         # Extracted results
```

## Test Cases

| ID | Scenario | Trigger | Language | Key Assertions |
|----|----------|---------|----------|----------------|
| T001 | Sales Analyst - Month End | Clock | Chinese | JSON structure, priorities, resources |
| T002 | SEO Specialist - Monday | Clock | English | Content goals, validation criteria |
| T003 | Customer Service - Weekend | Clock | Chinese | Urgent handling, no out-of-scope |
| T004 | User Task Request | Human | English | Delivery extraction, management target |
| T005 | New Lead Created | Event | English | Lead qualification, follow-up goals |
| T006 | Financial Analyst - Year End | Clock | Chinese | Annual report, no payroll access |

## Priority Guidelines

| Priority | Criteria | Examples |
|----------|----------|----------|
| **[High]** | Time-sensitive, blocking, explicit deadlines | Month-end reports, urgent tickets |
| **[Normal]** | Important but not urgent, regular duties | Weekly analysis, content creation |
| **[Low]** | Nice-to-have, can be deferred | Documentation, optimization |

## Integration

This agent is called by the Robot Executor during the P1 phase:

```go
// In robot/executor/standard/goals.go
goals, err := executor.RunGoals(ctx, robot, trigger)
```

The output `Goals` feeds into the P2 (Tasks) phase where goals are broken down into specific executable tasks.

## Related

- [Robot Inspiration Agent (P0)](../robot-inspiration/README.md)
- [Robot System Design](../../../docs/robot/DESIGN.md)
- [Robot Technical Reference](../../../docs/robot/TECHNICAL.md)
- [Agent Testing Guide](../../../ai-docs/testing.md)
