# Code Quality Standards 插件

## 插件作用

这是一个专业的代码质量保障插件,帮助你系统化地执行代码质量检查并自动修复问题。它会按照正确的顺序运行格式化、代码检查、类型检查和测试,并在发现问题时自动修复,迭代直到所有检查通过。

**核心价值:**
- ✅ 自动化质量检查 - 一键执行所有质量检查流程
- 🔄 智能迭代修复 - 发现问题自动修复,失败后重新检查
- 🛡️ 严格类型安全 - 绝对禁止类型逃逸机制,确保真正的类型安全
- 📋 完整报告 - 详细的检查结果和修复总结

---

## 运作方式

### 架构组成

插件采用 **Skill + Sub-agent 混合架构**:

```
功能实现完成
    ↓
Skill 提醒: "应该执行代码质量检查"
    ↓
Claude 决定委派给 Sub-agent
    ↓
Sub-agent 在独立上下文中执行:
  1. 运行格式化
  2. 运行 Lint (修复错误 → 重新检查)
  3. 运行类型检查 (修复类型错误 → 重新检查)
  4. 运行测试 (修复失败 → 重新检查)
  5. 生成完成报告
    ↓
返回: "✅ 所有检查通过,代码已就绪"
```

### 组件说明

**1. Skill (技能)**
- 位置: `skills/code-quality-standards/SKILL.md`
- 作用: 提醒何时需要执行质量检查,提供检查标准
- 触发: 完成功能、修复 bug、提交代码前自动激活

**2. Sub-agent (子代理)**
- 位置: `agents/code-quality-checker.md`
- 作用: 在独立上下文中执行完整的检查和修复流程
- 触发: 用户显式请求或 Claude 自动委派
- 优势: 多轮迭代修复不污染主对话,独立的 token 预算

---

## 工作方式

### 四步质量检查流程

Sub-agent 严格按照以下顺序执行:

#### Step 1: 初始化检查会话

1. 确认任务并创建检查清单
2. 读取 `package.json` 识别可用的检查命令
3. 确定项目使用的工具 (prettier, eslint, typescript, jest/vitest 等)

#### Step 2: 按顺序执行检查

**2.1 代码格式化 (Code Formatting)**
```bash
pnpm format
# 或
npm run format
# 或
npx prettier --write .
```

**目标**: 确保所有文件符合项目代码风格标准

**处理失败**:
- 查看错误信息
- 修复格式问题 (通常可自动修复)
- 重新运行格式化命令
- 通过后进入下一步

---

**2.2 代码检查 (Code Linting)**
```bash
pnpm lint
# 或
npm run lint
# 或
npx eslint . --fix
```

**目标**: 修复所有错误和警告

**严格规则**:
- ❌ 绝对禁止使用 `eslint-disable` 绕过问题
- ❌ 禁止使用 `// eslint-disable-next-line` 注释
- ✅ 必须正确修复每一个 linting 错误

**处理失败**:
1. 仔细阅读错误信息
2. 使用 Read 工具查看有错误的文件
3. 正确修复每个错误 (不是禁用规则!)
4. 重新运行 lint 命令
5. 迭代直到所有错误解决

---

**2.3 类型检查 (Type Checking)**
```bash
pnpm typecheck
# 或
npm run typecheck
# 或
tsc --noEmit
```

**目标**: 确保所有 TypeScript 类型错误都被正确修复

### 🔴 严格的类型错误修复规则

这是插件的**核心特性**,也是最严格的部分:

#### ❌ 绝对禁止的做法

以下方法**永远不允许**用于修复类型错误:

```typescript
// ❌ 类型断言
const value = something as string
const element = doc.getElementById('id') as HTMLInputElement
const data = response as unknown as DataType

// ❌ any 类型
const data: any = fetchData()
function process(param: any) { }

// ❌ TypeScript 抑制注释
// @ts-ignore
// @ts-expect-error

// ❌ ESLint 抑制注释 (针对 TypeScript 规则)
// eslint-disable-next-line @typescript-eslint/no-explicit-any
// eslint-disable-next-line @typescript-eslint/no-unsafe-assignment
/* eslint-disable @typescript-eslint/... */

// ❌ 非空断言
const value = array.find(x => x.id === id)!.name
const result = obj!.property
```

#### ✅ 强制要求的做法

修复类型错误时**必须**:

