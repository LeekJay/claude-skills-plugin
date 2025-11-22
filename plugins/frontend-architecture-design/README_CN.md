# Frontend Architecture Design 插件

## 插件作用

这是一个专业的前端架构审查与设计插件，帮助你全面评估和改进前端项目的架构质量。支持 React、Vue 等主流技术栈，从**五个关键维度**对项目进行深度分析，并提供可执行的改进建议。

**核心价值:**
- 🔍 系统化架构审查 - 从多个维度全面评估项目架构
- 📊 量化评分和优先级 - 清晰的评分标准和改进优先级排序
- 💡 可执行的建议 - 提供具体的代码位置和改进步骤
- 🎯 技术栈无关 - 支持 React、Vue、Svelte 等多种框架

---

## 运作方式

### 架构组成

插件采用 **Skill + Sub-agent 混合架构**:

```
用户请求架构审查
    ↓
Skill 自动激活 (提供背景知识和指导)
    ↓
Claude 决定委派给 Sub-agent
    ↓
Sub-agent 在独立上下文中执行深度分析:
  - 项目概览
  - 维度分析
  - 生成报告
    ↓
返回结构化的架构审查报告
```

### 组件说明

**1. Skill (技能)**
- 位置: `skills/frontend-architecture-design/SKILL.md`
- 作用: Claude 自动发现和使用，提供架构审查的背景知识
- 触发: 当用户提到"架构"、"重构"、"优化"等关键词时自动激活

**2. Sub-agent (子代理)**
- 位置: `agents/frontend-architecture-design.md`
- 作用: 在独立上下文中执行完整的架构审查流程
- 触发: 用户显式请求或 Claude 自动委派
- 优势: 独立上下文避免污染主对话，适合长流程分析

**3. 详细指南文档**
- `PERFORMANCE.md` - 性能优化详细指南
- `MAINTAINABILITY.md` - 可维护性详细指南
- `SCALABILITY.md` - 可扩展性详细指南
- `ENGINEERING.md` - 工程化详细指南
- `TYPESCRIPT.md` - 类型安全详细指南

---

## 工作方式

### 五大审查维度

Sub-agent 会从以下五个维度全面分析项目:

#### 1. 性能优化 (Performance)
- ✓ 代码分割和懒加载策略
- ✓ 资源加载和缓存机制
- ✓ 运行时性能优化
- ✓ 构建产物分析

#### 2. 可维护性 (Maintainability)
- ✓ 目录结构和代码组织
- ✓ 模块化和关注点分离
- ✓ 设计模式应用
- ✓ 代码复用策略

#### 3. 可扩展性 (Scalability)
- ✓ 插件架构设计
- ✓ 微前端架构
- ✓ 组件库和工具库设计
- ✓ API 设计和抽象层

#### 4. 工程化 (Engineering)
- ✓ 构建工具配置
- ✓ 代码规范和质量控制
- ✓ CI/CD 集成
- ✓ 开发者体验优化

#### 5. 类型安全 (Type Safety)
- ✓ TypeScript 配置
- ✓ 类型系统设计
- ✓ 类型安全实践
- ✓ 类型工具使用

### 系统化审查流程

