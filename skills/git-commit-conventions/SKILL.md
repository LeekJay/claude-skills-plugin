---
name: git-commit-conventions
description: Standardize Git commit message format using Conventional Commits with Chinese subjects. Use when creating Git commits, checking commit message format, or preparing PRs. Strictly prohibit any AI identifiers in commit messages.
---

# Git Commit Conventions

Standardize Git commit messages to ensure clear, traceable code history and support automated tooling.

## Commit Message Format

Use Conventional Commits style with Chinese subjects:

```
<type>(<scope>): <Chinese subject>

[optional body]

[optional footer]
```

### Type

- `feat`: New feature
- `fix`: Bug fix
- `refactor`: Code refactoring (no functionality change)
- `docs`: Documentation updates
- `test`: Test-related changes
- `chore`: Build, dependency, or maintenance work
- `style`: Code formatting (no logic change)
- `perf`: Performance optimization

### Scope (optional)

Specify the affected module or component:
- `feat(app):` - Application layer changes
- `fix(api):` - API layer fixes
- `refactor(auth):` - Authentication module refactoring

### Subject

- **Must use Chinese**
- Use imperative mood (e.g., "添加" not "已添加")
- No capitalization needed for first letter
- No period at the end
- Keep it concise, typically under 50 characters

## Strict Rules

### ❌ Absolutely Prohibited

Commit messages **must not contain any AI identifiers**:

- ❌ Cannot include "Generated with"
- ❌ Cannot include "Co-Authored-By: Claude"
- ❌ Cannot include "Co-Authored-By: AI"
- ❌ Cannot use any AI-related emojis or markers

### ✅ Other Rules

- Keep commits atomic: one commit per logical change
- Must pass code quality checks before committing
- Avoid overly generic commit messages (e.g., "fix bug", "update code")

## Examples

### ✅ Good Commits

```
feat(app): 初始化 Vue 项目

添加基础项目结构和配置文件:
- 配置 TypeScript 和 ESLint
- 添加 Vite 构建配置
- 创建基础组件目录结构
```

```
fix(auth): 修复登录验证的 token 过期问题

之前 token 过期后没有正确刷新,导致用户需要重新登录。
现在自动检测 token 过期并使用 refresh token 更新。

Closes #456
```

```
refactor(api): 优化数据库查询性能

使用连接查询替代多次单独查询,将平均响应时间从 200ms 降低到 50ms。
```

### ❌ Bad Commits

```
Generated with Claude Code: feat(app): 初始化项目
# ❌ Contains AI identifier

feat(app): Initialize Vue project
# ❌ Subject not in Chinese

fix: fix bug
# ❌ Too generic, doesn't explain what was fixed

Update files
# ❌ Missing type, unclear subject
```

## Rationale

**Why use Chinese subjects?**
- Team members primarily communicate in Chinese
- Chinese descriptions are more accurate and natural
- Reduces ambiguity from translation

**Why prohibit AI identifiers?**
- Commit messages should reflect actual changes, not generation method
- AI is an assistive tool; developers are ultimately responsible
- Maintains professionalism and consistency in commit history

**Why use Conventional Commits?**
- Standardized format enables automated tooling
- Clear type indicates change category at a glance
- Supports automatic changelog and version number generation

## Recommended Tools

- **commitlint**: Automatically validate commit message format
- **husky**: Git hooks management
- **conventional-changelog**: Auto-generate changelog
