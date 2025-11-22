# Language Preferences 插件

## 插件作用

语言和交互标准插件,规范 Claude 与用户的沟通语言、技术术语处理和需求澄清流程。

**核心价值:**
- 🇨🇳 统一沟通语言 - 用户响应使用中文
- 📚 保留技术术语 - 技术术语保持原文不翻译
- 🔍 需求澄清 - 不清楚时主动询问而不是猜测
- 📖 清晰沟通 - 避免歧义,提高效率

---

## 语言使用规则

| 场景 | 语言 |
|------|------|
| **用户提问** | 中文 |
| **内部执行** | 英文 (工具命令、prompts、代码注释) |
| **用户响应** | 中文 (进度更新、最终答案、解释) |
| **代码内容** | 保持原文 (不翻译) |

---

## 绝对不能翻译的内容

**绝对不要翻译**:
- 代码标识符 (变量名、函数名、类名)
- 文件路径
- 命令和脚本
- 错误消息和日志输出
- 配置文件的 key
- API 端点
- 环境变量名
- 包名称

### 示例

**✅ 保持原文**:
```typescript
interface UserConfig {
  apiKey: string        // 不要翻译为 api密钥
  maxRetries: number    // 不要翻译为 最大重试次数
}

const API_ENDPOINT = '/api/v1/users'  // 不要翻译
```

**✅ 保持错误消息原文**:
```
Error: Cannot find module 'typescript'
TypeError: Cannot read property 'value' of undefined

❌ 不要翻译为:
错误: 无法找到模块 'typescript'
```

---

## 需求澄清流程

当需求不清楚或模糊时:

1. **识别不明确的需求**
2. **用中文提出澄清问题**
3. **等待用户反馈**
4. **基于反馈继续工作**

### 示例

```
用户: "帮我优化这个函数"

响应:
"需要更多信息来帮助优化这个函数:

1. 你主要关注哪方面?
   - 性能 (执行速度)
   - 代码可读性
   - 内存使用
   - 代码简洁性

2. 这个函数是否有性能瓶颈?

3. 有什么特定的约束条件吗?"
```

---

## 使用例子

### 例子 1: 配置项目

```
用户问: "如何配置 ESLint?"

✅ 正确响应:
"我来帮你配置 ESLint。首先安装依赖:

运行命令:
npm install --save-dev eslint @typescript-eslint/parser

然后在项目根目录创建 .eslintrc.json 配置文件..."

❌ 错误响应 (翻译了命令):
"运行 npm 安装 --保存-开发 eslint..."
```

### 例子 2: 解释错误

```
用户问: "这个错误是什么意思?"
Error: TypeError: Cannot read property 'name' of undefined

✅ 正确响应:
"这个错误表示你正在尝试访问 undefined 的 'name' 属性。

错误消息:
TypeError: Cannot read property 'name' of undefined

可能的原因:
1. 对象在访问前没有初始化
2. API 返回了 null 或 undefined

建议的修复方法:
- 使用可选链: obj?.name
- 添加 null 检查: if (obj && obj.name) { ... }"

❌ 错误响应 (翻译了错误):
"类型错误: 无法读取 undefined 的属性 'name'"
```

---

## 实践原则

### 为什么用中文与用户沟通?
- 团队主要使用中文工作
- 中文表达更准确自然
- 减少理解成本

### 为什么不翻译技术术语和代码?
- 翻译可能引入歧义或错误
- 英文术语更容易搜索解决方案
- 遵循技术领域的行业标准
- 保持错误消息原文有助于查阅文档

### 为什么需要澄清流程?
- 避免在错误方向浪费时间
- 确保最终结果符合用户期望
- 减少返工和修改

---

## 沟通示例

### 解释技术概念

```
✅ 正确:
"TypeScript 的 type guard 是一种类型守卫机制,
用于在运行时检查类型并帮助 TypeScript 推断类型。

示例:
function isString(value: unknown): value is string {
  return typeof value === 'string'
}

这样TypeScript 就知道在 if (isString(value)) 代码块中,
value 的类型是 string。"

❌ 错误 (过度翻译):
"TypeScript 的类型守卫是一种类型防护机制..."
```

### 报告进度

```
✅ 正确:
"正在执行代码质量检查...

1. ✓ 格式化完成
2. ✓ Lint 检查通过
3. 正在进行类型检查...

发现 3 个类型错误,正在修复..."

❌ 错误 (翻译命令):
"正在执行 pnpm 格式化..."
```

---

## 技术术语处理

### 保持原文的术语

- API, SDK, CLI
- HTTP, REST, GraphQL
- JSON, XML, YAML
- Git, commit, push, pull
- TypeScript, JavaScript
- npm, pnpm, yarn
- async/await, Promise
- interface, type, class
- token, OAuth, JWT

### 可以使用中文的概念

- 函数、方法、类
- 变量、常量
- 数组、对象
- 错误、警告
- 配置、设置

**原则**: 如果是专有技术术语,保持原文;如果是通用概念,可以用中文。

---

## 常见问题

**Q: 何时应该澄清需求?**
A: 当用户的请求模糊、有多种解释、或可能导致不同结果时,总是先澄清。

**Q: 错误消息为什么不能翻译?**
A: 保持原文方便搜索引擎查找解决方案,也便于查阅官方文档。

**Q: 技术术语和代码标识符有什么区别?**
A: 技术术语是通用概念(如 API),代码标识符是具体的代码中的名称(如 apiKey 变量)。两者都应保持原文。

---

## 相关插件

- **git-commit-conventions** - 提交消息使用中文主题
- **documentation-guidelines** - 文档语言规范

---

## 作者

- 作者: LeekJay
- 邮箱: leekjay@foxmail.com
- 仓库: https://github.com/LeekJay/claude-skills-plugin
