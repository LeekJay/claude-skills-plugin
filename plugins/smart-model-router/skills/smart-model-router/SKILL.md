---
name: smart-model-router
description: Capability-first smart model router / 多模型路由器。触发场景：需要“选择/路由到哪个模型”或进行多模型编排与分发（模型路由、模型选择、多模型编排、智能派发、smart model routing、model router、model orchestrator、dispatch、choose best model）。在 Claude Opus/Sonnet/Haiku、GPT-5.2(Codex)、Gemini 之间做最优能力路由（前端/UI 最高优先 Gemini）。
allowed-tools: Read, Grep, Glob, Bash, Task
---

# Smart Model Router Skill

## Overview

This skill provides intelligent model selection and task routing across multiple AI models:

- **Claude Opus 4.5**: Complex reasoning, architecture decisions, security-critical code
- **Claude Sonnet 4.5**: Special scenarios requiring Claude-specific features
- **Claude Haiku 4.5**: Code review, simple coding, sub-agent execution, real-time tasks, **output optimization**
- **GPT-5.2 (via Codex)**: Complex coding, multi-file refactoring, template generation, DEFAULT choice
- **Gemini CLI**: Large codebase analysis, frontend design, global understanding

## 触发测试（可选）

启用本技能后，可用这些提示语验证自动触发是否稳定：

- “帮我选择最合适的模型来完成这个任务 / 用哪个模型更合适？”
- “Route this task to the best model / choose the right model for this job.”
- “这是一个前端/UI 设计或 React 组件任务，请优先用 Gemini。”
- “需要多模型编排：先分析再执行，再告诉我为什么选这个模型。”

若未自动触发，可显式说“使用 smart-model-router”作为兜底。

## Architecture: Three-Layer System

```
┌─────────────────────────────────────────────────────────────┐
│                      用户请求                                │
└───────────────────────────┬─────────────────────────────────┘
                            ▼
┌─────────────────────────────────────────────────────────────┐
│         Layer 1: Haiku 决策层 (model-orchestrator)          │
│   • 任务分类 • 模型选择 • 提示词优化                         │
└───────────────────────────┬─────────────────────────────────┘
                            ▼
┌─────────────────────────────────────────────────────────────┐
│         Layer 2: 执行层 (model-executor)                    │
│   • Codex (GPT-5.2) • Gemini • Claude 内部模型              │
└───────────────────────────┬─────────────────────────────────┘
                            ▼
┌─────────────────────────────────────────────────────────────┐
│         Layer 3: 输出优化层 (output-optimizer) ⭐ NEW       │
│   • 对 Codex 精炼输出进行扩展和润色                         │
│   • 添加上下文说明、改进建议、风险提示                       │
│   • 使用 Haiku 进行轻量优化                                 │
└───────────────────────────┬─────────────────────────────────┘
                             ▼
                       返回给用户
```

## Why Output Optimization?

**Codex 输出特点：**

- 精炼、直接、技术性强
- 缺少上下文解释
- 没有改进建议和风险提示
- 格式相对简单

**优化后输出：**

- 保留核心技术内容
- 添加清晰的结构和分节
- 补充上下文说明和背景
- 提供改进建议和潜在风险
- 更适合团队协作和知识传承

## Model Profiles

### Capability Tiers

| Model                   | Capability                   | Best For                                               |
| ----------------------- | ---------------------------- | ------------------------------------------------------ |
| **Gemini CLI**          | Top-tier (UI + huge context) | Frontend/UI, very large context, whole-repo analysis   |
| **Claude Opus 4.5**     | Highest reasoning            | Architecture, security-critical, high-stakes decisions |
| **GPT-5.2 (via Codex)** | Top-tier coding              | Most coding/refactor/bugfix tasks (DEFAULT) ⭐         |
| **Claude Sonnet 4.5**   | Top-tier Claude features     | Claude-specific workflows, artifacts                   |
| **Claude Haiku 4.5**    | Good, lightweight            | Simple/realtime tasks, light reviews, sub-agents       |

