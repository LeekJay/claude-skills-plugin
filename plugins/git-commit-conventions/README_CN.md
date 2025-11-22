# Git Commit Conventions 插件

## 插件作用

这是一个 Git 提交规范插件,帮助你遵循 Conventional Commits 标准,使用中文主题编写规范化的提交信息。确保项目的 Git 历史清晰、可追溯,并支持自动化工具(如自动生成 changelog、语义化版本控制)。

**核心价值:**
- 📝 标准化提交格式 - 遵循 Conventional Commits 规范
- 🇨🇳 中文主题 - 更自然准确的描述
- 🚫 严禁 AI 标识 - 专业的提交历史,不展示生成工具
- 🔍 可追溯性 - 支持自动化工具和历史搜索

---

## 运作方式

这是一个 **Skill (技能)**,Claude 会在创建 Git commit 时自动激活它。

```
用户请求创建 commit
    ↓
Skill 自动激活
    ↓
Claude 遵循 Conventional Commits 格式
使用中文编写 subject
严格禁止添加 AI 标识
    ↓
生成规范化的提交信息
```

---

## Commit 消息格式

### 标准格式

```
<type>(<scope>): <Chinese subject>

[optional body]

[optional footer]
```

### Type (类型)

- `feat`: 新功能
- `fix`: Bug 修复
- `refactor`: 代码重构 (不改变功能)
- `docs`: 文档更新
- `test`: 测试相关changes
- `chore`: 构建、依赖或维护工作
- `style`: 代码格式化 (不改变逻辑)
- `perf`: 性能优化

### Scope (范围) - 可选

指定影响的模块或组件:
- `feat(app):` - 应用层变更
- `fix(api):` - API 层修复
- `refactor(auth):` - 认证模块重构

### Subject (主题)

- **必须使用中文**
- 使用祈使语气 (例如"添加"而不是"已添加")
- 首字母无需大写
- 末尾不加句号
- 保持简洁,通常在 50 字符以内

---

## 严格规则

### ❌ 绝对禁止: AI 标识

**Git 提交消息中绝对不允许包含任何 AI 标识。**

这是**严格的、不可协商的要求**。以下内容**全部禁止**:

```
❌ "Generated with Claude Code"
❌ "Generated with [Claude Code](https://claude.com/claude-code)"
❌ "Co-Authored-By: Claude <noreply@anthropic.com>"
❌ "Co-Authored-By: AI"
❌ 任何 AI 相关的 emoji (🤖, 🚀 结合 AI 引用时)
❌ 在 commit body 或 footer 中提及 AI 辅助
❌ 任何指向 Claude Code 或 AI 工具网站的 URL
❌ 任何 "Powered by", "Made with", "Assisted by", "Co-Authored-By" 后跟 AI 工具名称
```

### 为什么这很重要?

- Commit 消息应该描述**内容**(what)和**原因**(why),而不是**方式**(how)
- AI 是辅助工具;开发者对所有提交负最终责任
- 专业的提交历史应聚焦代码变更,不是生成方法
- 保持一致性和清晰度

### 其他规则

- 保持提交原子化: 一个提交一个逻辑变更
- 提交前必须通过代码质量检查
- 避免过于笼统的提交消息 (例如"fix bug","update code")
- 必要时使用 scope 明确影响范围
- 在 footer 中引用相关 issue (例如"Closes #123")

---

## 使用例子

### ✅ 好的提交

**例子 1: 新功能**
```
feat(app): 初始化 Vue 项目

添加基础项目结构和配置文件:
- 配置 TypeScript 和 ESLint
- 添加 Vite 构建配置
- 创建基础组件目录结构
```

**例子 2: Bug 修复**
```
fix(auth): 修复登录验证的 token 过期问题

之前 token 过期后没有正确刷新,导致用户需要重新登录。
现在自动检测 token 过期并使用 refresh token 更新。

Closes #456
```

**例子 3: 重构**
```
refactor(api): 优化数据库查询性能

使用连接查询替代多次单独查询,将平均响应时间从 200ms 降低到 50ms。
```

**例子 4: 文档**
```
docs: 更新 API 文档

添加新增端点的使用示例和参数说明。
```

---

### ❌ 不好的提交

**例子 1: 包含 AI 标识**
```
Generated with Claude Code: feat(app): 初始化项目
# ❌ 包含 "Generated with" AI 标识

feat(app): 初始化 Vue 项目

🤖 Generated with [Claude Code](https://claude.com/claude-code)

Co-Authored-By: Claude <noreply@anthropic.com>
# ❌ 包含 AI emoji、URL 和 Co-Authored-By 标识
```

**例子 2: 使用英文主题**
```
feat(app): Initialize Vue project
# ❌ Subject 没有使用中文
```

**例子 3: 过于笼统**
```
fix: fix bug
# ❌ 太笼统,没有说明修复了什么

Update files
# ❌ 缺少 type,subject 不清晰
```

**例子 4: Body 中包含 AI 标识**
```
feat(app): 添加用户登录功能

Powered by Claude Code
# ❌ 在 body 中包含 "Powered by" AI 标识
```

---

