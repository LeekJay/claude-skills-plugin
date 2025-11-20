---
name: code-quality-standards
description: Code quality check standards and workflow. Use when completing feature implementation, fixing bugs, or before creating commits/PRs. Includes execution order for formatting, linting, type checking, and testing, plus strict TypeScript type error fixing rules with exception scenarios.
allowed-tools: Bash
---

# Code Quality Standards

Ensure code quality and project maintainability through standardized check workflows.

## When to Execute

✅ **Must execute full checks**:
- After completing feature implementation or bug fixes
- When user explicitly requests code quality checks
- Before creating Git commits or PRs

❌ **Not needed**:
- Minor text edits or formatting adjustments
- Documentation-only changes
- Exploratory code reading

## Check Workflow (Execute in Order)

### 1. Code Formatting

```bash
pnpm format
```

Ensure all files comply with project code style standards.

### 2. Code Linting

```bash
pnpm lint
```

- Fix all errors and warnings
- Don't use `eslint-disable` to bypass issues

### 3. Type Checking (TypeScript)

```bash
pnpm typecheck
# If typecheck script doesn't exist, use:
tsc --noEmit
```

#### Type Error Fixing Rules (Only when fixing type errors)

##### ❌ Absolutely Prohibited

- Type assertions: `as`, `as any`, `as unknown as X`
- `any` type declarations
- `@ts-ignore` or `@ts-expect-error` comments

##### ✅ Mandatory Approach

1. **Analyze root cause** - Understand why the type error occurs
2. **Correct actual types** - Fix type definitions, declare types explicitly
3. **Improve type definitions** - Specify parameter and return types explicitly
4. **Add type guards** - Use type guards for TypeScript auto-inference
5. **Refactor code structure** - If type errors are hard to fix, may be a design issue

##### 📝 Rationale

- Type assertions and any hide real problems, leading to runtime errors
- Loses type safety and IDE intellisense
- Accumulates technical debt
- Misleads other developers

##### ⚠️ Exception Scenarios (Must consult user first)

1. **Third-party library with incomplete type definitions**
   - Library lacks proper type definitions
   - Consult user first, consider contributing type definitions or using alternative library

2. **Complex dynamic type scenarios**
   - Complex dynamic types where proper typing requires significant architectural changes
   - Consult user first, evaluate refactoring cost vs technical debt

3. **TypeScript type system limitations**
   - Legitimate edge cases that TypeScript's type system cannot properly express
   - Consult user first, confirm this is indeed a TypeScript limitation
   - Add detailed comments explaining why type assertion is needed

##### 🔴 Complex Type Errors

If type error is complex or difficult to understand:
1. **Stop immediately** - Don't continue trying to fix
2. **Consult user** - Seek guidance
3. **Don't** use type assertions or any to bypass

##### 📌 Important Note

These strict rules **only apply when fixing type errors**!

For general code writing:
- Can follow existing project patterns
- No need to refactor all existing type assertions
- Follow existing code style

### 4. Unit Tests (if applicable)

```bash
pnpm test
```

Ensure all tests pass.

## Failure Handling Workflow

1. **Fix issues immediately** - Analyze error cause, apply correct fix
2. **Re-run failed check** - Confirm issue is resolved
3. **Re-run all checks** - If fix introduces new code changes, restart from step 1
4. **Keep todo status** - Until all checks pass
5. **Don't commit failing code**

## Completion Confirmation

After all checks pass, report:

```
✅ All code quality checks passed

- ✓ Code formatting (pnpm format)
- ✓ Code linting (pnpm lint)
- ✓ Type checking (pnpm typecheck)
- ✓ Unit tests (pnpm test)

Code is ready to commit.
```

## Special Cases

### Check Commands Don't Exist

1. Check `package.json` `scripts` section
2. Look for alternative commands
3. If truly doesn't exist, notify user and suggest configuration

### Multiple Checks Fail Simultaneously

Fix in order: **format → lint → typecheck → test**

### Check Runs Over 5 Minutes

Notify user and ask whether to continue waiting or interrupt to investigate.

### Unable to Fix Check Errors

Stop fix attempts, consult user, provide error details and attempted fixes.

## Recommended Tools

### Git Hooks

```json
{
  "husky": {
    "hooks": {
      "pre-commit": "pnpm lint-staged",
      "pre-push": "pnpm typecheck && pnpm test"
    }
  }
}
```

### CI/CD Automated Checks

Run all checks automatically in CI/CD pipeline to ensure every commit and PR passes quality checks.