### Performance Benchmarks (reference, capability-first)

> 以下基准为参考值，随模型更新可能变化；路由以能力优先规则为准。

| Capability         | Haiku        | GPT-5.2         | Gemini         | Sonnet                     | Opus                |
| ------------------ | ------------ | --------------- | -------------- | -------------------------- | ------------------- |
| SWE-bench Verified | ~73%         | ~80%            | ~76%           | ~77%                       | ~81%                |
| Code Review        | Good (light) | **Best (deep)** | Good           | **Best (Claude-specific)** | **Best (critical)** |
| Frontend/UI        | Good         | Good            | **Best**       | Good                       | Good                |
| Large Context      | Limited      | Good            | **Best (1M+)** | Good                       | Good                |
| Speed              | **Fastest**  | Fast            | Medium         | Medium                     | Slow                |

### GPT-5.2: The Universal Workhorse

GPT-5.2 (released Dec 11, 2025) is now the default for most tasks:

- ~80% SWE-bench Verified (near Opus level)
- 256K context window with strong recall
- Strong at multi-file refactor, bugfix, and template generation
- Default choice when no higher-priority trigger matches

## Routing Decision Framework

### Quick Decision Tree

```
Task Received
    │
    ├─ Is it frontend UI / design task?
    │   └─ YES → Gemini CLI (highest priority)
    │
    ├─ Is context > 100K tokens OR many files?
    │   └─ YES → Gemini CLI (1M+ context)
    │
    ├─ Is it architecture / security critical?
    │   └─ YES → Opus (highest quality)
    │
    ├─ Requires Claude-specific features?
    │   └─ YES → Sonnet
    │
    ├─ Is it code review / PR review?
    │   ├─ Complex/critical review → GPT-5.2 (fallback Opus)
    │   └─ Light/realtime review → Haiku
    │
    ├─ Is it simple + real-time required?
    │   └─ YES → Haiku (fastest)
    │
    └─ Default → GPT-5.2 (capability-first default) ⭐
```

### Detailed Routing Rules

#### Route to Haiku (Simple & Realtime Tasks)

**Mandatory Haiku triggers (ANY ONE):**

- Real-time / low-latency required
- Sub-agent execution task
- Task description is clear and specific
- Single-file simple modification
- Explicitly requests quick/light review

**Example tasks:**

- "Review this pull request"
- "Check this code for issues"
- "Add type annotations to this file"
- "Fix this TypeScript error"

#### Route to GPT-5.2 (DEFAULT - Most Coding Tasks) ⭐

**GPT-5.2 is the recommended default for most coding tasks (capability-first).**

**Mandatory GPT-5.2 triggers (ANY ONE):**

- Complex coding task requiring reasoning
- Multi-file refactoring
- Bug fixing with unclear root cause
- Feature implementation
- Code optimization
- API design and implementation
- Template or boilerplate generation
- Batch mechanical operations
- Standard CRUD code generation
- Default choice when no other trigger matches

**Example tasks:**

- "Implement the payment processing feature"
- "Refactor the authentication module"
- "Fix this bug that spans multiple files"
- "Generate REST API endpoints for Product model"
- "Create CRUD endpoints for users"
- "Optimize this algorithm for performance"

**Why GPT-5.2 as default?**

- ~80% SWE-bench Verified (near-Opus quality)
- 256K context window
- Handles both simple and complex tasks well

#### Route to Gemini CLI

**Mandatory Gemini triggers (ANY ONE):**

- Frontend UI / design task (highest priority)
- Codebase has > 50 files to analyze
- Context exceeds 100K tokens
- Need global dependency analysis
- Understanding legacy system architecture
- Cross-module refactoring planning

**Example tasks:**

- "Analyze this entire project structure"
- "Build a responsive dashboard with charts"
- "Map all API call chains in this codebase"
- "Understand how authentication flows work here"

