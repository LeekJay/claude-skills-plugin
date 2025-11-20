---
name: language-preferences
description: Language and interaction standards. Use when communicating with users, explaining code, reporting errors, or providing documentation. Specifies Chinese/English usage scenarios, keeping technical terms in original language, and requirement clarification process.
---

# Language and Interaction Preferences

Unified interaction language and communication standards to ensure clear and efficient collaboration.

## Language Usage Rules

| Scenario | Language |
|----------|----------|
| **User Questions** | Chinese |
| **Internal Execution** | English (tool commands, prompts, code comments) |
| **User Responses** | Chinese (progress updates, final answers, explanations) |
| **Code Artifacts** | Keep original (don't translate) |

## Content That Must Not Be Translated

**Absolutely don't translate**:
- Code identifiers (variable names, function names, class names)
- File paths
- Commands and scripts
- Error messages and log output
- Configuration file keys
- API endpoints
- Environment variable names
- Package names

**Examples**:

```typescript
// ✅ Keep original
interface UserConfig {
  apiKey: string;        // Don't translate to api密钥
  maxRetries: number;    // Don't translate to 最大重试次数
}

const API_ENDPOINT = '/api/v1/users';  // Don't translate
```

```
✅ Keep error messages original:
Error: Cannot find module 'typescript'
TypeError: Cannot read property 'value' of undefined

❌ Don't translate to:
错误: 无法找到模块 'typescript'
```

## Requirement Clarification Process

When requirements are unclear or ambiguous:

1. **Identify unclear requirements**
2. **Ask clarifying questions in Chinese**
3. **Wait for user feedback**
4. **Continue work based on feedback**

**Example**:

```
User: "Help me optimize this function"

Response:
"I need more information to help optimize this function:

1. What aspect are you mainly concerned about?
   - Performance (execution speed)
   - Code readability
   - Memory usage
   - Code conciseness

2. Does this function have performance bottlenecks?

3. Are there any specific constraints?"
```

## Examples

### Configuring Project

```
User asks: "How to configure ESLint?"

✅ Correct response:
"I'll help you configure ESLint. First install dependencies:

Run command:
npm install --save-dev eslint @typescript-eslint/parser

Then create .eslintrc.json config file in project root..."

❌ Wrong response (translated commands):
"Run npm 安装 --保存-开发 eslint..."
```

### Explaining Errors

```
User asks: "What does this error mean?"
Error: TypeError: Cannot read property 'name' of undefined

✅ Correct response:
"This error indicates you're trying to access the 'name' property of undefined.

Error message:
TypeError: Cannot read property 'name' of undefined

Possible causes:
1. Object not initialized before access
2. API returned null or undefined

Suggested fixes:
- Use optional chaining: obj?.name
- Add null check: if (obj && obj.name) { ... }"

❌ Wrong response (translated error):
"类型错误: 无法读取 undefined 的属性 'name'"
```

## Rationale

**Why communicate with users in Chinese?**
- Team primarily works in Chinese
- Chinese expressions are more accurate and natural
- Reduces understanding cost

**Why not translate technical terms and code?**
- Translation may introduce ambiguity or errors
- English terms are easier to search for solutions
- Follows industry standards in technical field
- Keeping error messages original aids documentation lookup

**Why need clarification process?**
- Avoids wasting time in wrong direction
- Ensures final result meets user expectations
- Reduces rework and modifications
