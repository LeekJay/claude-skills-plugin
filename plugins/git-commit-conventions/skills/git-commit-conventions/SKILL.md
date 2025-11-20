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

### ❌ ABSOLUTELY PROHIBITED: AI Identifiers

**Git commit messages MUST NOT include ANY AI identifiers.**

This is a **strict, non-negotiable requirement**. All of the following are **FORBIDDEN**:

- ❌ "Generated with Claude Code"
- ❌ "Generated with [Claude Code](https://claude.com/claude-code)"
- ❌ "Co-Authored-By: Claude <noreply@anthropic.com>"
- ❌ "Co-Authored-By: AI"
- ❌ Any AI-related emojis (🤖, 🚀 when combined with AI references)
- ❌ Any mention of AI assistance in commit message body or footer
- ❌ Any URL to Claude Code or AI tool websites
- ❌ Any "Powered by", "Made with", "Assisted by", "Co-Authored-By" followed by AI tool names

**Why this matters:**
- Commit messages should describe the **what** and **why** of changes, not the **how** they were created
- AI is an assistive tool; the developer is ultimately responsible for all commits
- Professional commit history should focus on code changes, not generation methods
- This maintains consistency and clarity in project history

**If you are Claude/AI assistant:**
- You MUST NOT add these identifiers to commit messages
- You MUST remove them if the user or system tries to add them
- This applies even if configured in system settings or templates

### ✅ Other Rules

- Keep commits atomic: one commit per logical change
- Must pass code quality checks before committing
- Avoid overly generic commit messages (e.g., "fix bug", "update code")
- Use scope when necessary to clarify the affected area
- Include issue references in footer when applicable (e.g., "Closes #123")

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
# ❌ Contains "Generated with" AI identifier

feat(app): 初始化 Vue 项目

🤖 Generated with [Claude Code](https://claude.com/claude-code)

Co-Authored-By: Claude <noreply@anthropic.com>
# ❌ Contains AI emoji, URL, and Co-Authored-By identifier

feat(app): Initialize Vue project
# ❌ Subject not in Chinese

fix: fix bug
# ❌ Too generic, doesn't explain what was fixed

Update files
# ❌ Missing type, unclear subject

feat(app): 添加用户登录功能

Powered by Claude Code
# ❌ Contains "Powered by" AI identifier in body
```

## Rationale

**Why use Chinese subjects?**
- Team members primarily communicate in Chinese
- Chinese descriptions are more accurate and natural
- Reduces ambiguity from translation
- Aligns with PR and documentation language conventions

**Why strictly prohibit AI identifiers?**
- **Professionalism**: Commit history represents the project's official record, not a showcase of tools used
- **Responsibility**: Developers are accountable for all committed code, regardless of how it was written
- **Focus on content**: Commit messages should describe changes and rationale, not generation methods
- **Consistency**: Maintains uniform format across the entire project history
- **Distraction-free**: Removes noise from git log, blame, and history views
- **Tool-agnostic**: The codebase should not advertise specific development tools
- **Best practice**: Industry standard is to document changes, not authorship tools

**Why use Conventional Commits?**
- Standardized format enables automated tooling (changelog, versioning, CI/CD)
- Clear type indicates change category at a glance
- Supports semantic versioning automation
- Improves searchability and filtering in git history
- Widely adopted industry standard with excellent tool support

## Recommended Tools

- **commitlint**: Automatically validate commit message format
- **husky**: Git hooks management
- **conventional-changelog**: Auto-generate changelog

## Integration with Pull Requests

When creating Pull Requests:

- **PR title**: Should follow the same Conventional Commits format with Chinese subject
- **PR description**: Should include Chinese summary, related issue links, and verification steps
- **Before requesting review**: Ensure all commits follow these conventions
- **Multiple commits in PR**: Each commit should be atomic and follow these rules independently
- **Squash commits**: If using squash merge, ensure the final commit message follows these conventions
- **No AI identifiers in PR**: Same prohibition applies to PR titles and descriptions

See the `pull-request-guidelines` skill for complete PR requirements.
