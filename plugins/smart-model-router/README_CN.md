# Smart Model Router - 智能模型路由器

> 智能多模型路由系统，根据任务特性动态选择最优 AI 模型

## 概述

Smart Model Router 是一个 Claude Code 插件，它能够智能地将任务路由到最合适的 AI 模型执行：

- **Claude Opus 4.5**: 复杂推理、架构决策、安全关键代码
- **Claude Sonnet 4.5**: Claude 特有功能场景
- **Claude Haiku 4.5**: 代码审查、简单编码、实时任务
- **GPT-5.2**: 复杂编码、模板生成、多文件重构 ⭐ 默认选择
- **Gemini CLI**: 大型代码库分析、前端设计、全局理解

## 核心价值

### 质量提升

基于最新基准测试数据：
- **代码审查**: Haiku 胜率 55% vs Sonnet 45% (Qodo 400 PRs 测试)
- **复杂编码**: GPT-5.2 SWE-bench 80.0%，仅次于 Opus 80.9%
- **前端设计**: Gemini 在 UI 任务上表现最优
- **架构决策**: Opus 80.9% SWE-bench，首个突破 80%

### GPT-5.2: 通用编码主力 ⭐

GPT-5.2 (2025年12月11日发布) 提供接近 Opus 的编码与推理能力：
- **80.0% SWE-bench** (vs Opus 80.9%, Sonnet 77.2%)
- **98.7% 工具调用准确率**
- **256K 上下文窗口**，近乎完美的召回率

## 安装

### 前置要求

1. **Claude Code** 已安装并配置
2. **Codex CLI** (用于 GPT-5.2)
   ```bash
   npm install -g @openai/codex
   export OPENAI_API_KEY="your-key"
   ```
3. **Gemini CLI** (可选，用于大上下文/前端任务)
   ```bash
   npm install -g @google/gemini-cli
   export GOOGLE_API_KEY="your-key"
   ```

### 插件安装

```bash
# 克隆插件仓库
git clone https://github.com/LeekJay/claude-skills-plugin.git

# 链接到 Claude Code
cd claude-skills-plugin
# 按照 Claude Code 插件安装指南操作
```

## 使用方法

### 方式 1: 自动路由 (推荐)

激活技能后，插件会自动分析任务并路由到最优模型：

```
用户: "Review this pull request"
插件: [检测到代码审查任务 → 复杂/关键审查路由到 GPT-5.2；轻量/实时审查路由到 Haiku]
结果: 输出审查要点与风险提示
```

### 方式 2: 显式编排

对于复杂任务，使用两层架构：

```
# 调用编排器进行智能路由
Task(
  subagent_type="smart-model-router:model-orchestrator",
  prompt="Design the caching strategy for our microservices"
)
```

编排器会输出：
- 任务分析
- 模型选择（含原因）
- 优化后的提示词

### 方式 3: 手动指定

用户可以显式指定模型：
```
用户: "Use Opus for this architecture review"
插件: [跳过路由，直接使用 Opus]
```

## 路由决策树

```
任务接收
    │
    ├─ 前端 UI / 设计任务?
    │   └─ 是 → Gemini CLI (最高优先级)
    │
    ├─ 上下文 > 100K tokens / 涉及大量文件?
    │   └─ 是 → Gemini CLI
    │
    ├─ 架构 / 安全关键?
    │   └─ 是 → Opus
    │
    ├─ 需要 Claude 特有功能?
    │   └─ 是 → Sonnet
    │
    ├─ 代码审查 / PR 审查?
    │   ├─ 复杂/关键审查 → GPT-5.2 (必要时回退 Opus)
    │   └─ 轻量/实时审查 → Haiku
    │
    ├─ 简单 + 实时响应?
    │   └─ 是 → Haiku
    │
    └─ 默认 → GPT-5.2 (能力优先默认) ⭐
```

## 模型能力矩阵

