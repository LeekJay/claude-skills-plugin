# Backend Architecture Design 插件

## 插件作用

这是一个专业的 Python FastAPI 后端架构审查与设计插件,帮助你全面评估和改进 FastAPI 项目的架构质量。从**六个关键维度**对项目进行深度分析,并提供可执行的改进建议。

**核心价值:**
- 🔍 系统化架构审查 - 从多个维度全面评估后端项目架构
- 📊 量化评分和优先级 - 清晰的评分标准和改进优先级排序
- 💡 可执行的建议 - 提供具体的代码位置和改进步骤
- 🛡️ 安全性优先 - 重点关注认证、授权和数据安全
- ⚡ 性能导向 - 强调异步编程和数据库查询优化

---

## 运作方式

### 架构组成

插件采用 **Skill + Sub-agent 混合架构**:

```
用户请求后端架构审查
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
- 位置: `skills/backend-architecture-design/SKILL.md`
- 作用: Claude 自动发现和使用,提供架构审查的背景知识
- 触发: 当用户提到"后端架构"、"API 设计"、"数据库优化"等关键词时自动激活

**2. Sub-agent (子代理)**
- 位置: `agents/backend-architecture-design.md`
- 作用: 在独立上下文中执行完整的架构审查流程
- 触发: 用户显式请求或 Claude 自动委派
- 优势: 独立上下文避免污染主对话,适合长流程分析

**3. 详细指南文档**
- `API_DESIGN.md` - RESTful API 设计、Pydantic 模型、依赖注入
- `DATA_ARCHITECTURE.md` - ORM 设计、查询优化、数据库迁移
- `SECURITY.md` - 认证授权、数据验证、注入防护
- `ASYNC_PERFORMANCE.md` - 异步编程、性能优化、缓存策略
- `CODE_ORGANIZATION.md` - 分层架构、项目结构、依赖管理
- `TESTING.md` - pytest 测试、API 测试、覆盖率

---

## 工作方式

### 六大审查维度

Sub-agent 会从以下六个维度全面分析项目:

#### 1. API 设计 (API Design)
- ✓ RESTful API 设计模式和命名规范
- ✓ Pydantic 模型设计 (Request/Response/Internal)
- ✓ 依赖注入使用 (`Depends()`)
- ✓ OpenAPI 文档完整性
- ✓ API 版本管理和错误响应标准化

#### 2. 数据架构 (Data Architecture)
- ✓ SQLAlchemy/Tortoise ORM 模型设计
- ✓ 数据库查询优化和 N+1 问题检测
- ✓ Alembic 迁移管理
- ✓ 数据库连接池配置
- ✓ 异步 ORM 使用模式

#### 3. 安全性 (Security)
- ✓ OAuth2/JWT 认证实现
- ✓ 授权和权限模型
- ✓ Pydantic 数据验证
- ✓ CORS 配置和注入防护
- ✓ 密码哈希和敏感数据处理

#### 4. 异步性能 (Async Performance)
- ✓ 正确的 async/await 使用
- ✓ 后台任务处理 (BackgroundTasks)
- ✓ 数据库连接池
- ✓ 缓存策略和响应优化
- ✓ 避免异步上下文中的阻塞操作

#### 5. 代码组织 (Code Organization)
- ✓ 分层架构 (router/service/repository)
- ✓ 依赖注入模式
- ✓ 中间件使用和异常处理
- ✓ 配置管理
- ✓ 业务逻辑与路由分离

#### 6. 测试与质量 (Testing & Quality)
- ✓ pytest 测试覆盖率
- ✓ API 集成/端到端测试
- ✓ Mock 和 fixture 使用
- ✓ 类型注解完整性 (mypy)
- ✓ 代码格式化 (black, ruff)

### 系统化审查流程

**Step 1: 项目概览**
1. 使用 Glob 查找配置文件 (pyproject.toml, alembic.ini, .env.example 等)
2. 使用 Read 读取配置文件,了解技术栈、依赖、数据库
3. 使用 Glob 查看目录结构 (app/**, tests/**, alembic/versions/**)

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
请使用 backend-architecture-design agent 审查当前 FastAPI 项目的架构
```

