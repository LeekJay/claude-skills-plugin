# Electron WebView Debugging 插件

## 插件作用

这是一个专业的 Electron WebView 调试插件,帮助你系统化地诊断和解决 Electron 应用中 `<webview>` 标签相关的各类问题。无论是脚本注入失败、事件监听异常、DevTools 无法打开,还是 IPC 通信故障,这个插件都能提供专业的调试指导。

**核心价值:**
- 🐛 系统化调试流程 - 四步诊断法快速定位问题
- 🔒 安全最佳实践 - 强制执行 WebView 安全配置
- 📚 专业知识库 - 涵盖脚本注入、事件处理、DevTools、IPC 四大领域
- 💡 实用解决方案 - 提供可执行的修复步骤和代码示例

---

## 运作方式

### 架构组成

插件采用 **Skill + Sub-agent 混合架构**:

```
用户遇到 WebView 问题
    ↓
Skill 自动激活 (提供调试知识和安全建议)
    ↓
Claude 决定委派给 Sub-agent
    ↓
Sub-agent 在独立上下文中执行系统化调试:
  - 识别问题类型
  - 检查配置
  - 针对性调试
  - 验证修复
    ↓
返回诊断结果和修复方案
```

### 组件说明

**1. Skill (技能)**
- 位置: `skills/electron-webview-debugging/SKILL.md`
- 作用: Claude 自动发现,提供 WebView 调试背景知识
- 触发: 当用户提到 "webview"、"Electron 调试" 等关键词时自动激活

**2. Sub-agent (子代理)**
- 位置: `agents/electron-webview-debugging.md`
- 作用: 在独立上下文中执行完整的调试流程
- 触发: 用户显式请求或 Claude 自动委派
- 优势: 系统化的多步骤调试,避免主对话混乱

**3. 详细调试指南**
- `SCRIPT_INJECTION.md` - 脚本注入调试详细指南
- `EVENT_HANDLING.md` - 事件处理调试详细指南
- `DEVTOOLS.md` - DevTools 使用详细指南
- `IPC_DEBUGGING.md` - IPC 通信调试详细指南

---

## 工作方式

### 五大问题类别

Sub-agent 能诊断和解决以下五类 WebView 问题:

#### 1. 配置问题 (Configuration Issues)
- ✓ WebView 标签属性配置错误
- ✓ preload 脚本路径不正确
- ✓ webpreferences 安全设置问题
- ✓ partition 会话设置错误

#### 2. 脚本注入问题 (Script Injection Issues)
- ✓ executeJavaScript() 注入失败
- ✓ preload 脚本未执行
- ✓ insertCSS() 样式注入问题
- ✓ 脚本执行时机错误

#### 3. 事件监听问题 (Event Listening Issues)
- ✓ 生命周期事件未触发
- ✓ console-message 事件捕获失败
- ✓ 错误和崩溃检测问题
- ✓ 导航事件处理异常

#### 4. DevTools 问题 (DevTools Issues)
- ✓ 无法打开 DevTools
- ✓ DevTools API 使用问题
- ✓ 远程调试配置错误
- ✓ 调试工具集成失败

#### 5. IPC 通信问题 (IPC Communication Issues)
- ✓ 主进程与 WebView 通信失败
- ✓ preload 脚本中的 IPC 桥接问题
- ✓ 消息发送但未接收
- ✓ contextBridge 配置错误

### 系统化调试流程

**Step 1: 识别问题类型**
- 询问用户问题表现
- 将问题归类到五大类别之一
- 如果类别不明确,提出澄清问题

**Step 2: 检查 WebView 配置**
1. 使用 Grep 搜索 `<webview>` 标签定义
2. 使用 Read 读取包含 WebView 的文件
3. 检查关键属性:
   - `src` - URL 是否正确
   - `preload` - 脚本路径是否正确
   - `nodeintegration` - 是否启用 (安全风险!)
   - `webpreferences` - 安全设置是否合理
   - `partition` - 会话隔离配置

**Step 3: 针对性调试**

根据问题类型执行相应的调试策略:

- **配置问题** → 验证路径、修复配置
- **脚本注入** → 检查加载时机、验证语法、查看 console
- **事件监听** → 验证事件绑定、检查事件名、添加日志
- **DevTools** → 尝试 API 调用、检查安全设置
- **IPC 通信** → 添加日志、验证 channel 名、检查 contextBridge

**Step 4: 验证修复**
- 引导用户验证 WebView 是否正常加载
- 检查脚本是否成功注入
- 确认事件监听器正常工作
- 验证 DevTools 可以打开
- 测试 IPC 双向通信
- 检查控制台无错误

---

## 怎么使用

