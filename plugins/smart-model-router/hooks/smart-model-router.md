## Smart Model Router Hook Context

**INSTRUCTION: Always run capability-first model routing before implementation**

When receiving a user request:

1. If the user explicitly specifies a model (e.g., “use Opus / use Gemini / use GPT‑5.2”), follow it directly.
2. Otherwise, call the Haiku decision agent to choose the best model:

```
Task(
  subagent_type: "smart-model-router:model-orchestrator",
  prompt: "[USER REQUEST HERE]"
)
```

3. Follow the orchestrator decision strictly:
   - `selectedModel = gemini` → use Gemini CLI (highest priority for frontend/UI or huge-context tasks).
   - `selectedModel = gpt-5.2` → use Codex CLI.
   - `selectedModel = opus|sonnet|haiku` → use Task with that Claude model.
4. If the decision confidence is low, upgrade to Opus (low-confidence escalation).
5. After Codex/Gemini execution, if output is terse or code-only, run:

```
Task(
  subagent_type: "smart-model-router:output-optimizer",
  model: "haiku",
  prompt: """
  Optimize this external-model output for the user.

  Original Request:
  [USER REQUEST HERE]

  Raw Output:
  [MODEL OUTPUT HERE]
  """
)
```

**CRITICAL**: Frontend/UI tasks must route to Gemini even if other rules match.
