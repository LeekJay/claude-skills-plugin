# Bug Fix Plugin (bug-fix)

[English](#english) | [中文](#中文)

---

## English

### Overview

The Bug Fix Plugin provides intelligent bug fixing capabilities through a hybrid skill + sub-agent architecture. It systematically identifies, analyzes, and fixes bugs while ensuring code quality and test coverage.

### Architecture

**Hybrid Design:**
- **Skill Mode**: Handles simple, isolated bugs directly in the main conversation
- **Sub-Agent Mode**: Delegates complex, multi-file bugs to an isolated executor for systematic debugging

### Features

#### Core Capabilities

1. **Bug Identification & Analysis**
   - Parse error messages and stack traces
   - Reproduce bugs reliably
   - Trace execution flow
   - Identify root causes

2. **Systematic Fixing**
   - Test-driven approach (write failing test first)
   - Minimal, targeted changes
   - Defensive programming patterns
   - Comprehensive verification

3. **Multi-Bug Pattern Support**
   - Null/undefined errors
   - Race conditions
   - Memory leaks
   - State synchronization issues
   - Type coercion bugs
   - Error swallowing

4. **Verification & Safety**
   - Automated test execution
   - Type checking
   - Linting
   - Build verification
   - Manual testing guidelines

### When to Use

#### ✅ Use This Plugin For:

**Simple Bugs (Skill Mode)**:
- Single-file bugs
- Logic errors in specific functions
- Missing null checks
- Type errors
- Simple state management issues

**Complex Bugs (Sub-Agent Mode)**:
- Multi-file bugs
- Race conditions
- Memory leaks
- Data consistency issues
- Integration bugs
- Bugs requiring investigation

#### ❌ Don't Use For:

- Feature requests (not bugs)
- Code refactoring (unless fixing a bug)
- Performance optimization (unless it's a performance bug)
- Adding new functionality

### Usage

#### Skill Mode (Simple Bugs)

Invoke the skill for straightforward bugs:

```
The app crashes when clicking on a user without a profile picture.
```

The skill will:
1. Identify the missing null check
2. Apply the fix directly
3. Suggest test cases
4. Report completion

#### Sub-Agent Mode (Complex Bugs)

For complex bugs, the skill automatically delegates to the sub-agent:

```
Search results sometimes show data from previous searches (race condition).
```

The sub-agent will:
1. Investigate the root cause
2. Plan the fix systematically
3. Implement request tracking
4. Add comprehensive tests
5. Verify thoroughly
6. Report detailed findings

**Manual Delegation:**
```typescript
// Use Task tool with proper subagent_type
subagent_type: "bug-fix:bug-fix-executor"
```

### Bug Fixing Workflow

#### Phase 1: Investigation
- Parse bug report
- Gather error messages and logs
- Reproduce the bug
- Trace execution flow
- Identify root cause

#### Phase 2: Planning
- Design minimal fix
- Identify required changes
- Assess risk
- Plan verification

#### Phase 3: Implementation
- Write failing test first (TDD)
- Apply the fix
- Handle edge cases
- Fix related bugs
- Update documentation

#### Phase 4: Verification
- Run tests (new + existing)
- Type checking
- Linting
- Build verification
- Manual testing

#### Phase 5: Reporting
- Document root cause
- Explain fix applied
- List files modified
- Provide verification results
- Suggest follow-up actions

### Bug Priority Levels

- **P0 (Critical)**: App crashes, data loss, security issues - fix immediately
- **P1 (High)**: Major features broken, affects many users - fix soon
- **P2 (Medium)**: Minor features broken, good workarounds exist
- **P3 (Low)**: Cosmetic issues, edge cases

### Best Practices

1. **Understand Before Fixing**
   - Investigate thoroughly
   - Find root cause, not just symptoms
   - Reproduce reliably

2. **Make Minimal Changes**
   - Fix only what's broken
   - Don't refactor unrelated code
   - Keep scope focused

3. **Add Tests**
   - Write test that reproduces bug
   - Ensure test fails before fix
   - Ensure test passes after fix

4. **Verify Thoroughly**
   - Run all tests
   - Check types and linting
   - Verify build
   - Test manually

5. **Document Well**
   - Explain non-obvious fixes
   - Add helpful comments
   - Write clear commit messages

### Examples

#### Example 1: Null Check Bug (Skill Mode)

**Bug Report:**
```
App crashes when clicking on user without profile picture.
Error: TypeError: Cannot read property 'avatar' of null
```

**Fix:**
```typescript
// Before (buggy)
<img src={user.profile.avatar} alt="Avatar" />

// After (fixed)
<img src={user.profile?.avatar ?? '/default-avatar.png'} alt="Avatar" />
```

**Result:**
- Added null safety with optional chaining
- Provided default fallback
- Added test cases

#### Example 2: Race Condition (Sub-Agent Mode)

**Bug Report:**
```
Search results sometimes show data from previous searches.
Happens when typing quickly.
```

**Investigation:**
- Root cause: No cancellation for outdated requests
- Affected files: SearchContext, useSearch hook, SearchResults component

**Fix:**
- Implemented request ID tracking
- Added request cancellation
- Updated components to ignore stale data

**Verification:**
- All tests pass
- Manual testing confirms fix
- No performance degradation

### Integration

Works well with other plugins:
- **typescript-strict-typing**: Ensure type safety in fixes
- **code-quality-standards**: Run quality checks after fixes
- **git-commit-conventions**: Create proper commits for bug fixes
- **claude-code-editor**: For large-scale bug fixes

### Files Structure

```
plugins/bug-fix/
├── .claude-plugin/
│   └── plugin.json           # Plugin configuration
├── skills/
│   └── bug-fix/
│       └── SKILL.md          # Skill prompt (main interface)
├── agents/
│   └── bug-fix-executor.md   # Sub-agent prompt (executor)
└── README_CN.md              # Documentation
```

### Tips for Effective Bug Reporting

When reporting bugs, provide:

1. **Clear description**: Expected vs actual behavior
2. **Reproduction steps**: Step-by-step instructions
3. **Error messages**: Full stack traces
4. **Environment info**: Browser, OS, runtime version
5. **Recent changes**: If known
6. **Impact assessment**: Who's affected, severity

**Good Bug Report Example:**
```
Bug: Login fails with "Invalid token" error

Reproduction:
1. Go to /login
2. Enter valid credentials
3. Click "Sign In"
4. Error appears: "Invalid token"

Error: JWTError: Token signature verification failed
  at verifyToken (auth.js:45)

Environment: Chrome 120, Node 18.17.0
Recent changes: Upgraded jsonwebtoken 8.5.1 → 9.0.0
Impact: All users cannot log in (P0 - Critical)
```

---

## 中文

### 概述

Bug Fix 插件通过 skill + sub-agent 混合架构提供智能化的 bug 修复能力。它能系统化地识别、分析和修复 bug，同时确保代码质量和测试覆盖率。

### 架构设计

**混合架构:**
- **Skill 模式**: 在主对话中直接处理简单、独立的 bug
- **Sub-Agent 模式**: 将复杂、多文件的 bug 委托给独立执行器进行系统化调试

### 功能特性

#### 核心能力

1. **Bug 识别与分析**
   - 解析错误信息和堆栈跟踪
   - 可靠地重现 bug
   - 追踪执行流程
   - 识别根本原因

2. **系统化修复**
   - 测试驱动方法(先写失败测试)
   - 最小化、精准的修改
   - 防御性编程模式
   - 全面的验证

3. **多种 Bug 模式支持**
   - 空值/未定义错误
   - 竞态条件
   - 内存泄漏
   - 状态同步问题
   - 类型强制转换 bug
   - 错误吞咽

4. **验证与安全**
   - 自动化测试执行
   - 类型检查
   - 代码检查
   - 构建验证
   - 手动测试指南

### 使用场景

#### ✅ 适用于:

**简单 Bug (Skill 模式)**:
- 单文件 bug
- 特定函数的逻辑错误
- 缺失的空值检查
- 类型错误
- 简单的状态管理问题

**复杂 Bug (Sub-Agent 模式)**:
- 多文件 bug
- 竞态条件
- 内存泄漏
- 数据一致性问题
- 集成 bug
- 需要调查的 bug

#### ❌ 不适用于:

- 功能请求(不是 bug)
- 代码重构(除非修复 bug)
- 性能优化(除非是性能 bug)
- 添加新功能

### 使用方法

#### Skill 模式(简单 Bug)

直接调用 skill 处理简单 bug:

```
点击没有头像的用户时应用崩溃了
```

Skill 将:
1. 识别缺失的空值检查
2. 直接应用修复
3. 建议测试用例
4. 报告完成情况

#### Sub-Agent 模式(复杂 Bug)

对于复杂 bug,skill 会自动委托给 sub-agent:

```
搜索结果有时会显示之前搜索的数据(竞态条件)
```

Sub-agent 将:
1. 调查根本原因
2. 系统化地规划修复
3. 实现请求跟踪
4. 添加全面的测试
5. 彻底验证
6. 报告详细发现

**手动委托:**
```typescript
// 使用 Task 工具,指定正确的 subagent_type
subagent_type: "bug-fix:bug-fix-executor"
```

### Bug 修复工作流

#### 阶段 1: 调查
- 解析 bug 报告
- 收集错误信息和日志
- 重现 bug
- 追踪执行流程
- 识别根本原因

#### 阶段 2: 规划
- 设计最小化修复方案
- 识别需要的更改
- 评估风险
- 规划验证方法

#### 阶段 3: 实施
- 先写失败测试(TDD)
- 应用修复
- 处理边缘情况
- 修复相关 bug
- 更新文档

#### 阶段 4: 验证
- 运行测试(新增 + 现有)
- 类型检查
- 代码检查
- 构建验证
- 手动测试

#### 阶段 5: 报告
- 记录根本原因
- 解释应用的修复
- 列出修改的文件
- 提供验证结果
- 建议后续操作

### Bug 优先级

- **P0 (严重)**: 应用崩溃、数据丢失、安全问题 - 立即修复
- **P1 (高)**: 主要功能损坏、影响大量用户 - 尽快修复
- **P2 (中)**: 次要功能损坏、有较好的替代方案
- **P3 (低)**: 外观问题、边缘情况

### 最佳实践

1. **理解后再修复**
   - 彻底调查
   - 找到根本原因,而非症状
   - 可靠地重现

2. **最小化修改**
   - 只修复损坏的部分
   - 不要重构无关代码
   - 保持范围聚焦

3. **添加测试**
   - 编写重现 bug 的测试
   - 确保修复前测试失败
   - 确保修复后测试通过

4. **彻底验证**
   - 运行所有测试
   - 检查类型和代码规范
   - 验证构建
   - 手动测试

5. **良好的文档**
   - 解释不明显的修复
   - 添加有用的注释
   - 编写清晰的提交信息

### 示例

#### 示例 1: 空值检查 Bug (Skill 模式)

**Bug 报告:**
```
点击没有头像的用户时应用崩溃
错误: TypeError: Cannot read property 'avatar' of null
```

**修复:**
```typescript
// 修复前(有 bug)
<img src={user.profile.avatar} alt="Avatar" />

// 修复后
<img src={user.profile?.avatar ?? '/default-avatar.png'} alt="Avatar" />
```

**结果:**
- 使用可选链添加了空值安全
- 提供了默认回退
- 添加了测试用例

#### 示例 2: 竞态条件 (Sub-Agent 模式)

**Bug 报告:**
```
搜索结果有时显示之前搜索的数据
快速输入时会发生
```

**调查:**
- 根本原因: 没有取消过期的请求
- 受影响文件: SearchContext、useSearch hook、SearchResults 组件

**修复:**
- 实现了请求 ID 跟踪
- 添加了请求取消机制
- 更新组件以忽略过期数据

**验证:**
- 所有测试通过
- 手动测试确认修复
- 无性能下降

### 集成

与其他插件配合良好:
- **typescript-strict-typing**: 确保修复中的类型安全
- **code-quality-standards**: 修复后运行质量检查
- **git-commit-conventions**: 为 bug 修复创建合适的提交
- **claude-code-editor**: 用于大规模 bug 修复

### 文件结构

```
plugins/bug-fix/
├── .claude-plugin/
│   └── plugin.json           # 插件配置
├── skills/
│   └── bug-fix/
│       └── SKILL.md          # Skill 提示词(主接口)
├── agents/
│   └── bug-fix-executor.md   # Sub-agent 提示词(执行器)
└── README_CN.md              # 文档
```

### 有效的 Bug 报告技巧

报告 bug 时,请提供:

1. **清晰描述**: 期望行为 vs 实际行为
2. **重现步骤**: 逐步说明
3. **错误信息**: 完整的堆栈跟踪
4. **环境信息**: 浏览器、操作系统、运行时版本
5. **最近更改**: 如果知道
6. **影响评估**: 受影响人群、严重程度

**良好的 Bug 报告示例:**
```
Bug: 登录失败,显示"Invalid token"错误

重现步骤:
1. 访问 /login
2. 输入有效凭据
3. 点击"登录"
4. 出现错误: "Invalid token"

错误: JWTError: Token signature verification failed
  at verifyToken (auth.js:45)

环境: Chrome 120, Node 18.17.0
最近更改: 升级 jsonwebtoken 8.5.1 → 9.0.0
影响: 所有用户无法登录 (P0 - 严重)
```

---

## License

MIT

## Author

LeekJay (leekjay@foxmail.com)

## Repository

https://github.com/LeekJay/claude-skills-plugin
