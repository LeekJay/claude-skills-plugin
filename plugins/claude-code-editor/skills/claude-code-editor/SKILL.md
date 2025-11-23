# Claude Code Editor Skill

## Overview

This skill provides intelligent code editing and refactoring capabilities through a hybrid architecture:
- **Skill mode**: Use directly in main conversation for simple, focused edits
- **Sub-agent mode**: Delegate complex, multi-file modifications to isolated context

## When to Use This Skill

### ✅ Use This Skill For:

**Large-Scale Refactoring:**
- Renaming functions/classes/variables across multiple files
- Restructuring directory layout and moving files
- Changing architectural patterns (e.g., converting class components to hooks)
- Extracting reusable utilities or components

**Batch Modifications:**
- Applying consistent changes across similar files
- Updating imports after file moves
- Converting code style or patterns project-wide
- Migrating from one library/API to another

**Pattern Replacements:**
- Replacing deprecated patterns with modern alternatives
- Standardizing error handling approaches
- Converting callback patterns to async/await
- Updating configuration formats

**Feature Implementation:**
- Implementing features that touch multiple files
- Adding new functionality that requires coordinated changes
- Integrating new libraries across the codebase

### ❌ Don't Use For:

- Single-file, small edits (use normal editing)
- Quick bug fixes in one location
- Adding simple comments or documentation
- Trivial formatting changes

## Decision Framework

Use this flowchart to decide when to delegate to sub-agent:

```
Is the task complex (3+ files or 5+ edits)?
  ├─ NO → Handle in main conversation (skill mode)
  └─ YES → Continue evaluation
      │
      Will it require multiple rounds of fixes?
        ├─ NO → Consider main conversation
        └─ YES → Continue evaluation
            │
            Will intermediate steps clutter conversation?
              ├─ NO → Main conversation is fine
              └─ YES → ✅ USE SUB-AGENT MODE
```

## Usage Modes

### Mode 1: Skill Mode (Main Conversation)

For focused, straightforward edits:

```markdown
When user asks for code modifications that are:
- Clearly scoped and well-defined
- 1-3 files maximum
- Unlikely to cause cascading errors
- Quick to verify

Handle directly in main conversation without spawning sub-agent.
```

**Example scenarios:**
- "Rename function `processData` to `transformData` in src/utils/data.ts and its tests"
- "Extract this code block into a new utility function"
- "Convert this component to use TypeScript"

### Mode 2: Sub-Agent Mode (Isolated Context)

For complex, multi-step modifications:

```markdown
When user asks for code modifications that are:
- Complex and multi-file
- Require systematic planning
- May need multiple fix iterations
- Will generate verbose intermediate output

Delegate to code-editor-executor sub-agent using:
subagent_type="claude-code-editor:code-editor-executor"
```

**Example scenarios:**
- "Refactor the authentication system to use JWT instead of sessions"
- "Move all components from src/components to src/ui and update all imports"
- "Convert all class components to functional components with hooks"
- "Implement a new feature: user preferences with backend, frontend, and tests"

## Sub-Agent Delegation

When delegating to sub-agent, use this format:

```typescript
// Use Task tool with proper subagent_type
subagent_type: "claude-code-editor:code-editor-executor"

// Provide clear task description
prompt: `
Task: [Clear description of what to modify]

Context:
- [Relevant context about the codebase]
- [Why this change is needed]
- [Any constraints or requirements]

Expected outcome:
- [What files should be modified]
- [What the end result should look like]
- [How to verify success]
`
```

## Best Practices

### 1. Clear Task Definition

**Good:**
```
Refactor user authentication:
- Move auth logic from components to src/services/auth.ts
- Update all components using old auth methods
- Add TypeScript types for all auth functions
- Update tests to reflect new structure
- Verify build passes after changes
```

**Bad:**
```
Fix the auth stuff
```

### 2. Provide Context

Always include:
- Which files/directories are involved
- Why the change is needed
- Any architectural constraints
- Expected behavior after changes

### 3. Verification Requirements

