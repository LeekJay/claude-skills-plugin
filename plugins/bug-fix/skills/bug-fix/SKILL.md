# Bug Fix Skill

## Overview

This skill provides intelligent bug fixing capabilities through a hybrid architecture:
- **Skill mode**: Direct handling in main conversation for simple, isolated bugs
- **Sub-agent mode**: Delegate complex, multi-file bugs to isolated executor context

## When to Use This Skill

### ✅ Use This Skill For:

**Bug Identification & Analysis:**
- Diagnosing runtime errors and exceptions
- Analyzing stack traces and error messages
- Identifying root causes of unexpected behavior
- Tracing bugs through call chains

**Simple Bug Fixes:**
- Single-file bug fixes
- Logic errors in specific functions
- Off-by-one errors and boundary conditions
- Type errors and null/undefined issues
- Simple state management bugs

**Complex Bug Fixes:**
- Bugs spanning multiple files
- Race conditions and concurrency issues
- Memory leaks and performance bugs
- Integration bugs across modules
- Data consistency issues

**Regression Fixes:**
- Bugs introduced by recent changes
- Breaking test cases
- API contract violations
- Behavior changes affecting downstream code

### ❌ Don't Use For:

- Feature requests (not bugs)
- Code refactoring (unless fixing a bug)
- Performance optimization (unless it's a performance bug)
- Code style improvements
- Adding new functionality

## Decision Framework

**Default behavior: For bug fixing tasks, prefer Sub-Agent mode unless the task is clearly simple.**

### 🔴 Mandatory Sub-Agent Triggers (ANY ONE triggers delegation)

1. **Multi-file scope**: Bug description mentions 2+ files or modules
2. **Unknown root cause**: Requires investigation ("sometimes...", "occasionally...", "intermittent...")
3. **Complex keywords**: Description contains race condition, memory leak, data inconsistency, state sync, cache issues, deadlock
4. **Cross-layer issues**: Involves frontend+backend, multiple services, database+application layer
5. **Debugging required**: Needs logging, breakpoints, execution flow tracing
6. **Regression/recurring**: Bug existed before, was fixed but reappeared, or has history
7. **Test-related**: Needs test writing for verification, involves test framework issues
8. **Unclear reproduction**: User cannot reliably reproduce the bug

### 🟢 Main Conversation Handling (ALL conditions must be met)

1. **Single-file fix**: Clearly only 1 file needs modification
2. **Clear cause**: User already knows the problem ("missing null check", "typo in variable name")
3. **Simple fix**: Adding a condition, fixing typo, adjusting parameter order
4. **No verification needed**: Can confirm fix is correct without running tests

### Decision Flow

```
Check for ANY 🔴 mandatory trigger?
  ├─ YES → ✅ USE SUB-AGENT MODE immediately
  └─ NO → Check if ALL 🟢 simple conditions are met?
           ├─ YES → Handle in main conversation
           └─ NO → ✅ USE SUB-AGENT MODE (default behavior)
```

### Quick Reference Examples

| User Description | Trigger Signal | Decision |
|-----------------|----------------|----------|
| "Login button doesn't respond" | 🔴 Unknown cause | Sub-Agent |
| "This map() call is missing key prop" | 🟢 All simple conditions met | Main conversation |
| "User data sometimes shows wrong values" | 🔴 Unknown cause + "sometimes" | Sub-Agent |
| "Remove this console.log" | 🟢 All conditions met | Main conversation |
| "Payment flow occasionally fails" | 🔴 Unknown cause + cross-layer | Sub-Agent |
| "TypeError in UserProfile component" | 🔴 Requires investigation | Sub-Agent |
| "Add missing semicolon on line 42" | 🟢 All conditions met | Main conversation |

## Usage Modes

### Mode 1: Skill Mode (Main Conversation)

For straightforward, isolated bugs:

```markdown
When user reports a bug that is:
- Clearly scoped to 1-2 files
- Has obvious root cause
- Unlikely to have cascading effects
- Can be fixed and verified quickly

Handle directly in main conversation without spawning sub-agent.
```

**Example scenarios:**
- "Fix the off-by-one error in the pagination function"
- "The login form doesn't validate email correctly"
- "TypeError in user profile component when name is missing"

### Mode 2: Sub-Agent Mode (Isolated Context)

For complex, multi-component bugs:

```markdown
When user reports a bug that is:
- Affects multiple files or modules
- Has unclear root cause requiring investigation
- May need multiple fix attempts
- Requires extensive testing and verification

Delegate to bug-fix-executor sub-agent using:
subagent_type="bug-fix:bug-fix-executor"
```

**Example scenarios:**
- "User data inconsistency between frontend and backend"
- "Race condition causing occasional duplicate records"
- "Memory leak in the real-time notification system"
- "Authentication state not persisting across page reloads"

## Bug Fixing Workflow

### Phase 1: Bug Identification

**Understand the Bug:**
1. **Read the bug report**
   - What is the expected behavior?
   - What is the actual behavior?
   - When does it occur?
   - How to reproduce?

2. **Gather context**
   - Error messages and stack traces
   - Browser console logs (for frontend)
   - Server logs (for backend)
   - Recent code changes (git log/diff)

3. **Identify affected areas**
   - Which files/modules are involved?
   - Which features are impacted?
   - Are there related bugs?

**Tools to use:**
```bash
# Search for error messages
Grep: "error message text"

# Find affected files
Glob: "**/*.{js,ts,py}"

# Read relevant code
Read: path/to/suspected/file.ts

# Check recent changes
Bash: git log --oneline -20
Bash: git diff HEAD~5
```

### Phase 2: Root Cause Analysis

**Trace the bug:**
1. **Follow the execution flow**
   - Start from the error point
   - Trace backwards to find the source
   - Check function calls and data flow

2. **Identify the root cause**
   - Is it a logic error?
   - Is it a missing null check?
   - Is it a race condition?
   - Is it incorrect data handling?

3. **Assess impact**
   - How critical is this bug?
   - What else might be affected?
   - Are there security implications?

**Decision Point:**
- **Simple, isolated bug** → Continue in skill mode
- **Complex, multi-file bug** → Delegate to sub-agent

### Phase 3: Fix Implementation

**In Skill Mode:**
1. **Plan the fix**
   - What code needs to change?
   - What are the minimal changes needed?
   - How to verify the fix works?

2. **Apply the fix**
   - Use Read to examine the file
   - Use Edit to make precise changes
   - Maintain code quality and style

3. **Add safety checks**
   - Add null checks if needed
   - Add input validation
   - Add error handling
   - Add defensive programming patterns

**Example fix:**
```typescript
// Before (buggy code)
function getUserName(user) {
  return user.profile.name;  // Crashes if profile is null
}

// After (fixed code)
function getUserName(user) {
  return user?.profile?.name ?? 'Anonymous';  // Safe with fallback
}
```

### Phase 4: Verification

**Test the fix:**
1. **Unit tests**
   - Add/update tests for the bug
   - Ensure tests fail before fix
   - Ensure tests pass after fix

2. **Integration tests**
   - Test the fix in context
   - Check for side effects
   - Verify related functionality

3. **Manual testing**
   - Reproduce the original bug
   - Verify it's fixed
   - Test edge cases

**Run verification:**
```bash
# Run tests
npm test
pytest

# Type checking
npm run typecheck
mypy .

# Linting
npm run lint
ruff check .

# Build
npm run build
```

### Phase 5: Documentation

**Document the fix:**
1. **Code comments** (if logic is complex)
2. **Commit message** (clear description of bug and fix)
3. **Update tests** (ensure bug is covered)
4. **Update docs** (if behavior changed)

## Sub-Agent Delegation

When delegating to sub-agent, use this format:

```typescript
// Use Task tool with proper subagent_type
subagent_type: "bug-fix:bug-fix-executor"

// Provide clear bug description
prompt: `
Bug Report:
[Describe the bug - what's wrong, when it happens, how to reproduce]

Error Messages:
[Stack traces, console errors, log messages]

Context:
- Affected files/modules: [list]
- Recent changes: [relevant commits or changes]
- Environment: [browser, OS, runtime version]

Expected Behavior:
[What should happen]

Actual Behavior:
[What actually happens]

Reproduction Steps:
1. [Step 1]
2. [Step 2]
3. [Step 3]

Additional Context:
[Any other relevant information]
`
```

## Best Practices

### 1. Understand Before Fixing

**Good:**
```
1. Read error message carefully
2. Trace through the code to find root cause
3. Understand why the bug happens
4. Plan minimal fix that addresses root cause
5. Apply fix with tests
```

**Bad:**
```
1. See error
2. Make random changes hoping it works
3. Don't understand why it fixed it
```

### 2. Minimal, Targeted Fixes

**Good:**
```typescript
// Minimal fix - add null check
if (user?.profile) {
  displayProfile(user.profile);
}
```

**Bad:**
```typescript
// Over-engineered fix - unnecessary refactoring
class ProfileManager {
  constructor(private validator: ProfileValidator) {}
  displayIfValid(user: User) {
    if (this.validator.validate(user?.profile)) {
      this.display(user.profile);
    }
  }
}
```

### 3. Add Tests for Bugs

**Always:**
- Write a test that reproduces the bug
- Ensure test fails before fix
- Ensure test passes after fix
- Add edge case tests

**Example:**
```typescript
describe('getUserName', () => {
  it('should handle missing profile gracefully', () => {
    const user = { id: 1, profile: null };
    expect(getUserName(user)).toBe('Anonymous');
  });

  it('should handle missing name gracefully', () => {
    const user = { id: 1, profile: {} };
    expect(getUserName(user)).toBe('Anonymous');
  });
});
```

### 4. Consider Side Effects

Before applying a fix, consider:
- Will this break other code?
- Are there other places with the same bug?
- Does this change the API contract?
- Will tests need updating?

### 5. Document Complex Fixes

For non-obvious fixes, add comments:
```typescript
// Fix: Check if profile exists before accessing name
// Bug: Crashed when users had no profile set up yet
// Happens for newly registered users before completing profile
if (user?.profile?.name) {
  return user.profile.name;
}
return 'Anonymous';
```

## Common Bug Patterns

### Pattern 1: Null/Undefined Errors

**Symptoms:**
```
TypeError: Cannot read property 'x' of null
TypeError: Cannot read property 'y' of undefined
```

**Fix Strategy:**
```typescript
// Add optional chaining and nullish coalescing
const value = obj?.prop?.nested ?? defaultValue;

// Add null checks
if (obj && obj.prop) {
  // safe to use obj.prop
}
```

### Pattern 2: Off-by-One Errors

**Symptoms:**
```
IndexError: list index out of range
Array index out of bounds
```

**Fix Strategy:**
```typescript
// Check array bounds
for (let i = 0; i < array.length; i++) {  // Use <, not <=
  // process array[i]
}

// Or use safer iteration
for (const item of array) {
  // process item
}
```

### Pattern 3: Async/Race Conditions

**Symptoms:**
```
Unpredictable behavior
Occasional failures
Data inconsistency
```

**Fix Strategy:**
```typescript
// Use proper async/await
async function fetchData() {
  try {
    const result = await api.getData();
    return result;
  } catch (error) {
    console.error('Failed to fetch:', error);
    throw error;
  }
}

// Prevent race conditions
let latestRequestId = 0;
async function search(query) {
  const requestId = ++latestRequestId;
  const results = await api.search(query);

  // Only use results if this is still the latest request
  if (requestId === latestRequestId) {
    displayResults(results);
  }
}
```

### Pattern 4: State Management Bugs

**Symptoms:**
```
UI not updating
Stale data displayed
State out of sync
```

**Fix Strategy:**
```typescript
// React: Use proper state updates
// Bad
state.items.push(newItem);  // Mutating state directly

// Good
setState({ items: [...state.items, newItem] });  // Immutable update

// Vue: Use reactive updates
// Bad
this.items.push(newItem);  // May not trigger reactivity

// Good
this.items = [...this.items, newItem];  // Triggers reactivity
```

### Pattern 5: Error Handling Bugs

**Symptoms:**
```
Unhandled promise rejection
Uncaught exceptions
Silent failures
```

**Fix Strategy:**
```typescript
// Add proper error handling
try {
  const result = await riskyOperation();
  return result;
} catch (error) {
  console.error('Operation failed:', error);
  // Handle error appropriately
  // - Show user-friendly message
  // - Log for debugging
  // - Return fallback value
  // - Re-throw if can't handle
  return fallbackValue;
}

// Handle promise rejections
promise
  .then(handleSuccess)
  .catch(handleError);

// Add global error handlers
window.addEventListener('unhandledrejection', (event) => {
  console.error('Unhandled rejection:', event.reason);
});
```

### Pattern 6: Type Errors (TypeScript)

**Symptoms:**
```
Type 'X' is not assignable to type 'Y'
Property 'z' does not exist on type 'A'
```

**Fix Strategy:**
```typescript
// Fix type definitions
interface User {
  id: number;
  name: string;
  email?: string;  // Optional property
}

// Use type guards
function isUser(obj: any): obj is User {
  return obj && typeof obj.id === 'number' && typeof obj.name === 'string';
}

// Use proper type assertions (sparingly)
const user = data as User;  // Only when you're certain
```

## Integration with Other Skills

This skill works well with:

- **code-quality-standards**: Run quality checks after bug fixes
- **typescript-strict-typing**: Ensure type safety in fixes
- **git-commit-conventions**: Create proper commits for bug fixes
- **claude-code-editor**: For large-scale bug fixes affecting many files

## Examples

### Example 1: Simple Null Check Bug (Skill Mode)

```
User: "The app crashes when clicking on a user without a profile picture"

Assistant (Skill Mode):
1. Use Grep to find profile picture rendering code
2. Use Read to examine the component
3. Identify missing null check
4. Use Edit to add optional chaining
5. Suggest adding test case
6. Report completion
```

**Fix applied:**
```typescript
// Before
<img src={user.profile.avatar} alt="Avatar" />

// After
<img src={user.profile?.avatar ?? '/default-avatar.png'} alt="Avatar" />
```

### Example 2: Race Condition Bug (Sub-Agent Mode)

```
User: "Search results sometimes show data from previous searches"

Assistant (Delegates to sub-agent):
"This is a race condition bug that likely affects multiple components:
- Search input handler
- API request management
- Results rendering
- State management

I'll delegate to the bug-fix-executor sub-agent for systematic investigation and fix."

[Task tool with subagent_type="bug-fix:bug-fix-executor"]
```

### Example 3: Data Inconsistency Bug (Sub-Agent Mode)

```
User: "User data shows different values on profile page vs. settings page"

Assistant (Delegates to sub-agent):
"This data inconsistency bug requires investigating:
- Data fetching in both components
- Cache management
- State synchronization
- API responses

The sub-agent will trace the data flow and fix the inconsistency."

[Spawns sub-agent with detailed bug report]
```

## Output Format

After bug fixes, expect:

**In Skill Mode:**
```
✓ Bug fixed in: src/components/UserProfile.tsx

Issue: Null pointer exception when user.profile is null
Root cause: Missing null check before accessing profile.avatar
Fix applied: Added optional chaining (?.) and default fallback

Changes:
- Line 45: Added null safety check
- Line 46: Added default avatar fallback

Suggested verification:
- Test with users without profile pictures
- Run tests: npm test
- Check types: npm run typecheck

Would you like me to add a test case for this bug?
```

**From Sub-Agent:**
```
✅ Bug fixed successfully

Bug: Race condition in search results display
Root Cause: No request cancellation for outdated search queries

Fix Summary:
- Files modified: 3
- Key changes:
  1. Added request ID tracking in SearchContext (src/context/SearchContext.tsx)
  2. Implemented request cancellation in useSearch hook (src/hooks/useSearch.ts)
  3. Updated SearchResults to ignore stale data (src/components/SearchResults.tsx)

Verification:
- ✓ Type check passes
- ✓ All tests pass (45/45)
- ✓ Manual testing: Rapid search queries no longer show stale results
- ✓ No performance degradation

Technical Details:
- Used incremental request ID to track latest search
- Results only displayed if request ID matches latest
- Canceled pending requests when new search initiated

The race condition has been eliminated. Search results now consistently
match the current search query, even with rapid typing.
```

## When Sub-Agent Should Ask for Help

The sub-agent will stop and ask for guidance when:
- Root cause is unclear after investigation
- Multiple potential fixes exist with trade-offs
- Fix requires architectural changes
- Fix might have significant side effects
- Business logic decisions needed (e.g., default behavior)

## Bug Priority Levels

**P0 - Critical:**
- App crashes or is unusable
- Data loss or corruption
- Security vulnerabilities
- Fix immediately in skill mode or sub-agent

**P1 - High:**
- Major feature broken
- Affects many users
- Workaround exists but not ideal
- Fix soon, use appropriate mode

**P2 - Medium:**
- Minor feature broken
- Affects some users
- Good workaround exists
- Schedule fix appropriately

**P3 - Low:**
- Cosmetic issues
- Edge cases
- Nice to have fixes
- Fix when convenient

## Limitations

- Cannot fix bugs without reproduction steps
- Cannot fix bugs in external dependencies (can only work around)
- Cannot fix bugs requiring architectural changes without guidance
- Cannot determine business logic without user input
- Cannot fix bugs without access to error messages/logs

## Tips for Effective Bug Reporting

When reporting bugs to this skill, provide:

1. **Clear description** of expected vs actual behavior
2. **Reproduction steps** (step-by-step)
3. **Error messages** (full stack traces)
4. **Environment info** (browser, OS, runtime version)
5. **Recent changes** (if known)
6. **Impact assessment** (who's affected, how critical)

**Good bug report:**
```
Bug: Login fails with "Invalid token" error after successful authentication

Reproduction:
1. Go to /login
2. Enter valid credentials
3. Click "Sign In"
4. Error appears: "Invalid token"

Error message:
  JWTError: Token signature verification failed
  at verifyToken (auth.js:45)
  at validateSession (middleware.js:23)

Environment: Chrome 120, Windows 11, Node 18.17.0

Recent changes: Upgraded jsonwebtoken from 8.5.1 to 9.0.0

Impact: All users cannot log in (P0 - Critical)
```

This gives the skill everything needed to quickly identify and fix the bug!