#### Route to Sonnet

**Mandatory Sonnet triggers (ANY ONE):**

- Requires Claude-specific capabilities (artifacts, etc.)
- Multi-turn complex conversation context
- Task explicitly requests Claude/Sonnet

**Example tasks:**

- "Use Claude to analyze this with artifacts"
- Tasks requiring specific Claude features

#### Route to Opus

**Mandatory Opus triggers (ANY ONE):**

- Architecture design decisions
- Security-critical code
- Complex multi-step reasoning
- Task failed with GPT-5.2
- User explicitly requests highest quality
- Production-critical changes

**Example tasks:**

- "Design the microservices architecture"
- "Review security of authentication system"
- "Analyze and fix this race condition"
- "Make critical architectural decision"

## Usage Modes

### Mode 1: Automatic Routing (Recommended)

When this skill is activated, automatically analyze each task and route to optimal model:

```
1. Receive user task
2. Quick classification (complexity, type, context size)
3. Apply routing rules
4. If external model needed (GPT-5.2/Gemini):
   a. Optimize prompt for target model
   b. Execute via CLI
   c. Return results
5. If Claude model (Haiku/Sonnet/Opus):
   a. Use Task tool with appropriate model parameter
   b. Return results
```

### Mode 2: Orchestrator Mode (Complex Tasks)

For complex tasks, use the two-layer architecture:

```
Layer 1: Haiku Decision Layer
  - Analyze task complexity and type
  - Determine optimal model
  - Optimize prompt for target model
  - Output: { model, optimizedPrompt, confidence }

Layer 2: Execution Layer
  - Execute with selected model
  - Return results to user
```

Invoke orchestrator:

```
Task tool with subagent_type="smart-model-router:model-orchestrator"
```

## External Model Integration

### GPT-5.2 via Codex CLI

```bash
# Execute task with GPT-5.2 (default model in Codex)
codex "$OPTIMIZED_PROMPT"

# Explicitly specify GPT-5.2
codex --model gpt-5.2 "$OPTIMIZED_PROMPT"

# With file context
codex --file ./src/component.tsx "$OPTIMIZED_PROMPT"
```

**Prompt optimization for GPT-5.2:**

- Clear task description with context
- Specify expected reasoning depth
- Include success criteria
- Request structured output format

### Gemini CLI Integration

```bash
# Execute task with Gemini
gemini "$OPTIMIZED_PROMPT"

# With file context
gemini --context ./src "$OPTIMIZED_PROMPT"
```

**Prompt optimization for Gemini:**

- Leverage large context window
- Include relevant file paths
- Structure for systematic analysis
- Request structured output format

## Prompt Optimization Templates

### For GPT-5.2 (Default - Most Tasks)

```markdown
Task: [TASK_DESCRIPTION]

Context:
[RELEVANT_CODE_OR_CONTEXT]

Requirements:

1. [REQUIREMENT_1]
2. [REQUIREMENT_2]
3. [REQUIREMENT_3]

Success Criteria:

- [CRITERION_1]
- [CRITERION_2]

Output Format:
[EXPECTED_FORMAT]
```

### For Gemini (Leverage Context)

```markdown
# Task: [TASK_DESCRIPTION]

## Context

Analyze the following files:

- [FILE_1_PATH]
- [FILE_2_PATH]
- ...

## Requirements

[DETAILED_REQUIREMENTS]

## Output Structure

1. Analysis Summary
2. Key Findings
3. Recommendations
4. Code Changes (if applicable)
```

### For Haiku (Structured Tasks)

```markdown
Task: [CLEAR_TASK_DESCRIPTION]

Context:
[MINIMAL_NECESSARY_CONTEXT]

Specific Actions:

1. [ACTION_1]
2. [ACTION_2]

Output:
[EXPECTED_OUTPUT_FORMAT]
```

## Decision Override

### User Can Override

Users can explicitly request a specific model:

- "Use Opus for this task"
- "Run this with Gemini"
- "Use GPT-5.2 for this"