```
任务类型         | GPT-5.2 | Haiku | Gemini | Sonnet | Opus
-----------------|---------|-------|--------|--------|------
代码审查          |  ★★★    |   ★★  |   ★★   |  ★★★   |  ★★★
模板生成          |  ★★★    |   ★★  |   ★★   |   ★★   |  ★★
复杂编码          |  ★★★    |   ★★  |   ★★   |   ★★   | ★★★
前端 UI           |   ★★    |   ★★  |  ★★★   |   ★★   |  ★★
大上下文          |   ★★    |   ★   |  ★★★   |   ★★   |  ★★
多文件重构        |  ★★★    |   ★★  |   ★★   |  ★★★   | ★★★
架构设计          |   ★★    |   ★★  |   ★★   |   ★★   | ★★★
安全分析          |   ★★    |   ★★  |   ★★   |   ★★   | ★★★
```

## 架构设计

### 三层架构 ⭐ 新特性

```
┌─────────────────────────────────────────────────────────────┐
│                      用户请求                                │
└───────────────────────────┬─────────────────────────────────┘
                            ▼
┌─────────────────────────────────────────────────────────────┐
│         Layer 1: Haiku 决策层 (model-orchestrator)          │
│                                                             │
│   • 任务分类 (< 500 tokens)                                 │
│   • 模型选择 (默认 GPT-5.2)                                 │
│   • 提示词优化                                              │
└───────────────────────────┬─────────────────────────────────┘
                            │
        ┌───────────────────┼───────────────────┐
        ▼                   ▼                   ▼
┌─────────────┐     ┌─────────────┐     ┌─────────────┐
│ Claude 内部  │     │ GPT-5.2     │     │ Gemini CLI  │
│ (Haiku/     │     │ (Codex      │     │             │
│  Sonnet/    │     │  CLI)       │     │ model-      │
│  Opus)      │     │             │     │ executor    │
└──────┬──────┘     └──────┬──────┘     └──────┬──────┘
       │                   │                   │
       └───────────────────┼───────────────────┘
                           ▼
┌─────────────────────────────────────────────────────────────┐
│       Layer 3: 输出优化层 (output-optimizer) ⭐ 新特性      │
│                                                             │
│   • 对 Codex 精炼输出进行扩展和润色                         │
│   • 添加上下文说明、改进建议、风险提示                       │
│   • 使用 Haiku 进行轻量优化                                 │
└───────────────────────────┬─────────────────────────────────┘
                            ▼
                      返回给用户
```

### 为什么需要输出优化？

**Codex/GPT-5.2 输出特点：**
- ✅ 精炼、直接、技术性强
- ✅ 执行速度快
- ❌ 缺少上下文解释
- ❌ 没有改进建议和风险提示
- ❌ 格式相对简单

**优化后输出：**
- ✅ 保留核心技术内容
- ✅ 添加清晰的结构和分节
- ✅ 补充上下文说明和背景
- ✅ 提供改进建议和潜在风险
- ✅ 更适合团队协作和知识传承

### 输出优化示例

**Codex 原始输出：**
```
Add index on user_id column in orders table.
```

**优化后输出：**
```markdown
## Summary
为 orders 表的 user_id 列添加索引以优化查询性能。

## Solution
```sql
CREATE INDEX idx_orders_user_id ON orders(user_id);
```

## Explanation
当前查询 `SELECT * FROM orders WHERE user_id = ?` 执行全表扫描，
复杂度为 O(n)。添加索引后复杂度降为 O(log n)。

## Potential Risks
⚠️ 在大表上创建索引可能锁表数分钟
⚠️ 索引将增加约 200MB 存储空间

## Next Steps
1. 在非高峰期执行索引创建
2. 验证查询性能提升
```

## 提示词优化

### GPT-5.2 优化策略

原始请求:
```
Add validation to the form
```

优化后:
```
Add form validation to the following React component:

Context: [relevant code]

Requirements:
- Email: must be valid email format
- Password: min 8 chars, 1 uppercase, 1 number

Success Criteria:
- All validations pass
- Error messages displayed correctly

Output: Modified component code with brief explanation
```

### Gemini 优化策略

原始请求:
```
Understand the auth system
```

优化后:
```
Analyze the authentication system:
1. Trace login flow from frontend to database
2. Map all auth-related files and dependencies
3. Identify security patterns used
4. Output findings in structured format
```

## 错误处理

### 故障转移策略

```
主模型失败
    │
    ├─ GPT-5.2 失败 → 尝试 Opus
    ├─ Haiku 失败 → 尝试 GPT-5.2
    ├─ Sonnet 失败 → 尝试 Opus
    └─ Gemini 失败 → 尝试 GPT-5.2 (分块上下文)
```