### 方式 1: 显式调用 Sub-agent (推荐用于复杂问题)

```
请使用 electron-webview-debugging agent 帮我调试 WebView 加载失败的问题
```

或者指定问题类型:

```
请使用 electron-webview-debugging agent 调试 WebView 的脚本注入问题
```

### 方式 2: 自然语言描述问题

```
我的 Electron 应用中 WebView 一直是空白页面,怎么调试?
```

```
WebView 中的 preload 脚本好像没有执行,如何排查?
```

```
主进程无法向 WebView 发送消息,IPC 通信有问题
```

### 方式 3: 请求安全审查

```
帮我检查一下 WebView 的安全配置是否合理
```

```
我启用了 nodeintegration,会有安全风险吗?
```

---

## 使用例子

### 例子 1: WebView 空白页面调试

**用户请求:**
```
我的 Electron 应用中 WebView 一直显示空白,看不到任何内容,怎么排查?
```

**Sub-agent 调试流程:**

**Step 1: 识别问题类型**
```
这可能是配置问题或加载失败。让我先检查你的 WebView 配置。
```

**Step 2: 检查配置**
```bash
# 搜索 WebView 标签
grep -r "<webview" --include="*.html" --include="*.js" --include="*.ts"
```

发现文件: `src/renderer/index.html`

**读取文件:**
```html
<webview
  id="myWebview"
  src="https://example.com"
  preload="./preload.js"
></webview>
```

**Step 3: 诊断**
```
发现问题:
1. preload 路径使用了相对路径 "./preload.js"
2. 缺少 webpreferences 安全配置
3. 未设置 partition

建议修复:
1. 使用绝对路径或正确的相对路径
2. 添加必要的 webpreferences
3. 添加事件监听以便调试
```

**Step 4: 提供修复方案**
```html
<!-- 修复后的配置 -->
<webview
  id="myWebview"
  src="https://example.com"
  preload="file://${__dirname}/preload.js"
  partition="persist:myapp"
  webpreferences="contextIsolation=yes, nodeIntegration=no, sandbox=yes"
></webview>

<script>
const webview = document.getElementById('myWebview');

// 添加事件监听以便调试
webview.addEventListener('did-start-loading', () => {
  console.log('WebView 开始加载');
});

webview.addEventListener('did-finish-load', () => {
  console.log('WebView 加载完成');
});

webview.addEventListener('did-fail-load', (e) => {
  console.error('WebView 加载失败:', e.errorDescription);
});

webview.addEventListener('console-message', (e) => {
  console.log('WebView Console:', e.message);
});
</script>
```

**验证清单:**
- [ ] WebView 加载目标 URL 成功
- [ ] 控制台输出 "WebView 加载完成"
- [ ] 无加载失败错误
- [ ] 页面内容正常显示

---

### 例子 2: 脚本注入失败调试

**用户请求:**
```
我想在 WebView 中注入一段 JavaScript,但是 executeJavaScript 没有效果
```

**Sub-agent 调试流程:**

**Step 1: 识别问题类型**
```
这是脚本注入问题。让我检查你的注入代码和时机。
```

**Step 2: 检查注入代码**
```javascript
// 用户的代码
const webview = document.getElementById('myWebview');
webview.executeJavaScript('console.log("Hello from WebView")');
```

**Step 3: 诊断问题**
```
问题分析:
1. 脚本在 WebView 加载完成前执行 → 时机错误
2. 未检查执行结果
3. 未监听 console-message 事件

常见原因:
- WebView DOM 元素还未准备好
- WebView 页面还未加载完成
- 脚本语法错误但未捕获
```

**Step 4: 提供正确方案**
```javascript
const webview = document.getElementById('myWebview');

// ✅ 正确做法: 等待 WebView 加载完成
webview.addEventListener('did-finish-load', () => {
  // 注入脚本并处理结果
  webview.executeJavaScript('console.log("Hello from WebView")')
    .then(result => {
      console.log('脚本执行成功:', result);
    })
    .catch(error => {
      console.error('脚本执行失败:', error);
    });
});

// 监听 WebView 控制台输出
webview.addEventListener('console-message', (e) => {
  console.log(`[WebView Console] ${e.level}: ${e.message}`);
});

// 注入更复杂的脚本示例
webview.addEventListener('did-finish-load', () => {
  const script = `
    (function() {
      document.body.style.backgroundColor = 'lightblue';
      return document.title;
    })()
  `;

  webview.executeJavaScript(script)
    .then(title => {
      console.log('页面标题:', title);
    });
});
```

**验证:**
- [ ] 在 did-finish-load 事件后注入脚本
- [ ] 脚本成功执行
- [ ] 可以在控制台看到 WebView 的输出
- [ ] 能够获取脚本返回值

