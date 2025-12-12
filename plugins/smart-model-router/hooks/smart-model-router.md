# Smart Model Router Pre-Routing

**INSTRUCTION: Before implementation, always activate `smart-model-router` to choose the best model (capability-first).**

Workflow:

1. If user explicitly specifies a model (e.g., "use Opus / use Gemini / use GPT-5.2"), respect user preference and skip routing.
2. Otherwise, activate the skill first:
   Skill(smart-model-router)
3. Let smart-model-router output model selection + reasoning + optimized prompt,
   then execute the task according to routing result.

Capability-first routing rules (determined by smart-model-router):

- Frontend/UI/Design tasks -> Gemini (highest priority)
- Large context/cross-module understanding -> Gemini
- Architecture/Security-critical/High-complexity design -> Opus
- Requires Claude-specific features -> Sonnet
- Code review default -> GPT-5.2 (lightweight/real-time review can use Haiku)
- Other tasks default -> GPT-5.2
