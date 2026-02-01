# Robot Validation Agent - P3 Phase

An intelligent agent that performs semantic validation of task execution results against expected outputs and validation rules for autonomous Robot execution systems.

![Robot Validation Agent](cover.jpg)

> **Agent Type**: `robot-validation`
>
> This agent can be selected in **Mission Control → Agent → Advanced Settings → Developer Options → Validation** dropdown. The `type` field in `package.yao` determines which execution phase this agent is available for.

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
| `robot-validation` | Mission Control → Agent → Advanced Settings → Developer Options → Validation |

## Quick Start

```bash
# Test with all scenarios
yao agent test -n yao.robot-validation -i tests/inputs.jsonl -v

# Run a single test case
yao agent test -n yao.robot-validation -i tests/inputs.jsonl --run "T001" -v

# Extract results for review
yao agent extract tests/output-*.jsonl
```

## About This Agent

This agent is the **Validation Component** of the Robot execution pipeline (used during P3 Run phase). It receives task definitions and their execution results, then performs semantic validation to determine if the results meet the expected criteria.

### Key Features

| Feature | Description |
|---------|-------------|
| **Semantic Validation** | Evaluates results based on meaning, not just format |
| **Quality Scoring** | Provides 0.0-1.0 score reflecting result quality |
| **Issue Detection** | Identifies specific problems with actionable details |
| **Rule Compliance** | Checks against explicit validation rules |
| **JSON Output** | Returns structured validation result for automation |

### Input Structure

The agent receives structured validation prompts containing:

1. **Task Section** - Task ID, Executor, Instructions, expected_output, validation_rules
2. **Result Section** - Actual output from task execution (JSON or text)
3. **Success Criteria** - Overall criteria for task success

### Output Structure

```json
{
  "passed": true,
  "score": 0.95,
  "issues": [],
  "details": "Result contains all required elements with good quality."
}
```

### Fields

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `passed` | boolean | Yes | Whether result meets minimum criteria |
| `score` | number | Yes | Quality score from 0.0 to 1.0 |
| `issues` | string[] | Yes | List of specific problems (empty if none) |
| `details` | string | No | Brief explanation of assessment |

### Score Ranges

| Range | Meaning |
|-------|---------|
| 1.0 | Perfect, exceeds expectations |
| 0.8-0.99 | Good, minor improvements possible |
| 0.6-0.79 | Acceptable, some issues |
| 0.4-0.59 | Marginal, significant issues |
| 0.0-0.39 | Unacceptable, fails requirements |

## Validation Criteria

The agent evaluates results based on:

1. **Semantic Completeness**: Does result address all expected output aspects?
2. **Rule Compliance**: Does result satisfy all validation rules?
3. **Quality Assessment**: Is content accurate, relevant, and well-formatted?
4. **Practical Usability**: Can the result be used for its intended purpose?

## Use Cases

| Scenario | Task Type | Validation Focus |
|----------|-----------|------------------|
| Data Analysis | Agent | Correct data structure, required fields |
| Report Generation | Agent | Content completeness, formatting |
| Database Query | MCP | Expected records, column presence |
| API Call | MCP | Status code, response structure |
| File Processing | Process | Output file existence, format |

## Configuration

### Agent Type

The `type` field in `package.yao` determines which Robot execution phase this agent can be used for:

| Type | Phase | Description |
|------|-------|-------------|
| `robot-inspiration` | P0 | Discover insights and generate inspiration report |
| `robot-goals` | P1 | Convert insights into prioritized goals |
| `robot-tasks` | P2 | Decompose goals into executable tasks |
| `robot-validation` | P3 | Validate task execution results |
| `robot-delivery` | P4 | Format and deliver results |
| `robot-learning` | P5 | Extract insights from execution |

When you set `"type": "robot-validation"`, this agent will appear in **Mission Control → Agent → Advanced Settings → Developer Options → Validation** dropdown.

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
| `temperature` | 0.2 | Lower for consistent validation judgments |
| `max_tokens` | 2048 | Sufficient for validation details |
| `search` | disabled | Validates only provided content |

## File Structure

```
assistants/yao/robot-validation/
├── package.yao      # Agent configuration
├── prompts.yml      # System prompts
├── README.md        # This file
├── cover.jpg        # Cover image
└── tests/
    ├── inputs.jsonl # Test cases
    └── *.md         # Extracted results
```

## Test Cases

| ID | Scenario | Expected Outcome |
|----|----------|------------------|
| T001 | Complete result matching all requirements | passed=true, score≥0.9 |
| T002 | Missing required fields/regions | passed=false, score<0.5 |
| T003 | Basic but acceptable result | passed=true, score≈0.6 |
| T004 | MCP tool query result validation | passed=true, score≥0.9 |
| T005 | Empty but valid result | passed=true (context-dependent) |

## Integration

This agent is called by the Robot Executor's Validator during the P3 phase:

```go
// In robot/executor/standard/validator.go
validationAgentID := "__yao.validation" // default
if robot.Config.Resources.Phases["validation"] != "" {
    validationAgentID = robot.Config.Resources.Phases["validation"]
}

result, err := caller.CallWithMessages(ctx, validationAgentID, validationPrompt)
validation := v.ParseAgentResult(result)
```

The validation result (`passed`, `score`, `issues`) is used to determine whether the task succeeded and whether to retry or escalate.

## Related

- [Robot Tasks Agent (P2)](../robot-tasks/README.md)
- [Robot Goals Agent (P1)](../robot-goals/README.md)
- [Robot Inspiration Agent (P0)](../robot-inspiration/README.md)
- [Robot System Design](../../../docs/robot/DESIGN.md)
- [Robot Technical Reference](../../../docs/robot/TECHNICAL.md)
- [Agent Testing Guide](../../../ai-docs/testing.md)