---

### 例子 3: IPC 通信调试

**用户请求:**
```
主进程无法向 WebView 发送消息,IPC 通信不工作
```

**Sub-agent 调试流程:**

**Step 1: 识别问题类型**
```
这是 IPC 通信问题。让我检查你的 preload 脚本和通信代码。
```

**Step 2: 检查 preload 脚本配置**
```html
<webview
  id="myWebview"
  src="https://example.com"
  preload="./preload.js"
  webpreferences="contextIsolation=yes, nodeIntegration=no"
></webview>
```

**Step 3: 检查 preload 脚本**
```javascript
// preload.js (有问题的版本)
const { ipcRenderer } = require('electron');

// ❌ 错误: contextIsolation=yes 时,这样无法暴露给页面
window.sendToMain = (channel, data) => {
  ipcRenderer.send(channel, data);
};
```

**Step 4: 提供正确方案**

**修复后的 preload.js:**
```javascript
const { contextBridge, ipcRenderer } = require('electron');

// ✅ 正确: 使用 contextBridge 安全暴露 API
contextBridge.exposeInMainWorld('electronAPI', {
  // 发送消息到主进程
  sendToMain: (channel, data) => {
    // 白名单验证 channel
    const validChannels = ['toMain', 'webview-message'];
    if (validChannels.includes(channel)) {
      ipcRenderer.send(channel, data);
    }
  },

  // 接收来自主进程的消息
  onMainMessage: (callback) => {
    ipcRenderer.on('fromMain', (event, ...args) => {
      callback(...args);
    });
  },

  // 移除监听器
  removeListener: (channel) => {
    ipcRenderer.removeAllListeners(channel);
  }
});
```

**WebView 页面中使用:**
```javascript
// WebView 内部的页面代码
// 发送消息到主进程
window.electronAPI.sendToMain('toMain', { message: 'Hello from WebView' });

// 接收来自主进程的消息
window.electronAPI.onMainMessage((data) => {
  console.log('收到主进程消息:', data);
});
```

**主进程代码:**
```javascript
const { ipcMain, webContents } = require('electron');

// 接收来自 WebView 的消息
ipcMain.on('toMain', (event, data) => {
  console.log('收到 WebView 消息:', data);

  // 回复 WebView
  event.sender.send('fromMain', { reply: 'Hello from Main' });
});
```

**调试技巧:**
```javascript
// 在 preload.js 中添加日志
console.log('Preload script loaded');

// 在 WebView 页面中检查 API 是否可用
console.log('electronAPI available:', typeof window.electronAPI);

// 在主进程中列出所有 webContents
webContents.getAllWebContents().forEach((wc, index) => {
  console.log(`WebContent ${index}:`, wc.getURL());
});
```

**验证清单:**
- [ ] preload 脚本正确加载
- [ ] contextBridge 成功暴露 API
- [ ] WebView 页面能调用 electronAPI
- [ ] 主进程能接收 WebView 消息
- [ ] WebView 能接收主进程消息
- [ ] IPC 双向通信正常

---

### 例子 4: 安全配置审查

**用户请求:**
```
帮我检查 WebView 的安全配置是否合理
```

**Sub-agent 审查流程:**

**读取配置:**
```html
<webview
  id="myWebview"
  src="https://external-site.com"
  nodeintegration="true"
  webpreferences="contextIsolation=no"
></webview>
```

**安全问题诊断:**
```
🚨 发现严重安全问题:

1. ❌ nodeintegration="true"
   - 风险等级: 严重
   - 问题: 将 Node.js API 暴露给远程内容
   - 影响: 远程网站可以执行任意 Node.js 代码,访问文件系统

2. ❌ contextIsolation=no
   - 风险等级: 高
   - 问题: preload 脚本和页面内容共享上下文
   - 影响: 页面可以访问 preload 中的所有变量和方法

3. ❌ 未启用 sandbox
   - 风险等级: 中
   - 问题: 未启用沙箱隔离
   - 影响: 降低进程隔离级别

4. ❌ 加载外部网站
   - 风险等级: 高 (结合 nodeintegration)
   - 问题: 外部网站不可信
   - 影响: 恶意代码可能利用 Node.js 权限
```

**推荐的安全配置:**
```html
✅ 安全配置示例:
<webview
  id="myWebview"
  src="https://external-site.com"
  preload="./secure-preload.js"
  partition="persist:myapp"
  webpreferences="contextIsolation=yes, nodeIntegration=no, sandbox=yes"
  disablewebsecurity="false"
></webview>
```