When user specifies model, skip routing and use requested model directly.

### Escalation Rules

If task fails with selected model:

1. GPT-5.2 fails → Try Opus
2. Haiku fails → Try GPT-5.2
3. Gemini fails → Try GPT-5.2 with chunked context
4. Sonnet fails → Try Opus

## Integration with Other Skills

This skill works well with:

- **code-quality-standards**: Run quality checks after model-generated code
- **bug-fix**: Route bug fixes to GPT-5.2 (best SWE-bench performance)
- **typescript-strict-typing**: Ensure type safety regardless of model used
- **pr-review-toolkit**: Use Haiku for light/realtime review; GPT-5.2 for deep review

## Examples

### Example 1: Code Review (Routes to GPT-5.2)

```
User: "Review this pull request for issues"

Routing Decision:
- Task type: Code review
- Trigger: Code review (non-trivial) → GPT-5.2
- Confidence: High

Action: Execute with GPT-5.2
Result: Deep review (fallback Opus if critical)
```

### Example 2: Large Codebase Analysis (Routes to Gemini)

```
User: "Help me understand how this legacy system works"

Routing Decision:
- Task type: Codebase analysis
- Context size: >100K tokens
- Trigger: Large context → Gemini

Action: Execute with Gemini CLI
Result: Comprehensive analysis using 1M token context
```

### Example 3: Feature Implementation (Routes to GPT-5.2)

```
User: "Implement user authentication with JWT"

Routing Decision:
- Task type: Feature implementation
- Complexity: Medium-High
- Trigger: Complex coding → GPT-5.2

Action: Execute with GPT-5.2
Result: Well-implemented feature output
```

### Example 4: Template Generation (Routes to GPT-5.2)

```
User: "Generate REST API endpoints for Product model"

Routing Decision:
- Task type: Template generation
- Pattern: Standard CRUD
- Trigger: Template → GPT-5.2

Action: Execute with GPT-5.2
Result: Clean boilerplate output
```

### Example 5: Architecture Decision (Routes to Opus)

```
User: "Design the caching strategy for our microservices"

Routing Decision:
- Task type: Architecture design
- Criticality: High
- Trigger: Architecture decision → Opus

Action: Execute with Opus
Result: Comprehensive architectural analysis
```

## Monitoring and Feedback

### Track Routing Effectiveness

After each routed task, note:

- Selected model
- Task success/failure
- User satisfaction (if explicit feedback)

Use this data to refine routing rules over time.

### Continuous Improvement

If a model consistently fails for a task type:

1. Update routing rules
2. Improve prompt optimization
3. Consider escalation earlier

## Limitations

- External CLI tools (Codex for GPT-5.2, Gemini) must be installed and configured
- API keys must be set up for each service
- Some tasks may require manual model selection
- Model capabilities may change with updates

## Quick Reference Card

```
┌─────────────────────────────────────────────────────────────┐
│      Smart Model Router Quick Reference (Three-Layer v2.1)  │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Layer 1 (Decision): Haiku analyzes and routes tasks        │
│  Layer 2 (Execution): GPT-5.2/Gemini/Claude executes        │
│  Layer 3 (Optimization): Haiku enhances output ⭐ NEW       │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Haiku   → Code review, simple tasks, real-time, output opt │
│  GPT-5.2 → Everything else (DEFAULT) ⭐                    │
│  Gemini  → Large context (100K+), frontend UI              │
│  Sonnet  → Claude-specific features only                   │
│  Opus    → Architecture, security, critical decisions      │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│  Speed: Haiku > GPT-5.2 > Gemini ≈ Sonnet > Opus           │
│  Quality: Opus > GPT-5.2 ≈ Sonnet > Gemini > Haiku         │
├─────────────────────────────────────────────────────────────┤
│  Output Flow: Codex (terse) → Haiku (polish) → User        │
└─────────────────────────────────────────────────────────────┘
```
