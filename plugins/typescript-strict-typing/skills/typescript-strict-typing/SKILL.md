---
name: typescript-strict-typing
description: Enforce strict TypeScript type safety standards for ALL code writing scenarios. Prohibits any, type assertions, @ts-ignore, @ts-expect-error, ESLint disable comments (eslint-disable-next-line @typescript-eslint/no-explicit-any, etc.), non-null assertions, implicit any, and other type-unsafe patterns. Use this skill when writing ANY TypeScript code, fixing type errors, implementing features, or refactoring.
allowed-tools: []
---

# TypeScript Strict Typing Standards

**CRITICAL**: This skill applies to **ALL TypeScript code writing scenarios** - not just when fixing type errors.

## When to Use This Skill

✅ **ALWAYS use when**:
- Writing any TypeScript code
- Implementing new features
- Fixing bugs
- Refactoring code
- Modifying existing code
- Adding new functions, classes, or modules
- Working with third-party libraries
- Handling async operations
- Processing data structures

## Absolutely Prohibited Patterns

### ❌ 1. Type Escape Mechanisms

**Never use these under any circumstances:**

```typescript
// ❌ WRONG - any type
const data: any = fetchData()

// ❌ WRONG - type assertion
const element = document.getElementById('id') as HTMLInputElement

// ❌ WRONG - double assertion
const value = something as unknown as SomeType

// ❌ WRONG - TypeScript suppression comments
// @ts-ignore
// @ts-expect-error

// ❌ WRONG - ESLint suppression comments for type rules
// eslint-disable-next-line @typescript-eslint/no-explicit-any
// eslint-disable-next-line @typescript-eslint/ban-types
// eslint-disable-next-line @typescript-eslint/no-unsafe-assignment
// eslint-disable-next-line @typescript-eslint/no-unsafe-member-access
// eslint-disable-next-line @typescript-eslint/no-unsafe-call
// eslint-disable-next-line @typescript-eslint/no-unsafe-return
/* eslint-disable @typescript-eslint/no-explicit-any */

// ❌ WRONG - ESLint config files that disable type safety rules
// .eslintrc.js with rules turned off
```

### ❌ 2. Non-null Assertion Operator (!)

```typescript
// ❌ WRONG - non-null assertion
const value = array.find(x => x.id === id)!.name

// ❌ WRONG - property access assertion
const result = obj!.property

// ✅ CORRECT - proper null checking
const item = array.find(x => x.id === id)
if (item) {
  const value = item.name
}
```

### ❌ 3. Implicit Any

```typescript
// ❌ WRONG - implicit any parameters
function process(data) {
  return data.value
}

// ❌ WRONG - implicit any return
function getData() {
  return fetchSomething()
}

// ✅ CORRECT - explicit types
function process(data: DataType): string {
  return data.value
}

function getData(): Promise<DataType> {
  return fetchSomething()
}
```

### ❌ 4. Unsafe Generic Types

```typescript
// ❌ WRONG - Object type
function process(obj: Object) { }

// ❌ WRONG - Function type
function execute(fn: Function) { }

// ❌ WRONG - generic any constraint
function handle<T = any>(data: T) { }

// ✅ CORRECT - specific types
function process(obj: Record<string, unknown>) { }

function execute(fn: () => void) { }

function handle<T extends SpecificType>(data: T) { }
```

### ❌ 5. Unsafe Array/Object Operations

```typescript
// ❌ WRONG - array access without checking
const first = array[0].property

// ❌ WRONG - find without null check
const user = users.find(u => u.id === id).name

// ❌ WRONG - bracket notation without type guard
const value = obj['dynamicKey']

// ✅ CORRECT - safe access
const first = array[0]?.property
if (first !== undefined) {
  // use first
}

const user = users.find(u => u.id === id)
if (user) {
  const name = user.name
}

// Use proper typing for dynamic keys
const value = obj[key as keyof typeof obj]
```

### ❌ 6. Unsafe Error Handling

```typescript
// ❌ WRONG - any in catch block
try {
  await operation()
} catch (error) {
  console.log(error.message)
}

// ✅ CORRECT - proper error typing
try {
  await operation()
} catch (error) {
  if (error instanceof Error) {
    console.log(error.message)
  } else {
    console.log('Unknown error:', error)
  }
}
```

### ❌ 7. Missing Function Type Annotations

```typescript
// ❌ WRONG - no parameter types
function calculate(a, b) {
  return a + b
}

// ❌ WRONG - no return type
function fetchUser(id: string) {
  return fetch(`/api/users/${id}`)
}

// ✅ CORRECT - full type annotations
function calculate(a: number, b: number): number {
  return a + b
}

function fetchUser(id: string): Promise<User> {
  return fetch(`/api/users/${id}`).then(res => res.json())
}
```

### ❌ 8. Unsafe Object.keys() Usage