### 常见错误

| 错误 | 原因 | 解决方案 |
|------|------|----------|
| CLI not found | 未安装 | `npm install -g @openai/codex` |
| API key missing | 未配置 | `export OPENAI_API_KEY="..."` |
| Rate limit | 请求过多 | 等待后重试或切换模型 |
| Context too large | 超出限制 | 切换到 Gemini 或分块 |

## 与其他插件集成

- **code-quality-standards**: 模型生成代码后运行质量检查
- **bug-fix**: 根据 bug 复杂度路由到合适模型
- **typescript-strict-typing**: 确保所有模型输出类型安全
- **pr-review-toolkit**: 可用 Haiku 进行轻量/实时审查

## 配置选项

在 `CLAUDE.md` 中添加：

```markdown
## Smart Model Router 配置

### 默认行为
- 自动路由: 启用
- 决策层模型: Haiku
- 失败重试: 启用

### 模型偏好
- 前端/UI: Gemini (强制, 最高优先级)
- 大上下文: Gemini (强制)
- 架构/安全关键: Opus (强制)
- 需要 Claude 特有功能: Sonnet
- 代码审查: GPT-5.2 (默认), 轻量/实时 → Haiku
- 默认: GPT-5.2 (能力优先)

```

## 示例场景

### 场景 1: PR 代码审查

```
用户: "Review this PR for the auth module"

路由决策:
  任务类型: 代码审查
  触发规则: 代码审查(复杂/关键) → GPT-5.2
  置信度: 高

执行: GPT-5.2
结果: 深度审查（必要时回退 Opus）
```

### 场景 2: 生成 REST API

```
用户: "Generate CRUD endpoints for Product model"

路由决策:
  任务类型: 模板生成
  触发规则: 默认 → GPT-5.2
  置信度: 高

执行: GPT-5.2
结果: 完整 API 代码与说明
```

### 场景 3: 大型代码库分析

```
用户: "Help me understand this legacy payment system"

路由决策:
  任务类型: 代码库分析
  上下文: > 100K tokens
  触发规则: 大上下文 → Gemini
  置信度: 高

执行: Gemini CLI (1M 上下文)
结果: 全面分析报告
```

### 场景 4: 架构决策

```
用户: "Design caching strategy for microservices"

路由决策:
  任务类型: 架构设计
  关键性: 高
  触发规则: 架构决策 → Opus
  置信度: 高

执行: Opus
结果: 深度架构分析
```

## 性能指标

### 目标

- 决策时间: < 2 秒
- 决策 token 消耗: < 500 tokens
- 路由准确率: > 90%

### 监控

每次执行后记录：
- 使用的模型
- 消耗的 tokens
- 执行时间
- 成功/失败

## 限制

- 外部 CLI 工具 (Codex, Gemini) 需单独安装和配置
- 各服务需要设置 API 密钥
- 某些任务可能需要手动选择模型
- 模型能力可能随更新变化

## 快速参考卡

```
┌─────────────────────────────────────────────────────────────┐
│        Smart Model Router 快速参考 (三层架构 v2.1)          │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Layer 1 (决策): Haiku 分析任务并选择最优模型               │
│  Layer 2 (执行): GPT-5.2/Gemini/Claude 执行任务             │
│  Layer 3 (优化): Haiku 润色和增强输出 ⭐ 新特性             │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│                                                             │
│  Haiku   → 代码审查、简单任务、实时、子代理、输出优化        │
│  GPT-5.2 → 其他所有任务 (默认选择) ⭐                       │
│  Gemini  → 大上下文 (100K+)、前端 UI                        │
│  Sonnet  → Claude 特有功能                                  │
│  Opus    → 架构、安全、关键决策                             │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│  速度: Haiku > GPT-5.2 > Gemini ≈ Sonnet > Opus            │
│  质量: Opus > GPT-5.2 ≈ Sonnet > Gemini > Haiku            │
├─────────────────────────────────────────────────────────────┤
│  输出优化: Codex 精炼输出 → Haiku 润色 → 完整响应           │
└─────────────────────────────────────────────────────────────┘
```

## 贡献

欢迎提交 Issue 和 Pull Request！

## 许可证

MIT License