1. **分析根本原因** - 理解为什么会出现类型错误
2. **修正实际类型** - 修复类型定义,显式声明类型
3. **改进类型定义** - 明确指定参数和返回类型
4. **添加类型守卫** - 使用类型守卫让 TypeScript 自动推断
5. **重构代码结构** - 如果类型错误难以修复,可能是设计问题

#### 正确的修复示例

**示例 1: 使用类型守卫代替类型断言**

```typescript
// ❌ 错误做法
const value = something as string

// ✅ 正确做法
function isString(value: unknown): value is string {
  return typeof value === 'string'
}

if (isString(value)) {
  // TypeScript 知道这里 value 是 string
  console.log(value.toUpperCase())
}
```

**示例 2: 显式类型定义代替 any**

```typescript
// ❌ 错误做法
function process(data: any) {
  return data.value
}

// ✅ 正确做法
interface DataType {
  value: string
}

function process(data: DataType): string {
  return data.value
}
```

**示例 3: 泛型约束代替 any**

```typescript
// ❌ 错误做法
function handle<T = any>(data: T) { }

// ✅ 正确做法
interface BaseType {
  id: string
}

function handle<T extends BaseType>(data: T) { }
```

**示例 4: 判别联合类型处理错误**

```typescript
// ✅ 正确做法
type Result<T> =
  | { success: true; data: T }
  | { success: false; error: string }

function handleResult<T>(result: Result<T>) {
  if (result.success) {
    return result.data  // TypeScript 知道 data 存在
  } else {
    throw new Error(result.error)  // TypeScript 知道 error 存在
  }
}
```

#### ⚠️ 异常场景 (必须先咨询用户)

**仅在以下极少数情况**才可能考虑类型断言 (但必须先咨询用户):

1. **第三方库缺少类型定义**
   - 库没有正确的类型定义
   - 行动: 停止并咨询用户
   - 选项: 贡献类型定义或使用替代库

2. **复杂动态类型场景**
   - 正确的类型需要大规模架构改动
   - 行动: 停止并咨询用户
   - 选项: 评估重构成本 vs 技术债

3. **TypeScript 类型系统限制**
   - 确实是 TypeScript 无法表达的合法场景
   - 行动: 停止并咨询用户
   - 要求: 确认确实是 TypeScript 限制,添加详细注释

#### 🔴 遇到复杂类型错误的协议

如果类型错误复杂或难以理解:

1. **立即停止** - 不要继续尝试修复
2. **咨询用户** - 提供详细的错误上下文寻求指导
3. **绝不使用** - 不使用类型断言或 any 绕过

**处理失败**:
1. 阅读所有类型错误信息
2. 使用 Read 工具查看有类型错误的文件
3. 分析每个错误的根本原因
4. 应用正确的修复 (遵循上述严格规则)
5. 重新运行 typecheck
6. 如果出现复杂错误,停止并咨询用户
7. 迭代直到所有类型错误解决

---

**2.4 单元测试 (Unit Tests)**
```bash
pnpm test
# 或
npm run test
# 或
npx jest
# 或
npx vitest run
```

**目标**: 确保所有测试通过

**处理失败**:
1. 阅读测试失败信息
2. 使用 Read 工具查看失败的测试文件
3. 确定是测试需要更新还是代码需要修复
4. 修复根本原因
5. 重新运行测试
6. 迭代直到所有测试通过

---

#### Step 3: 处理检查失败

当任何检查失败时:

1. **分析错误** - 仔细阅读错误信息
2. **定位问题** - 使用 Read/Grep 查找有问题的代码
3. **应用正确修复** - 遵循严格规则 (特别是类型错误)
4. **重新运行失败的检查** - 确认特定问题已解决
5. **重新运行所有检查** - 如果修复引入了新代码,从步骤 1 重新开始
6. **迭代直到通过** - 不停止直到检查通过

### 为什么要按这个顺序?

```
format → lint → typecheck → test
```

**原因:**
- 格式化错误可能导致 lint 错误
- Lint 错误可能导致类型错误
- 类型错误可能导致测试失败
- 必须从基础到高级依次修复

---

#### Step 4: 完成报告

所有检查通过后,生成详细报告:

```markdown
✅ 所有代码质量检查通过

- ✓ 代码格式化 (pnpm format)
- ✓ 代码检查 (pnpm lint)
- ✓ 类型检查 (pnpm typecheck)
- ✓ 单元测试 (pnpm test)

代码已就绪,可以提交。

## 总结
- 修复的问题总数: 15
- 格式化问题: 3
- Lint 问题: 5
- 类型错误: 6
- 测试失败: 1

所有问题已按照严格的代码质量标准解决。
```