**安全最佳实践清单:**
- [ ] nodeintegration=no (除非绝对必要)
- [ ] contextIsolation=yes (总是启用)
- [ ] sandbox=yes (启用沙箱)
- [ ] 使用 contextBridge 暴露有限 API
- [ ] 验证 IPC 消息来源
- [ ] 使用 CSP (Content Security Policy)
- [ ] 不信任来自 WebView 的数据
- [ ] 使用 partition 隔离会话

---

## 常见陷阱和解决方案

### 陷阱 1: 路径问题
```javascript
// ❌ 错误
preload="./preload.js"  // 相对路径可能解析失败

// ✅ 正确
preload="${path.join(__dirname, 'preload.js')}"  // 绝对路径
```

### 陷阱 2: 时机问题
```javascript
// ❌ 错误: WebView 还未加载就注入
webview.executeJavaScript('...');

// ✅ 正确: 等待加载完成
webview.addEventListener('did-finish-load', () => {
  webview.executeJavaScript('...');
});
```

### 陷阱 3: 事件绑定问题
```javascript
// ❌ 错误: WebView 元素还不存在
const webview = document.getElementById('myWebview');
webview.addEventListener('did-finish-load', ...);  // null reference

// ✅ 正确: 等待 DOM 就绪
document.addEventListener('DOMContentLoaded', () => {
  const webview = document.getElementById('myWebview');
  webview.addEventListener('did-finish-load', ...);
});
```

### 陷阱 4: 安全配置
```html
<!-- ❌ 危险配置 -->
<webview nodeintegration="true" src="https://untrusted.com"></webview>

<!-- ✅ 安全配置 -->
<webview
  webpreferences="contextIsolation=yes, nodeIntegration=no, sandbox=yes"
  src="https://untrusted.com">
</webview>
```

---

## 推荐工具

Sub-agent 会根据情况推荐使用:

- **Electron DevTools Extension** - 增强开发工具
- **electron-devtools-installer** - 安装 Chrome DevTools 扩展
- **electron-debug** - 简化 Electron 应用调试
- **devtron** (已废弃,但可参考) - Electron 专用 DevTools

---

## 参考文档

Sub-agent 会在适当时候引导用户查看:

- [Electron 官方文档 - WebView Tag](https://www.electronjs.org/docs/latest/api/webview-tag)
- [Electron 安全指南](https://www.electronjs.org/docs/latest/tutorial/security)
- [Context Isolation](https://www.electronjs.org/docs/latest/tutorial/context-isolation)
- [IPC 通信](https://www.electronjs.org/docs/latest/tutorial/ipc)

---

## 最佳实践

### 何时使用这个插件

✅ **推荐使用的场景:**
- WebView 加载失败或空白
- 脚本注入不生效
- 事件监听器未触发
- DevTools 无法打开
- IPC 通信失败
- 需要审查安全配置
- 首次集成 WebView 需要指导

❌ **不适合的场景:**
- 主进程代码调试 (用 Node.js 调试工具)
- BrowserWindow 调试 (用常规 DevTools)
- Electron 打包问题
- 纯 Web 应用调试 (非 Electron)

---

## 技术细节

**工具权限:**
- `Read` - 读取配置文件和代码
- `Grep` - 搜索 WebView 标签和配置
- `Glob` - 查找相关文件
- `Bash` - 执行调试命令

**使用的模型:**
- Sonnet - 平衡调试深度和响应速度

**独立上下文优势:**
- 多步骤调试不污染主对话
- 可以深入递归分析
- 独立的 token 预算

---

## 常见问题

**Q: Sub-agent 会修改我的代码吗？**
A: 不会自动修改。Sub-agent 会提供修复建议和示例代码,你需要手动应用。

**Q: 如何确认 preload 脚本是否加载？**
A: 在 preload.js 开头添加 `console.log('Preload loaded')`,然后监听 console-message 事件。

**Q: nodeintegration 在什么情况下可以启用？**
A: 仅在加载完全可信的本地内容时。如果 WebView 加载任何外部或用户提供的内容,绝对不要启用。

**Q: contextIsolation 必须启用吗？**
A: 是的。contextIsolation=yes 是现代 Electron 安全的基础,应该总是启用。

**Q: 如何调试 WebView 内部的页面？**
A: 使用 `webview.openDevTools()` 打开 WebView 的 DevTools,或者监听 console-message 事件。

---

## 版本历史

- **v2.0.0** (当前) - 添加 Sub-agent 支持,系统化调试流程
- **v1.0.0** - 初始版本,仅包含 Skill

---

## 作者和贡献

- 作者: LeekJay
- 邮箱: leekjay@foxmail.com
- 仓库: https://github.com/LeekJay/claude-skills-plugin

欢迎提交 Issue 和 PR!
