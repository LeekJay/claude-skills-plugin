---
name: pull-request-guidelines
description: Standardize Pull Request workflow and content requirements. Use when creating PRs, preparing for code review, or validating PR content. Ensure PRs include Chinese summary, related issue links, verification steps, and impact assessment.
---

# Pull Request Guidelines

Standardize Pull Request workflow to ensure code quality, traceability, and efficient code review.

## PR Title

Follow Conventional Commits format:

```
<type>(<scope>): <Chinese subject>
```

Examples:
- ✅ `feat(auth): 添加 OAuth 2.0 登录支持`
- ✅ `fix(api): 修复用户列表分页问题`

## PR Description Must Include

### 1. Chinese Summary

Clearly describe in Chinese:
- What does this PR do?
- Why is this change needed?
- What problem does it solve?

### 2. Related Issue Links

```markdown
Closes #123
Related to #456
```

### 3. Verification Steps

#### Automated Checks
- [ ] `pnpm lint` passes
- [ ] `pnpm format` passes
- [ ] `pnpm typecheck` passes
- [ ] `pnpm test` passes

#### Manual Testing
1. Specific test steps
2. Expected results
3. Test environment info

### 4. Impact Assessment

Include:
- Database migration requirements
- Configuration changes
- API changes
- Backward compatibility

## Pre-Review Checklist

- [ ] Local lint passes
- [ ] Code formatting complete
- [ ] Type checking passes
- [ ] Tests pass
- [ ] No type assertions (unless justified and documented)
- [ ] Commit messages follow conventions
- [ ] No AI identifiers
- [ ] Manual testing complete
- [ ] Documentation updated (if necessary)

## PR Template Example

```markdown
## Summary

[Chinese description of main changes and purpose]

## Related Issues

Closes #[issue number]

## Main Changes

- [Change 1]
- [Change 2]

## Verification Steps

### Automated Checks
- [ ] `pnpm lint` passes
- [ ] `pnpm format` passes
- [ ] `pnpm typecheck` passes
- [ ] `pnpm test` passes

### Manual Testing
1. [Test step 1]
2. [Test step 2]

## Impact Assessment

### Database Migration
- [ ] Migration needed / [x] No migration

### Configuration Changes
- [ ] New env variables / [x] No config changes

### API Changes
- [ ] Breaking changes / [x] Backward compatible
```

## Code Review Guidelines

Reviewer focus areas:
1. **Functional correctness**: Does the code implement expected functionality?
2. **Code quality**: Does it follow project standards and best practices?
3. **Test coverage**: Are there adequate tests?
4. **Performance**: Are there performance issues?
5. **Security**: Are there security vulnerabilities?
6. **Maintainability**: Is the code easy to understand and maintain?

## Rationale

**Why require detailed PR descriptions?**
- Helps reviewers quickly understand change context
- Provides traceable change documentation
- Reduces review time and communication cost

**Why require Chinese summary?**
- Team primarily communicates in Chinese
- Chinese descriptions are more accurate and natural

**Why require verification steps?**
- Ensures changes are thoroughly tested
- Provides guidance for reviewer verification
- Prevents regression issues