Specify how to verify success:
- Run specific tests
- Check type errors
- Verify build succeeds
- Test specific functionality

### 4. Incremental Approach

For very large tasks, break into phases:
```
Phase 1: Rename core functions and update direct usages
Phase 2: Update tests and type definitions
Phase 3: Update documentation and examples
```

## Skill Workflow (Main Conversation)

When handling directly in main conversation:

1. **Understand the task**
   - Clarify requirements with user if needed
   - Identify files that need modification

2. **Plan the changes**
   - List files to modify
   - Describe changes for each file
   - Identify potential issues

3. **Execute modifications**
   - Use Read to examine files first
   - Use Edit for precise modifications
   - Maintain consistency across changes

4. **Verify changes**
   - Check syntax and logic
   - Suggest running tests/build
   - Confirm expected behavior

## Integration with Other Skills

This skill works well with:

- **typescript-strict-typing**: Ensure type safety during refactoring
- **code-quality-standards**: Run quality checks after modifications
- **git-commit-conventions**: Create proper commits for changes

## Examples

### Example 1: Function Renaming (Skill Mode)

```
User: "Rename getUserData to fetchUserProfile across the project"

Assistant (Skill Mode - handles directly):
1. Use Grep to find all occurrences
2. Use Read to examine each file
3. Use Edit to rename in each location
4. Report completion with file list
```

### Example 2: Architecture Refactoring (Sub-Agent Mode)

```
User: "Refactor authentication to use a centralized auth service"

Assistant (Delegates to sub-agent):
"I'll delegate this to the code-editor-executor sub-agent since it involves:
- Creating new service files
- Refactoring multiple components
- Updating imports across the project
- Modifying tests

The sub-agent will work in isolation and report back when complete."

[Task tool with subagent_type="claude-code-editor:code-editor-executor"]
```

### Example 3: Library Migration (Sub-Agent Mode)

```
User: "Migrate from moment.js to date-fns throughout the codebase"

Assistant (Delegates to sub-agent):
[Spawns sub-agent with detailed migration plan]

Sub-agent executes:
1. Find all moment.js usages
2. Replace with date-fns equivalents
3. Update imports
4. Remove moment.js from dependencies
5. Run tests and fix issues
6. Report completion summary
```

## Tips for Effective Usage

1. **Be Specific**: Provide clear, actionable descriptions
2. **Provide Context**: Include relevant file paths and patterns
3. **Set Expectations**: Define what "success" looks like
4. **Start Small**: For large refactorings, consider phases
5. **Verify Results**: Always review changes and run tests

## Limitations

- Cannot make subjective design decisions without guidance
- Requires clear instructions for complex architectural changes
- May need user input when encountering ambiguous situations
- Should not be used for exploratory coding (use for defined tasks)

## Output Format

After modifications, expect:

**In Skill Mode:**
```
✓ Modified files:
  - src/utils/data.ts (renamed getUserData → fetchUserProfile)
  - src/components/User.tsx (updated function call)
  - src/tests/data.test.ts (updated test cases)

Suggested next steps:
- Run tests: npm test
- Check types: npm run typecheck
```

**From Sub-Agent:**
```
✅ Refactoring completed successfully

Changes Summary:
- Files modified: 12
- Files created: 3
- Files deleted: 1
- Lines changed: ~450

Key modifications:
1. Created src/services/auth.ts (centralized auth service)
2. Refactored 8 components to use new auth service
3. Updated all auth-related tests
4. Removed deprecated auth utilities

Verification:
- ✓ Build passes
- ✓ Type check passes
- ✓ All tests pass (125/125)

The authentication system has been successfully refactored to use a centralized service pattern.
```

## When Sub-Agent Should Ask for Help

The sub-agent will stop and ask for guidance when:
- Encountering design ambiguities
- Finding conflicting patterns in codebase
- Unable to determine correct approach
- Hitting unexpected errors it cannot resolve
- Needing architectural decisions

This ensures quality while maintaining autonomy for mechanical tasks.