**Step 1: 项目概览**
1. 使用 Glob 查找配置文件 (package.json, tsconfig.json, vite.config.* 等)
2. 使用 Read 读取配置文件，了解技术栈、依赖、工具链
3. 使用 Glob 查看目录结构 (src/**, public/**, tests/**)

**Step 2: 维度分析**
- 根据项目需求和用户请求选择相关维度
- 深入分析每个维度的具体问题
- 使用 Grep 查找代码模式和反模式

**Step 3: 生成报告**
- 为每个维度打分 (⭐⭐⭐⭐⭐)
- 列出发现的问题 (包含具体位置和影响评估)
- 按优先级排序改进建议 (P0/P1/P2)
- 提供短期、中期、长期改进计划

---

## 怎么使用

### 方式 1: 显式调用 Sub-agent (推荐用于深度分析)

```
请使用 frontend-architecture-design agent 审查当前项目的架构
```

或者指定关注的维度:

```
请使用 frontend-architecture-design agent 重点分析项目的性能和可维护性
```

### 方式 2: 自然语言请求 (Claude 自动决定是否使用)

```
帮我审查一下前端项目的架构，看看有哪些需要改进的地方
```

```
这个项目的架构感觉不太合理，能帮我分析一下吗？
```

```
我们准备重构这个模块，先帮我评估一下现有架构
```

### 方式 3: 针对性分析

```
分析项目的代码分割策略，找出优化机会
```

```
对比 src/old-feature 和 src/new-feature 的架构设计，总结最佳实践
```

```
我们计划从 Webpack 迁移到 Vite，帮我分析需要考虑的架构调整
```

---

## 使用例子

### 例子 1: 全面架构审查

**用户请求:**
```
请全面审查当前 React 项目的架构，重点关注性能和可维护性
```

**Sub-agent 执行流程:**
1. 读取 package.json, tsconfig.json, vite.config.ts
2. 分析目录结构 src/**/*
3. 检查路由配置中的懒加载使用情况
4. 分析组件文件大小和职责分离
5. 查找代码重复和重构机会
6. 评估状态管理方案

**生成报告示例:**
```markdown
# Frontend Architecture Review Report

## Project Overview
- Tech Stack: React 18 + TypeScript + Vite
- TypeScript Version: 5.2.0
- Main Dependencies: react-router-dom, zustand, tailwindcss

## Architecture Scores

| Dimension | Score | Notes |
|-----------|-------|-------|
| Performance | ⭐⭐⭐☆☆ | 缺少代码分割，部分组件未优化 |
| Maintainability | ⭐⭐⭐⭐☆ | 目录结构清晰，但存在 God Component |
| Scalability | ⭐⭐⭐⭐⭐ | 良好的抽象层设计 |
| Engineering | ⭐⭐⭐⭐☆ | 完善的工程化配置 |
| Type Safety | ⭐⭐⭐☆☆ | 存在较多 any 使用 |

## Issues Found

### Performance Related
1. **缺少路由级别的代码分割**
   - Location: `src/router/index.tsx:15`
   - Impact: High
   - Recommendation: 使用 React.lazy() 和 Suspense 实现路由懒加载

2. **大型第三方库未按需引入**
   - Location: `src/utils/icons.ts:3`
   - Impact: Medium
   - Recommendation: 使用 tree-shaking 或按需引入

### Maintainability Related
1. **God Component 问题**
   - Location: `src/pages/Dashboard.tsx:1` (450 lines)
   - Impact: High
   - Recommendation: 拆分为多个子组件，提取自定义 Hooks

## Improvement Priorities

### P0 (Must fix immediately)
- [ ] 拆分 Dashboard 组件 (src/pages/Dashboard.tsx:1)
- [ ] 实现路由懒加载 (src/router/index.tsx:15)

### P1 (Should fix short-term)
- [ ] 减少 any 类型使用 (全局搜索结果: 23 处)
- [ ] 优化图标库引入方式 (src/utils/icons.ts:3)

### P2 (Medium to long-term optimization)
- [ ] 引入 React.memo 优化重渲染
- [ ] 配置构建产物分析

## Architecture Recommendations

### Short-term Improvements
1. 拆分 Dashboard 组件
   - 提取 DashboardHeader, DashboardStats, DashboardCharts
   - 创建自定义 Hook: useDashboardData

2. 实现路由懒加载
   ```typescript
   const Dashboard = lazy(() => import('./pages/Dashboard'))
   const Settings = lazy(() => import('./pages/Settings'))
   ```

### Medium-term Optimizations
1. 建立组件库
2. 优化状态管理粒度
3. 引入性能监控

### Long-term Planning
1. 考虑微前端架构
2. 建立设计系统
```

---

### 例子 2: 针对性分析

**用户请求:**
```
分析项目的代码分割策略，找出优化机会
```

**Sub-agent 聚焦分析:**
1. 检查路由配置
2. 分析 dynamic import 使用情况
3. 检查 package.json 依赖大小
4. 查看构建配置中的 code splitting 设置

**精简报告:**
```markdown
# 代码分割策略分析

## 当前状态
- ❌ 路由未实现懒加载
- ❌ 第三方库未按需引入
- ✅ Vite 默认启用了代码分割

## 优化机会

1. **实现路由懒加载** (预计减少 40% 首屏加载)
2. **按需引入 lodash** (预计减少 70KB)
3. **配置 chunk 分割策略** (改善缓存命中率)

## 具体实施步骤
[详细步骤...]
```

---

### 例子 3: 迁移规划

**用户请求:**
```
我们计划从 Webpack 迁移到 Vite，帮我分析需要考虑的架构调整
```

**Sub-agent 对比分析:**
1. 读取当前 webpack.config.js
2. 识别需要迁移的配置
3. 分析潜在的兼容性问题
4. 提供迁移清单

**迁移报告:**
```markdown
# Webpack → Vite 迁移架构分析

## 需要调整的配置

### 构建配置迁移
- [ ] alias 路径配置
- [ ] 环境变量处理
- [ ] CSS 预处理器配置

### 插件迁移
- [ ] html-webpack-plugin → vite-plugin-html
- [ ] copy-webpack-plugin → vite-plugin-static-copy

### 代码调整
- [ ] require() → import
- [ ] process.env → import.meta.env
- [ ] 动态 require → import.meta.glob

## 潜在风险
1. CommonJS 依赖兼容性
2. 开发服务器代理配置差异

## 迁移优先级
P0: 基础配置迁移
P1: 开发环境验证
P2: 生产构建优化
```

---

## 输出标准

每次架构审查都会包含:

1. **具体代码位置** - 使用 `文件路径:行号` 格式
2. **影响评估** - High/Medium/Low
3. **改进建议** - 可执行的具体方法
4. **优先级排序** - P0/P1/P2
5. **参考文档** - 链接到详细的最佳实践文档

---

## 常见反模式识别

Sub-agent 能自动识别这些常见架构问题:

- ❌ **God Component** - 单个组件过大，职责过多
- ❌ **循环依赖** - 模块间存在循环引用
- ❌ **缺少抽象层** - 业务代码直接调用第三方库
- ❌ **过度工程化** - 简单需求使用复杂架构模式
- ❌ **类型系统滥用** - 过度复杂的类型定义或大量 any

---

## 最佳实践

### 何时使用这个插件

✅ **推荐使用的场景:**
- 项目启动时规划架构
- 定期架构审查 (季度/半年)
- 大规模重构前的评估
- 技术栈迁移规划
- 性能优化前的诊断
- 代码审查时发现架构问题

❌ **不适合的场景:**
- 简单的 bug 修复
- 单个组件的代码审查
- 纯文档修改

### 如何充分利用审查结果

1. **优先处理 P0 问题** - 这些是必须立即修复的架构缺陷
2. **制定改进计划** - 将 P1/P2 问题纳入迭代计划
3. **团队讨论** - 基于报告组织架构改进讨论会
4. **持续跟踪** - 定期重新审查，验证改进效果
5. **建立标准** - 将最佳实践固化为团队规范

---

## 技术细节

**工具权限:**
- `Read` - 读取配置文件和源代码
- `Grep` - 搜索代码模式
- `Glob` - 查找文件
- `Bash` - 执行构建命令分析产物

**使用的模型:**
- Sonnet - 平衡分析深度和速度

**独立上下文优势:**
- 不污染主对话
- 可以进行深度递归分析
- 独立的 token 预算

---

## 相关插件

这个插件与以下插件配合使用效果更佳:

- **code-quality-standards** - 代码质量检查
- **typescript-strict-typing** - TypeScript 严格类型约束
- **documentation-guidelines** - 文档规范

---

## 支持的框架和工具

- ✅ React (含 Next.js, Create React App)
- ✅ Vue (含 Nuxt.js, Vue CLI)
- ✅ Svelte (含 SvelteKit)
- ✅ Vite
- ✅ Webpack
- ✅ TypeScript
- ✅ ESLint / Prettier

---

## 常见问题

**Q: Sub-agent 会修改我的代码吗？**
A: 不会。这个 Sub-agent 仅进行分析和提供建议，不会修改任何代码。

**Q: 分析一个项目需要多长时间？**
A: 取决于项目规模，通常 2-5 分钟。小型项目可能只需 1 分钟。

**Q: 可以只分析特定维度吗？**
A: 可以。在请求时指定维度即可，例如"重点分析性能"。

**Q: 报告中的评分标准是什么？**
A: ⭐⭐⭐⭐⭐ (优秀), ⭐⭐⭐⭐☆ (良好), ⭐⭐⭐☆☆ (中等), ⭐⭐☆☆☆ (需改进), ⭐☆☆☆☆ (严重问题)

**Q: P0/P1/P2 优先级如何定义？**
A:
- P0: 严重影响性能、安全或可维护性，必须立即修复
- P1: 明显的问题，应在短期内修复
- P2: 改进机会，可在中长期优化

---

## 版本历史

- **v2.0.0** (当前) - 添加 Sub-agent 支持，增强架构审查能力
- **v1.0.0** - 初始版本，仅包含 Skill

---

## 作者和贡献

- 作者: LeekJay
- 邮箱: leekjay@foxmail.com
- 仓库: https://github.com/LeekJay/claude-skills-plugin

欢迎提交 Issue 和 PR!