或者指定关注的维度:

```
请使用 backend-architecture-design agent 重点分析项目的安全性和 API 设计
```

### 方式 2: 自然语言请求 (Claude 自动决定是否使用)

```
帮我审查一下 FastAPI 后端项目的架构,看看有哪些需要改进的地方
```

```
这个项目的数据库查询性能很差,能帮我分析一下吗?
```

```
检查一下项目的安全性,特别是认证和授权部分
```

### 方式 3: 针对性分析

```
分析项目的 N+1 查询问题,找出所有需要优化的地方
```

```
检查项目中的异步使用是否正确,有没有阻塞事件循环的代码
```

```
审查 API 端点的权限控制,确保没有安全漏洞
```

---

## 使用例子

### 例子 1: 全面架构审查

**用户请求:**
```
请全面审查当前 FastAPI 项目的架构,重点关注安全性和性能
```

**Sub-agent 执行流程:**
1. 读取 pyproject.toml, alembic.ini, .env.example
2. 分析目录结构 app/**/*
3. 检查认证实现 (JWT, OAuth2)
4. 分析数据库查询模式 (N+1 问题)
5. 查找异步使用问题
6. 评估 API 设计规范

**生成报告示例:**
```markdown
# Backend Architecture Review Report

## Project Overview
- Framework: FastAPI 0.104.0
- Python Version: 3.11
- ORM: SQLAlchemy 2.0 (Async)
- Database: PostgreSQL 14
- Main Dependencies: pydantic, alembic, httpx, redis

## Architecture Scores

| Dimension | Score | Notes |
|-----------|-------|-------|
| API Design | ⭐⭐⭐⭐☆ | 良好的 RESTful 设计,部分缺少文档 |
| Data Architecture | ⭐⭐⭐☆☆ | 存在 N+1 查询问题 |
| Security | ⭐⭐⭐⭐⭐ | 完善的认证授权机制 |
| Async Performance | ⭐⭐⭐☆☆ | 部分路由使用同步库 |
| Code Organization | ⭐⭐⭐⭐☆ | 清晰的分层架构 |
| Testing & Quality | ⭐⭐⭐⭐☆ | 测试覆盖率 85% |

## Issues Found

### Data Architecture Related
1. **N+1 查询问题**
   - Location: `app/api/v1/endpoints/posts.py:45`
   - Impact: High
   - Recommendation: 使用 selectinload() 预加载 author 关系

2. **缺少数据库索引**
   - Location: `app/models/post.py:15`
   - Impact: Medium
   - Recommendation: 在 author_id 和 published 字段上添加复合索引

### Async Performance Related
1. **异步路由中使用同步 HTTP 库**
   - Location: `app/services/external_api.py:23`
   - Impact: High
   - Recommendation: 将 requests 替换为 httpx.AsyncClient

## Improvement Priorities

### P0 (Must fix immediately)
- [ ] 修复 N+1 查询问题 (app/api/v1/endpoints/posts.py:45)
- [ ] 替换同步 HTTP 库为异步版本 (app/services/external_api.py:23)

### P1 (Should fix short-term)
- [ ] 添加数据库索引 (app/models/post.py:15)
- [ ] 完善 API 文档示例

### P2 (Medium to long-term optimization)
- [ ] 引入 Redis 缓存层
- [ ] 优化大数据导出功能

## Architecture Recommendations

### Short-term Improvements
1. 修复 N+1 查询
   ```python
   stmt = select(Post).options(selectinload(Post.author))
   ```

2. 使用异步 HTTP 客户端
   ```python
   async with httpx.AsyncClient() as client:
       response = await client.get(url)
   ```

### Medium-term Optimizations
1. 引入 Redis 缓存
2. 实现 API 响应压缩
3. 添加请求速率限制

### Long-term Planning
1. 考虑微服务拆分
2. 实现 CQRS 模式
```