```typescript
interface User {
  name: string
  age: number
}

const user: User = { name: 'Alice', age: 30 }

// ❌ WRONG - assumes keys are type-safe
Object.keys(user).forEach(key => {
  console.log(user[key]) // Type error!
})

// ✅ CORRECT - proper type handling
Object.keys(user).forEach((key) => {
  const typedKey = key as keyof User
  console.log(user[typedKey])
})

// ✅ BETTER - use type-safe alternative
(Object.keys(user) as Array<keyof User>).forEach(key => {
  console.log(user[key])
})
```

## Mandatory Approach for Type Safety

### ✅ 1. Explicit Type Annotations

Always declare types explicitly for:
- Function parameters
- Function return values
- Complex variables that cannot be inferred
- Object properties
- Class properties

### ✅ 2. Type Guards

Use type guards for runtime type checking:

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
```

### ✅ 3. Proper Null/Undefined Handling

```typescript
// Use optional chaining
const value = obj?.property?.nested

// Use nullish coalescing
const result = value ?? defaultValue

// Use type narrowing
if (value !== null && value !== undefined) {
  // value is now narrowed
}
```

### ✅ 4. Discriminated Unions

```typescript
type Result<T> =
  | { success: true; data: T }
  | { success: false; error: string }

function handleResult<T>(result: Result<T>) {
  if (result.success) {
    return result.data
  } else {
    throw new Error(result.error)
  }
}
```

### ✅ 5. Generic Constraints

```typescript
function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key]
}

function processArray<T extends { id: string }>(items: T[]): string[] {
  return items.map(item => item.id)
}
```

### ✅ 6. Proper Third-Party Library Types

```typescript
// Install @types packages
// npm install --save-dev @types/node @types/express

// If no types available, create declaration file
declare module 'untyped-library' {
  export function someFunction(param: string): number
}
```

## Exception Handling (RARE Cases Only)

### ⚠️ When Type Assertion May Be Acceptable

Only in these extremely rare cases (must consult user first):

1. **TypeScript Compiler Bug**: Confirmed TypeScript limitation with GitHub issue reference
2. **Impossible Type Narrowing**: When TypeScript cannot narrow types despite runtime guarantees
3. **Legacy Code Migration**: Temporary bridge during large-scale migration (must have removal plan)

**Requirements for exceptions:**
- Must add detailed comment explaining why
- Must link to TypeScript issue or migration plan
- Must have user approval
- Must plan to remove in future

```typescript
// Acceptable exception example:
// TODO: Remove assertion after TypeScript 5.x upgrade
// See: https://github.com/microsoft/TypeScript/issues/xxxxx
// This is a known TypeScript limitation where...
const value = something as SpecificType
```

## Enforcement Rules

### 🔴 When You Encounter Type Errors

1. **STOP** - Do not use any, assertions, or @ts-ignore
2. **ANALYZE** - Understand the root cause
3. **FIX PROPERLY** - Apply correct typing solution
4. **CONSULT** - If truly complex, ask user for guidance

### 🔴 When Writing New Code

1. **TYPE FIRST** - Define types before implementation
2. **ANNOTATE EXPLICITLY** - Don't rely solely on inference
3. **VALIDATE RUNTIME** - Use type guards for external data
4. **TEST TYPES** - Ensure TypeScript catches errors

### 🔴 When Refactoring

1. **IMPROVE TYPES** - Don't preserve existing bad patterns
2. **REMOVE ASSERTIONS** - Replace with proper types
3. **ADD MISSING ANNOTATIONS** - Complete partial types

## Rationale

### Why These Rules Matter

- **Runtime Safety**: Type assertions hide bugs that crash at runtime
- **Maintainability**: Proper types document code behavior
- **Refactoring**: Type-safe code can be refactored confidently
- **Team Collaboration**: Clear types prevent misunderstandings
- **IDE Support**: Proper types enable autocomplete and navigation
- **Technical Debt**: Type shortcuts accumulate and compound over time

## Quick Reference Checklist

Before writing any TypeScript code, verify:

- [ ] No `any` types
- [ ] No `as` type assertions
- [ ] No `@ts-ignore` or `@ts-expect-error`
- [ ] No `!` non-null assertions
- [ ] All function parameters have types
- [ ] All functions have return types
- [ ] Null/undefined handled properly
- [ ] Array operations checked for undefined
- [ ] Error handling properly typed
- [ ] Generic constraints are specific
- [ ] Third-party libraries have types

## Commands for Type Checking

```bash
# Check for type errors
pnpm typecheck
# or
tsc --noEmit

# Enable strict mode in tsconfig.json
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

## Remember

**Type safety is not optional. It's not about passing the compiler - it's about writing correct, maintainable, production-ready code.**

If you're tempted to use `any`, an assertion, or `@ts-ignore`, you're doing it wrong. Stop and find the proper solution.
