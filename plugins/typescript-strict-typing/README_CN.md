# TypeScript Strict Typing 插件

## 插件作用

强制执行严格的 TypeScript 类型安全标准,禁止所有类型逃逸机制。确保代码具有真正的类型安全,而不是"假装"的类型安全。

**核心价值:**
- 🛡️ 真正的类型安全 - 禁止 any、类型断言等逃逸机制
- 🐛 减少运行时错误 - 类型系统在编译时捕获问题
- 📚 更好的文档 - 类型即文档
- 🔧 更好的 IDE 支持 - 自动完成和类型提示

---

## 核心原则

**ALL TypeScript 代码编写场景都适用**,不仅仅是修复类型错误时!

---

## 绝对禁止的模式

### ❌ 1. 类型逃逸机制

```typescript
// ❌ 错误 - any 类型
const data: any = fetchData()

// ❌ 错误 - 类型断言
const element = document.getElementById('id') as HTMLInputElement
const value = something as unknown as SomeType

// ❌ 错误 - TypeScript 抑制注释
// @ts-ignore
// @ts-expect-error

// ❌ 错误 - ESLint 抑制注释 (针对类型规则)
// eslint-disable-next-line @typescript-eslint/no-explicit-any
/* eslint-disable @typescript-eslint/... */
```

### ❌ 2. 非空断言操作符 (!)

```typescript
// ❌ 错误
const value = array.find(x => x.id === id)!.name
const result = obj!.property

// ✅ 正确 - 添加 null 检查
const item = array.find(x => x.id === id)
if (item) {
  const value = item.name
}
```

### ❌ 3. 隐式 Any

```typescript
// ❌ 错误
function process(data) {  // 隐式 any
  return data.value
}

// ✅ 正确
interface DataType {
  value: string
}

function process(data: DataType): string {
  return data.value
}
```

---

## 强制要求的做法

### ✅ 1. 显式类型注解

所有函数参数、返回值、复杂变量都必须有类型:

```typescript
// ✅ 正确
function calculate(a: number, b: number): number {
  return a + b
}

async function fetchUser(id: string): Promise<User> {
  const response = await fetch(`/api/users/${id}`)
  return response.json()
}
```

### ✅ 2. 类型守卫

使用类型守卫进行运行时类型检查:

```typescript
function isString(value: unknown): value is string {
  return typeof value === 'string'
}

function isUser(obj: unknown): obj is User {
  return (
    typeof obj === 'object' &&
    obj !== null &&
    'name' in obj &&
    'age' in obj
  )
}

// 使用
if (isUser(data)) {
  console.log(data.name)  // TypeScript 知道 data 是 User
}
```

### ✅ 3. Null/Undefined 处理

```typescript
// 使用可选链
const value = obj?.property?.nested

// 使用空值合并
const result = value ?? defaultValue

// 使用类型收窄
if (value !== null && value !== undefined) {
  // value 现在被收窄了
}
```

### ✅ 4. 判别联合类型

```typescript
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

### ✅ 5. 泛型约束

```typescript
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key]
}

function processArray<T extends { id: string }>(items: T[]): string[] {
  return items.map(item => item.id)
}
```

---

## 使用场景

✅ **总是使用这个 Skill:**
- 编写任何 TypeScript 代码时
- 实现新功能时
- 修复 bug 时
- 重构代码时
- 修改现有代码时
- 处理第三方库时
- 处理异步操作时

---

## 异常场景 (极少数)

仅在以下情况可能考虑例外 (必须先咨询用户):

1. **第三方库缺少类型定义**
2. **复杂动态类型场景** (需要大规模架构改动)
3. **TypeScript 类型系统限制** (确实无法表达的场景)

**要求:**
- 必须咨询用户
- 必须添加详细注释
- 必须有移除计划

---

## 快速检查清单

编写 TypeScript 代码前验证:

- [ ] 无 `any` 类型
- [ ] 无 `as` 类型断言
- [ ] 无 `@ts-ignore` 或 `@ts-expect-error`
- [ ] 无 `!` 非空断言
- [ ] 所有函数参数有类型
- [ ] 所有函数有返回类型
- [ ] Null/undefined 正确处理
- [ ] 数组操作检查 undefined
- [ ] 错误处理正确类型化
- [ ] 泛型约束具体
- [ ] 第三方库有类型

---

## tsconfig.json 推荐配置

```json
{
  "compilerOptions": {
    "strict": true,
    "noImplicitAny": true,
    "strictNullChecks": true,
    "strictFunctionTypes": true,
    "strictBindCallApply": true,
    "strictPropertyInitialization": true,
    "noImplicitThis": true,
    "alwaysStrict": true
  }
}
```

---

## 常见问题

**Q: 为什么这么严格?**
A: 类型断言和 any 隐藏真实问题,导致运行时错误。真正的类型安全能在编译时捕获 bug。

**Q: 什么时候可以使用类型断言?**
A: 几乎没有。即使在例外场景也必须咨询用户并详细注释。

**Q: 如何处理第三方库的类型问题?**
A: 1) 安装 @types 包 2) 创建类型声明文件 3) 咨询用户是否使用替代库

---

## 相关插件

- **code-quality-standards** - 包含类型检查流程

---

## 作者

- 作者: LeekJay
- 邮箱: leekjay@foxmail.com
- 仓库: https://github.com/LeekJay/claude-skills-plugin