## 工作方式

### 自动激活

当你请求 Claude 创建 Git commit 时,这个 Skill 会自动激活:

```
用户: "帮我创建一个 commit"
    ↓
Skill 激活
    ↓
Claude:
1. 查看 git status 了解变更
2. 查看 git diff 了解具体改动
3. 查看 git log 学习项目的提交风格
4. 根据 Conventional Commits 格式编写提交消息
5. 使用中文编写 subject
6. 绝对不添加任何 AI 标识
7. 创建 commit
```

### 提交流程

Claude 会遵循以下流程创建 commit:

1. **运行 git status** - 查看未跟踪文件
2. **运行 git diff** - 查看暂存和未暂存的变更
3. **运行 git log** - 查看最近的提交消息,了解项目风格
4. **分析变更** - 理解变更的性质和目的
5. **编写提交消息**:
   - 总结变更性质 (新功能、增强、Bug 修复、重构等)
   - 使用中文编写简洁的描述 (1-2 句话)
   - 聚焦"为什么"而不是"什么"
6. **添加文件到暂存区** - 添加相关的未跟踪文件
7. **创建 commit** - 使用规范化的消息
8. **运行 git status** - 验证提交成功

---

## 格式示例

### 简单提交

```
feat(ui): 添加暗色模式切换按钮
```

### 带 Body 的提交

```
fix(auth): 修复 token 刷新竞态条件

当多个请求同时触发 token 刷新时,会导致重复刷新和认证失败。
现在使用互斥锁确保同一时间只有一个刷新请求。
```

### 带 Footer 的提交

```
feat(api): 实现用户权限管理

添加基于角色的访问控制 (RBAC):
- 定义角色和权限模型
- 实现权限检查中间件
- 添加权限管理 API 端点

Closes #123
Closes #456
```

### Breaking Change

```
refactor(api)!: 重构用户认证 API

BREAKING CHANGE: 认证端点从 /auth/login 改为 /api/v2/auth/login

迁移指南:
- 更新客户端 API 基础 URL
- 使用新的响应格式
```

---

## 最佳实践

### 原子化提交

每个 commit 应该是一个独立的逻辑变更:

```
✅ 好
commit 1: feat(auth): 添加用户登录功能
commit 2: feat(auth): 添加密码重置功能
commit 3: test(auth): 添加认证相关测试

❌ 不好
commit 1: feat(auth): 添加登录、密码重置和测试
```

### 描述性主题

```
✅ 好
fix(api): 修复用户列表分页计算错误

❌ 不好
fix: 修复 bug
```

### 适当使用 Scope

```
✅ 好
feat(ui): 添加加载指示器
feat(api): 实现数据导出功能

❌ 不好 (scope 太笼统)
feat(app): 添加功能
```

---

## 整合到工作流

### Pre-commit Hook

推荐使用 commitlint 自动验证提交消息格式:

```bash
# 安装
npm install --save-dev @commitlint/cli @commitlint/config-conventional

# 配置 commitlint.config.js
module.exports = {
  extends: ['@commitlint/config-conventional'],
  rules: {
    'subject-case': [0],  // 允许中文
  }
}

# 配合 husky
npx husky add .husky/commit-msg 'npx --no -- commitlint --edit ${1}'
```

### PR 创建

创建 Pull Request 时:
- PR 标题应遵循相同的 Conventional Commits 格式
- PR 描述应包含中文总结
- 不要在 PR 中包含 AI 标识
- 参考 `pull-request-guidelines` 插件

---

## 支持的自动化工具

遵循 Conventional Commits 格式可以使用:

- **conventional-changelog** - 自动生成 changelog
- **semantic-release** - 自动语义化版本控制
- **commitlint** - 验证提交消息格式
- **standard-version** - 自动版本管理

---

## 常见问题

**Q: 为什么主题必须用中文?**
A: 团队主要使用中文沟通,中文描述更准确自然,减少翻译歧义,与 PR 和文档语言保持一致。

**Q: 为什么严禁 AI 标识?**
A:
- 专业性: Commit 历史是项目的正式记录,不是工具展示
- 责任: 开发者对所有提交的代码负责,无论如何编写
- 聚焦内容: 提交消息应描述变更和理由,不是生成方法
- 一致性: 保持整个项目历史格式统一
- 无干扰: 避免 git log、blame、history 视图中的噪音

**Q: 技术术语是否需要翻译?**
A: 不需要。保持技术术语原文(如 API、token、OAuth),参考 `language-preferences` 插件。

**Q: 如何处理多个文件的变更?**
A: 如果变更是同一个逻辑功能,可以在一个 commit 中。如果是不同的功能,应该拆分为多个 commit。

---

## 相关插件

- **git-operations-safety** - Git 操作安全规范
- **pull-request-guidelines** - PR 创建规范
- **language-preferences** - 语言使用规范

---

## 版本历史

- **v1.0.0** (当前) - 初始版本

---

## 作者和贡献

- 作者: LeekJay
- 邮箱: leekjay@foxmail.com
- 仓库: https://github.com/LeekJay/claude-skills-plugin

欢迎提交 Issue 和 PR!