---

## 怎么使用

### 方式 1: 显式调用 Sub-agent (推荐)

```
请使用 code-quality-checker agent 执行完整的代码质量检查
```

### 方式 2: 自然语言请求

```
帮我检查代码质量
```

```
我刚完成了一个功能,帮我运行所有质量检查
```

```
准备提交代码了,先检查一下质量
```

### 方式 3: 在完成功能后

完成功能实现后,Claude 会通过 Skill 提醒你需要执行质量检查,然后自动委派给 Sub-agent。

---

## 使用例子

### 例子 1: 完成功能后的质量检查

**场景:**
用户刚完成了一个用户认证功能,准备提交代码。

**用户请求:**
```
功能已经实现完成了,帮我检查一下代码质量
```

**Sub-agent 执行流程:**

**Step 1: 初始化**
```
开始代码质量检查...

检查 package.json,发现可用的命令:
✓ pnpm format
✓ pnpm lint
✓ pnpm typecheck
✓ pnpm test

创建检查清单...
```

**Step 2.1: 代码格式化**
```bash
$ pnpm format
✓ 格式化完成,3 个文件已修改
```

**Step 2.2: 代码检查**
```bash
$ pnpm lint

❌ 发现 5 个问题:
1. src/auth/login.ts:15 - 'userName' is defined but never used
2. src/auth/login.ts:23 - Unexpected console statement
3. src/auth/utils.ts:8 - Missing return type on function
4. src/auth/utils.ts:12 - Prefer const over let
5. src/types/user.ts:5 - Trailing comma required
```

**修复过程:**
```
修复 Lint 问题...

1. 移除未使用的变量 userName
2. 移除调试用的 console.log
3. 为函数添加返回类型
4. 将 let 改为 const
5. 添加尾随逗号

重新运行 lint...
```

```bash
$ pnpm lint
✓ 所有 lint 检查通过
```

**Step 2.3: 类型检查**
```bash
$ pnpm typecheck

❌ 发现 6 个类型错误:
1. src/auth/login.ts:25 - Type 'string | undefined' is not assignable to type 'string'
2. src/auth/utils.ts:15 - Argument of type 'any' is not assignable to parameter
3. src/auth/api.ts:10 - Property 'token' does not exist on type 'Response'
4. src/auth/validator.ts:8 - Cannot find name 'User'
5. src/types/auth.ts:12 - Type assertion detected (as AuthResponse)
6. src/auth/hooks.ts:20 - Object is possibly 'undefined'
```

**修复过程 (严格遵循类型安全规则):**

```typescript
// 错误 1: Type 'string | undefined' is not assignable to type 'string'
// ❌ 禁止使用: username as string
// ✅ 正确修复: 添加类型守卫
const username = getUsername()
if (username !== undefined) {
  processUsername(username)  // TypeScript 知道这里 username 是 string
}

// 错误 2: Argument of type 'any' is not assignable
// ❌ 禁止保留 any
// ✅ 正确修复: 定义正确的类型
interface UserData {
  id: string
  email: string
}

function validateUser(data: UserData): boolean {
  return data.id !== '' && data.email.includes('@')
}

// 错误 3: Property 'token' does not exist on type 'Response'
// ❌ 禁止使用: response as AuthResponse
// ✅ 正确修复: 定义响应类型并验证
interface AuthResponse {
  token: string
  user: User
}

async function login(): Promise<AuthResponse> {
  const response = await fetch('/api/login')
  const data = await response.json()
  // 运行时验证
  if (!isAuthResponse(data)) {
    throw new Error('Invalid response format')
  }
  return data
}

function isAuthResponse(data: unknown): data is AuthResponse {
  return (
    typeof data === 'object' &&
    data !== null &&
    'token' in data &&
    'user' in data &&
    typeof (data as AuthResponse).token === 'string'
  )
}

// 错误 4: Cannot find name 'User'
// ✅ 正确修复: 导入缺失的类型
import type { User } from '@/types/user'

// 错误 5: Type assertion detected (as AuthResponse)
// ❌ 这是类型逃逸,必须移除
// ✅ 正确修复: 使用上面的类型守卫方法

// 错误 6: Object is possibly 'undefined'
// ❌ 禁止使用: user!.name
// ✅ 正确修复: 添加 null 检查
const user = users.find(u => u.id === id)
if (user) {
  console.log(user.name)  // 安全访问
}
```