---

## 常见反模式识别

Sub-agent 能自动识别这些常见架构问题:

### 后端特有反模式

- ❌ **Fat Router** - 路由函数包含大量业务逻辑
- ❌ **Sync in Async** - 异步路由中使用同步 I/O
- ❌ **Missing Dependency Injection** - 硬编码依赖而非使用 `Depends()`
- ❌ **Poor Pydantic Model Design** - 请求/响应/数据库复用同一模型
- ❌ **N+1 Query Problem** - 循环中执行数据库查询
- ❌ **No Database Session Management** - 数据库会话生命周期管理不当
- ❌ **Missing Error Handling** - 缺少自定义异常处理器
- ❌ **Weak Security Practices** - 缺少或不当的认证授权

---

## 最佳实践

### 何时使用这个插件

✅ **推荐使用的场景:**
- 项目启动时规划后端架构
- 定期架构审查 (季度/半年)
- 大规模重构前的评估
- 性能问题诊断和优化
- 安全审计
- 代码审查时发现架构问题
- 技术栈迁移 (如从 Flask 迁移到 FastAPI)

❌ **不适合的场景:**
- 简单的 bug 修复
- 单个函数的代码审查
- 纯文档修改

### 如何充分利用审查结果

1. **优先处理 P0 问题** - 这些是必须立即修复的架构缺陷
2. **制定改进计划** - 将 P1/P2 问题纳入迭代计划
3. **团队讨论** - 基于报告组织架构改进讨论会
4. **持续跟踪** - 定期重新审查,验证改进效果
5. **建立标准** - 将最佳实践固化为团队规范

---

## 技术细节

**工具权限:**
- `Read` - 读取配置文件和源代码
- `Grep` - 搜索代码模式
- `Glob` - 查找文件
- `Bash` - 执行测试命令和数据库查询分析

**使用的模型:**
- Sonnet - 平衡分析深度和速度

**独立上下文优势:**
- 不污染主对话
- 可以进行深度递归分析
- 独立的 token 预算

---

## 支持的技术栈

- ✅ FastAPI (主要支持)
- ✅ SQLAlchemy (Sync/Async)
- ✅ Tortoise ORM
- ✅ Alembic (数据库迁移)
- ✅ Pydantic v2
- ✅ PostgreSQL / MySQL / SQLite
- ✅ Redis (缓存)
- ✅ pytest (测试)

---

## 常见问题

**Q: Sub-agent 会修改我的代码吗?**
A: 不会。这个 Sub-agent 仅进行分析和提供建议,不会修改任何代码。

**Q: 分析一个项目需要多长时间?**
A: 取决于项目规模,通常 2-5 分钟。小型项目可能只需 1 分钟。

**Q: 可以只分析特定维度吗?**
A: 可以。在请求时指定维度即可,例如"重点分析安全性"。

**Q: 支持 Django 吗?**
A: 当前版本主要针对 FastAPI 优化,未来可能支持 Django。

**Q: 报告中的评分标准是什么?**
A: ⭐⭐⭐⭐⭐ (优秀), ⭐⭐⭐⭐☆ (良好), ⭐⭐⭐☆☆ (中等), ⭐⭐☆☆☆ (需改进), ⭐☆☆☆☆ (严重问题)

**Q: P0/P1/P2 优先级如何定义?**
A:
- P0: 严重影响性能、安全或可维护性,必须立即修复
- P1: 明显的问题,应在短期内修复
- P2: 改进机会,可在中长期优化

---

## 版本历史

- **v1.0.0** (当前) - 初始版本,支持 Python FastAPI 项目架构审查

---

## 作者和贡献

- 作者: LeekJay
- 邮箱: leekjay@foxmail.com
- 仓库: https://github.com/LeekJay/claude-skills-plugin

欢迎提交 Issue 和 PR!
