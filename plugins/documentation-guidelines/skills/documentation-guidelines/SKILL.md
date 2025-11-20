---
name: documentation-guidelines
description: Documentation creation and update standards. Use when considering creating documentation or when code changes involve documentation. Don't proactively create docs (unless critical task or explicitly requested), update relevant docs only after code changes pass all checks.
---

# Documentation Guidelines

Standardize documentation creation and update workflow to avoid unnecessary documentation generation.

## Core Principles

### 1. Passive Documentation Creation

**Core**: Don't proactively create documentation unless there's clear need

**Reasons**:
- Avoid creating unmaintained documentation
- Documentation easily becomes outdated and out of sync
- Code itself should be self-documenting
- Time better spent on code quality

### 2. Code First

**Priority**:
1. Write clear, readable code
2. Add necessary code comments
3. Write type definitions and interface documentation
4. Consider external documentation last

## When NOT to Create Documentation

❌ Don't create documentation for:

1. **Non-critical tasks** - Temporary code adjustments, small bug fixes
2. **Exploratory work** - Trying different approaches, prototyping
3. **User didn't explicitly request** - After completing features
4. **Code is already clear** - Clear naming, complete type definitions
5. **Temporary code** - Debug helper code, temporary scripts

## When to Create Documentation

✅ Create documentation for:

1. **User explicitly requests** - "Help me write API usage documentation"
2. **Critical tasks or important features** - Core modules, complex architecture, critical configs, important APIs
3. **Complex setup processes** - Deployment workflows, environment config, third-party service integration
4. **For other developers** - Open source projects, internal shared libraries, CLI tools
5. **Existing documentation needs update** - Code changes affect existing documentation

## Documentation Update Workflow

**Strictly follow this order**:

```
1. Complete code implementation
   ↓
2. Run code quality checks
   ├─ pnpm format
   ├─ pnpm lint
   ├─ pnpm typecheck
   └─ pnpm test
   ↓
3. All checks pass
   ↓
4. Update relevant documentation
   ↓
5. Commit code and documentation
```

**Why this order?**
- Avoid duplicate work - code may change during checks
- Ensure accuracy - documentation based on final, verified code
- Improve efficiency - focus on one task at a time
- Reduce omissions - documentation update not forgotten after code passes checks

## Documentation in Code

### Recommended Forms

#### 1. Type Definitions (TypeScript)

```typescript
/**
 * User registration data
 */
interface UserRegistrationData {
  /** User email address */
  email: string;
  /** Password (minimum 8 characters) */
  password: string;
}
```

#### 2. JSDoc/TSDoc Comments

```typescript
/**
 * Validate user registration data
 *
 * @param data - User registration data
 * @returns Validation result with validity and error info
 *
 * @example
 * ```ts
 * const result = validateUserRegistration({
 *   email: 'user@example.com',
 *   password: 'securepass123'
 * });
 * ```
 */
function validateUserRegistration(data: UserRegistrationData): ValidationResult {
  // ...
}
```

#### 3. Inline Comments (Only for Complex Logic)

```typescript
// ✅ Good comment: explains why
// Use bcrypt instead of md5 because md5 is proven insecure
const hashedPassword = await bcrypt.hash(password, 10);

// ❌ Unnecessary comment
// Set username to username
user.name = username;
```

## Documentation Types

### 1. README.md

**When to create**: New project initialization (if user requests), open source projects

**Must include**:
- Project introduction
- Installation instructions
- Basic usage examples
- Development guide links

### 2. API Documentation

**When to create**: Providing public API, user explicitly requests, API is complex and non-intuitive

**Recommended formats**:
- OpenAPI/Swagger spec
- JSDoc/TSDoc comments
- Dedicated API.md file

### 3. CHANGELOG.md

**When to create**: When releasing versions, users need to know change history

**Recommended format**: Follow [Keep a Changelog](https://keepachangelog.com/)

## Documentation Maintenance

### Keep Documentation in Sync

**Strategies**:
1. **Check related docs when code changes**
   - Modify API → check API docs
   - Change config → check config docs

2. **Require documentation updates in PRs**
   - PR checklist includes "documentation updated"
   - Check documentation during code review

3. **Periodically review documentation**
   - Quarterly or semi-annual review
   - Delete outdated documentation
   - Update stale information

## Examples

### After New Feature Complete

```
Scenario: Implemented user authentication feature

✅ Correct workflow:
1. Complete code implementation
2. Add tests
3. Run all quality checks and pass
4. Check if existing documentation needs update
5. Create documentation if user requested; otherwise only update existing docs
6. Commit code
```

### API Endpoint Changes

```
Scenario: Modified /api/users endpoint response format

✅ Correct workflow:
1. Modify code
2. Update tests
3. Run quality checks → pass
4. Check if docs/API.md exists
   ├─ Exists → update endpoint documentation
   └─ Doesn't exist → ask user if creation needed
5. Record changes in CHANGELOG.md (if exists)
6. Commit code and documentation updates
```

## Recommended Tools

- **JSDoc/TSDoc**: In-code documentation
- **Docusaurus**: Documentation website (React-based)
- **VitePress**: Documentation website (Vue-based)
- **Swagger/OpenAPI**: API documentation
- **Storybook**: UI component documentation

## Summary

| Principle | Description |
|-----------|-------------|
| **Passive creation** | Don't proactively create docs |
| **Code first** | Prioritize writing clear code |
| **Delayed update** | Update docs after code passes checks |
| **Keep in sync** | Check related docs when code changes |
| **Timely cleanup** | Delete outdated and irrelevant docs |
