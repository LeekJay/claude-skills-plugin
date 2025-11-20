# Claude Code 开发最佳实践插件

[English](./README.md)

一个为 Claude Code 提供的开发最佳实践代理技能集合，包括 Git 规范、代码质量标准、文档指南和前端架构设计。

## 📦 包含内容

本插件提供 7 个核心技能，帮助 Claude 理解并执行开发最佳实践：

### 1. **Git 提交规范** (`git-commit-conventions`)
使用 Conventional Commits 格式标准化 Git 提交信息，使用中文主题。

**使用场景：**
- 创建 Git 提交
- 检查提交信息格式
- 准备 Pull Request

**主要特性：**
- Conventional Commits 格式 + 中文主题
- 严格禁止在提交信息中包含 AI 标识
- 支持自动化工具集成

### 2. **Pull Request 指南** (`pull-request-guidelines`)
标准化 Pull Request 工作流和内容要求。

**使用场景：**
- 创建 PR
- 准备代码审查
- 验证 PR 内容

**主要特性：**
- 中文摘要要求
- 关联 Issue 链接
- 验证步骤和影响评估

### 3. **代码质量标准** (`code-quality-standards`)
代码质量检查标准和工作流。

**使用场景：**
- 完成功能实现后
- 修复 Bug 后
- 创建提交/PR 之前

**主要特性：**
- 格式化、Lint、类型检查、测试的执行顺序
- 严格的 TypeScript 类型错误修复规则
- 异常场景处理

### 4. **语言偏好** (`language-preferences`)
沟通时的语言和交互标准。

**使用场景：**
- 与用户沟通
- 解释代码
- 报告错误
- 提供文档

**主要特性：**
- 中英文使用场景划分
- 技术术语保持原文
- 需求澄清流程

### 5. **文档指南** (`documentation-guidelines`)
文档创建和更新标准。

**使用场景：**
- 考虑创建文档时
- 代码变更涉及文档时

**主要特性：**
- 除非关键任务，否则不主动创建文档
- 仅在代码变更通过所有检查后更新相关文档

### 6. **Git 操作安全** (`git-operations-safety`)
Git 操作安全标准。

**使用场景：**
- 执行任何 Git 操作之前
- 特别是破坏性操作（reset、force push、clean）之前

**主要特性：**
- 操作前必须运行 git status
- 破坏性操作需要用户明确确认
- 对 main/master 分支的 force push 有特殊限制

### 7. **前端架构设计** (`frontend-architecture-design`)
审查和重新设计前端项目架构。

**使用场景：**
- 需要架构审查
- 代码结构分析
- 架构重新设计
- 性能优化
- 架构改进

**主要特性：**
- 从 5 个维度分析：性能、可维护性、可扩展性、工程化、类型安全
- 支持 React、Vue 等技术栈
- 提供详细的审查报告和可操作的建议

## 🚀 安装

### 前提条件
- 已安装并运行 Claude Code
- 系统已安装 Git

### 快速安装

1. **将此市场添加到 Claude Code：**
```shell
/plugin marketplace add LeekJay/claude-skills-plugin
```

2. **安装插件：**
```shell
/plugin install development-best-practices@LeekJay
```

3. **重启 Claude Code** 以激活技能

4. **验证安装：**
技能将自动可用。可通过以下命令检查：
```shell
claude --help
```

### 备选：本地安装

1. **克隆此仓库：**
```bash
git clone https://github.com/LeekJay/claude-skills-plugin.git
cd claude-skills-plugin
```

2. **添加为本地市场：**
```shell
/plugin marketplace add ./claude-skills-plugin
```

3. **安装插件：**
```shell
/plugin install development-best-practices@local
```

## 📖 使用方法

安装后，Claude 会在检测到适当的上下文时自动使用这些技能。你无需手动调用它们 - Claude 会根据你的请求识别何时应用每个技能。

### 示例场景

**当你提交代码时：**
```bash
# Claude 将自动应用 git-commit-conventions
git add .
# 让 Claude 创建提交
```

**当你创建 PR 时：**
```bash
# Claude 将应用 pull-request-guidelines
# 让 Claude 创建 Pull Request
```

**当你请求架构审查时：**
```
"请审查我项目的前端架构"
# Claude 将使用 frontend-architecture-design 技能
```

**在任何 git 操作之前：**
```bash
# Claude 将在执行破坏性 git 命令之前
# 自动应用 git-operations-safety
```

## 🏗️ 插件结构

```
claude-skills-plugin/
├── .claude-plugin/
│   ├── plugin.json              # 插件元数据
│   └── marketplace.json         # 市场配置
├── skills/                       # 所有代理技能
│   ├── git-commit-conventions/
│   │   └── SKILL.md
│   ├── pull-request-guidelines/
│   │   └── SKILL.md
│   ├── code-quality-standards/
│   │   └── SKILL.md
│   ├── language-preferences/
│   │   └── SKILL.md
│   ├── documentation-guidelines/
│   │   └── SKILL.md
│   ├── git-operations-safety/
│   │   └── SKILL.md
│   └── frontend-architecture-design/
│       ├── SKILL.md
│       ├── PERFORMANCE.md
│       ├── MAINTAINABILITY.md
│       ├── SCALABILITY.md
│       ├── ENGINEERING.md
│       └── TYPESCRIPT.md
├── README.md                     # 英文文档
├── README_CN.md                  # 本文件
└── LICENSE                       # MIT 许可证
```

## 🔧 自定义

你可以为你的团队自定义这些技能：

1. Fork 此仓库
2. 修改 `skills/` 目录中的技能文件以匹配你团队的标准
3. 更新 `plugin.json` 中的信息
4. 通过你自己的市场与团队共享

## 🤝 贡献

欢迎贡献！以下是你可以帮助的方式：

1. Fork 此仓库
2. 创建功能分支 (`git checkout -b feature/amazing-skill`)
3. 提交你的更改（遵循我们的 git-commit-conventions！）
4. 推送到分支 (`git push origin feature/amazing-skill`)
5. 打开 Pull Request（遵循我们的 pull-request-guidelines！）

## 📄 许可证

本项目采用 MIT 许可证 - 详见 [LICENSE](LICENSE) 文件。

## 🙏 致谢

- 为 [Claude Code](https://claude.com/claude-code) 构建
- 受行业最佳实践和团队协作需求启发
- 感谢所有贡献者和用户

## 📞 支持

- **Issues**: [GitHub Issues](https://github.com/LeekJay/claude-skills-plugin/issues)
- **讨论**: [GitHub Discussions](https://github.com/LeekJay/claude-skills-plugin/discussions)
- **文档**: [Claude Code 插件指南](https://docs.anthropic.com/claude/docs/plugins)

## 🔄 版本历史

### v1.0.0（首次发布）
- Git 提交规范技能
- Pull Request 指南技能
- 代码质量标准技能
- 语言偏好技能
- 文档指南技能
- Git 操作安全技能
- 前端架构设计技能

---

**用 ❤️ 打造更好的开发实践**