**重新检查:**
```bash
$ pnpm typecheck
✓ 所有类型检查通过
```

**Step 2.4: 单元测试**
```bash
$ pnpm test

❌ 发现 1 个测试失败:
FAIL src/auth/login.test.ts
  ✓ should validate email format (5ms)
  ✕ should handle login success (12ms)
  ✓ should handle login failure (8ms)
```

**修复过程:**
```typescript
// 测试失败原因: 修改了 login 函数的返回类型
// 需要更新测试的类型断言

// 修复测试
it('should handle login success', async () => {
  const result = await login('test@example.com', 'password')

  // 更新类型断言
  expect(result.token).toBeDefined()
  expect(result.user.email).toBe('test@example.com')
})
```

**重新运行测试:**
```bash
$ pnpm test
✓ 所有测试通过 (3 tests)
```

**Step 4: 生成报告**
```markdown
✅ 所有代码质量检查通过

- ✓ 代码格式化 (pnpm format)
- ✓ 代码检查 (pnpm lint)
- ✓ 类型检查 (pnpm typecheck)
- ✓ 单元测试 (pnpm test)

代码已就绪,可以提交。

## 总结
- 修复的问题总数: 15
- 格式化问题: 3
- Lint 问题: 5
- 类型错误: 6
- 测试失败: 1

所有问题已按照严格的代码质量标准解决,未使用任何类型逃逸机制。
```

---

### 例子 2: 类型错误修复的深入示例

**场景:**
代码中有复杂的类型错误。

**类型错误:**
```typescript
// src/utils/data.ts
export function processData(data) {  // 隐式 any
  return data.items.map(item => {
    return {
      ...item,
      processed: true
    }
  })
}

// src/api/users.ts
const response = await fetch('/api/users')
const users = response.json() as User[]  // 类型断言!

// src/hooks/useAuth.ts
export function useAuth() {
  const user = getUser()
  return user!.profile  // 非空断言!
}
```

**Sub-agent 修复过程:**

```typescript
// 修复 1: 隐式 any
// ❌ 原代码
export function processData(data) { ... }

// ✅ 修复: 添加完整的类型定义
interface DataItem {
  id: string
  name: string
}

interface ProcessedDataItem extends DataItem {
  processed: boolean
}

interface DataInput {
  items: DataItem[]
}

export function processData(data: DataInput): ProcessedDataItem[] {
  return data.items.map(item => ({
    ...item,
    processed: true
  }))
}

// 修复 2: 类型断言
// ❌ 原代码
const users = response.json() as User[]

// ✅ 修复: 运行时验证 + 类型守卫
interface User {
  id: string
  name: string
  email: string
}

function isUser(obj: unknown): obj is User {
  return (
    typeof obj === 'object' &&
    obj !== null &&
    'id' in obj &&
    'name' in obj &&
    'email' in obj &&
    typeof (obj as User).id === 'string' &&
    typeof (obj as User).name === 'string' &&
    typeof (obj as User).email === 'string'
  )
}

function isUserArray(data: unknown): data is User[] {
  return Array.isArray(data) && data.every(isUser)
}

const response = await fetch('/api/users')
const data = await response.json()
if (!isUserArray(data)) {
  throw new Error('Invalid user data format')
}
const users: User[] = data  // 类型安全!

// 修复 3: 非空断言
// ❌ 原代码
return user!.profile

// ✅ 修复: 正确的 null 处理
export function useAuth() {
  const user = getUser()
  if (!user) {
    throw new Error('User not authenticated')
  }
  return user.profile  // 类型安全!
}

// 或者返回可选值
export function useAuth(): UserProfile | null {
  const user = getUser()
  return user ? user.profile : null
}
```

**重要说明:**
```
在整个修复过程中,Sub-agent 严格遵循了以下原则:
1. ✅ 没有使用任何 as 类型断言
2. ✅ 没有使用 any 类型
3. ✅ 没有使用 @ts-ignore 或 @ts-expect-error
4. ✅ 没有使用非空断言 (!)
5. ✅ 所有类型都是显式定义的
6. ✅ 使用类型守卫进行运行时验证
7. ✅ 使用 null 检查代替非空断言

结果: 真正的类型安全,不是假装的类型安全!
```

---

## 特殊情况处理

### 情况 1: 检查命令不存在

如果项目中没有某个检查命令:

```
检查 package.json...

发现:
✓ pnpm format 可用
✓ pnpm lint 可用
✗ pnpm typecheck 不存在
✓ pnpm test 可用

建议:
添加 typecheck 脚本到 package.json:
{
  "scripts": {
    "typecheck": "tsc --noEmit"
  }
}

继续执行其他检查...
```

### 情况 2: 多个检查同时失败

严格按顺序修复:

```
运行所有检查...

结果:
✗ format 失败 (3 个文件)
✗ lint 失败 (12 个问题)
✗ typecheck 失败 (8 个错误)
✗ test 失败 (2 个测试)

修复策略:
1. 先修复 format (因为可能导致 lint 错误)
2. 再修复 lint (因为可能导致类型错误)
3. 再修复 typecheck (因为可能导致测试失败)
4. 最后修复 test

开始修复 format...
```

### 情况 3: 无法修复的错误

如果遇到无法修复的错误:

```
类型错误分析:

src/legacy/adapter.ts:45 - 复杂的第三方库类型不匹配

这个错误涉及到:
- 旧版第三方库 'legacy-sdk' (无类型定义)
- 需要桥接到新的类型系统
- 可能需要重构整个适配器层

⚠️ 这超出了自动修复的范围。

建议:
1. 为 'legacy-sdk' 创建类型定义文件
2. 或考虑迁移到有类型支持的替代库
3. 或重构适配器层的架构

需要你的决策才能继续。
```

---

## 最佳实践

### 何时使用这个插件

✅ **必须使用的场景:**
- 完成功能实现或 bug 修复后
- 创建 Git commit 或 PR 之前
- 用户明确请求代码质量检查时
- 代码有大量修改需要验证时

❌ **不需要使用的场景:**
- 只修改文档 (无代码更改)
- 简单的文本编辑或格式调整
- 探索性地阅读代码时

### 充分利用检查结果

1. **理解修复而不是盲目接受**
   - Sub-agent 会解释为什么这样修复
   - 学习正确的类型安全模式

2. **建立团队规范**
   - 将这些严格标准固化为团队规范
   - 在 code review 中执行相同标准

3. **配置 Git hooks**
   - 使用 husky 在提交前自动运行检查
   - 防止不合格代码进入仓库

4. **持续改进**
   - 定期审查类型定义质量
   - 减少技术债累积

---

## 技术细节

**工具权限:**
- `Bash` - 运行检查命令
- `Read` - 读取错误文件和配置
- `Grep` - 搜索代码模式
- `Glob` - 查找文件
- `Edit` - 修复问题
- `Write` - 创建新文件 (罕见)

**使用的模型:**
- Sonnet - 平衡修复质量和速度

**独立上下文优势:**
- 多轮迭代修复不污染主对话
- 可以深入分析和修复
- 独立的 token 预算用于复杂修复

---

## 相关插件

配合使用效果更佳:

- **typescript-strict-typing** - TypeScript 严格类型约束 Skill
- **git-commit-conventions** - Git 提交规范
- **pull-request-guidelines** - PR 创建规范

---

## 常见问题

**Q: Sub-agent 会自动修改代码吗?**
A: 是的。Sub-agent 会自动修复发现的问题,这是它的核心功能。

**Q: 如果我不同意某个修复怎么办?**
A: 可以在修复后手动调整。但请注意,大部分修复都是为了真正的代码质量和类型安全。

**Q: 为什么对类型错误这么严格?**
A: 因为类型断言和 any 只是"假装"的类型安全,实际上隐藏了真实的问题,导致运行时错误。我们追求真正的类型安全。

**Q: 什么情况下可以使用类型断言?**
A: 几乎没有。即使在极少数例外场景(第三方库问题、TypeScript 限制),也必须先咨询用户并添加详细注释。

**Q: 检查需要多长时间?**
A: 取决于项目规模和问题数量。小项目通常 1-3 分钟,大项目可能需要 5-10 分钟。

**Q: 如果检查超过 5 分钟怎么办?**
A: Sub-agent 会通知你并询问是继续等待还是中断调查。

---

## 版本历史

- **v2.0.0** (当前) - 添加 Sub-agent 支持,自动化执行和修复流程
- **v1.0.0** - 初始版本,仅包含检查标准 Skill

---

## 作者和贡献

- 作者: LeekJay
- 邮箱: leekjay@foxmail.com
- 仓库: https://github.com/LeekJay/claude-skills-plugin

欢迎提交 Issue 和 PR!
