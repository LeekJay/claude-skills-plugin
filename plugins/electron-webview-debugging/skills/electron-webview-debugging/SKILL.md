---
name: electron-webview-debugging
description: Debug and troubleshoot Electron WebView Tag with script injection, event monitoring, DevTools integration, and IPC communication debugging capabilities.
allowed-tools: Read, Grep, Glob, Bash
---

# Electron WebView Tag Debugging Skill

## When to Use This Skill

### ✅ Required Usage

- User needs to debug Electron `<webview>` Tag
- Need to inject scripts or styles into WebView
- Need to monitor WebView lifecycle events
- Need to use DevTools for debugging WebView content
- Need to debug IPC communication between main process and WebView
- WebView experiencing load failures, blank screens, or crashes
- Need to check WebView configuration and permission settings

### ❌ Not Required

- Debugging main process code (use regular Node.js debugging tools)
- Debugging renderer process windows (non-WebView, use BrowserWindow DevTools)
- Electron packaging and build issues
- Pure web application debugging (non-Electron environment)

## WebView Debugging Workflow

### Step 1: Identify Problem Type

First determine which category the problem belongs to:

1. **Configuration Issues** - WebView tag attribute misconfiguration
2. **Script Injection Issues** - preload, executeJavaScript injection failures
3. **Event Listening Issues** - Events not triggering or listening properly
4. **DevTools Issues** - Unable to open or use DevTools
5. **IPC Communication Issues** - Communication failures between main process and WebView

### Step 2: Check WebView Configuration

Use Read tool to read files containing `<webview>` tag:

```bash
# Search for WebView tag definitions
grep -r "<webview" --include="*.html" --include="*.js" --include="*.jsx" --include="*.ts" --include="*.tsx"
```

Check these key attributes:
- `src` - Is the loaded URL correct
- `preload` - Is the preload script path correct
- `nodeintegration` - Whether Node.js integration is needed (security risk!)
- `webpreferences` - Web preference settings
- `partition` - Session partition settings

### Step 3: Debug Based on Problem Type

According to the problem type, refer to the corresponding detailed documentation:

#### Script Injection Related
→ See [SCRIPT_INJECTION.md](./SCRIPT_INJECTION.md)
- How to use `executeJavaScript()` to inject scripts
- How to configure preload scripts
- How to use `insertCSS()` to inject styles
- Security considerations for script injection

#### Event Listening Related
→ See [EVENT_HANDLING.md](./EVENT_HANDLING.md)
- WebView lifecycle event listening
- console-message event capture
- Error handling and crash detection
- Navigation event handling

#### DevTools Debugging Related
→ See [DEVTOOLS.md](./DEVTOOLS.md)
- How to open WebView DevTools
- How to use DevTools API
- Remote debugging configuration
- Debugging tips and best practices

#### IPC Communication Debugging Related
→ See [IPC_DEBUGGING.md](./IPC_DEBUGGING.md)
- ipcRenderer/ipcMain communication debugging
- IPC bridging in preload scripts
- Message tracking and logging
- Common IPC communication troubleshooting

### Step 4: Verify Fix

After debugging and fixing, verify the following:

- [ ] WebView loads target URL successfully
- [ ] Scripts are injected and executed successfully
- [ ] Event listeners work properly
- [ ] DevTools can be opened and used
- [ ] IPC communication is bidirectional
- [ ] No errors in console
- [ ] WebView performance is normal

## Common Tools and APIs

### WebView Tag Attributes
```html
<webview
  id="myWebview"
  src="https://example.com"
  preload="./preload.js"
  partition="persist:myapp"
  webpreferences="contextIsolation=yes, nodeIntegration=no"
></webview>
```

### JavaScript API
```javascript
const webview = document.getElementById('myWebview');

// Execute script
webview.executeJavaScript('console.log("Hello from WebView")');

// Open DevTools
webview.openDevTools();

// Listen to events
webview.addEventListener('did-finish-load', () => {
  console.log('WebView loaded');
});
```

### Node.js API (Main Process)
```javascript
const { webContents } = require('electron');

// Get WebView's webContents
const allWebContents = webContents.getAllWebContents();
```

## Security Considerations

### ⚠️ Important Security Rules

1. **Disable nodeintegration**
   - Do not enable `nodeintegration` unless absolutely necessary
   - This exposes Node.js APIs to remote content, creating serious security risks

2. **Use preload scripts**
   - Expose limited APIs through preload scripts
   - Use `contextBridge` to safely expose functionality

3. **Enable contextIsolation**
   - Always enable `contextIsolation=yes` in webpreferences
   - Isolate preload script and page content contexts

4. **Validate sources**
   - Validate sources when handling IPC messages
   - Do not trust data from WebView

5. **Use CSP**
   - Configure Content Security Policy
   - Restrict resource loading sources

### ✅ Secure Configuration Example

```html
<webview
  src="https://example.com"
  preload="./preload.js"
  webpreferences="contextIsolation=yes, nodeIntegration=no, sandbox=yes"
  disablewebsecurity="false"
></webview>
```

## Rationale (Design Reasoning)

### Why This Skill is Needed

1. **WebView Debugging Complexity**
   - WebView is an independent renderer process with different debugging methods than main window
   - Special APIs and tools are needed for effective debugging

2. **Security Considerations**
   - Improper WebView configuration can lead to serious security vulnerabilities
   - Clear guidance is needed to avoid common security pitfalls

3. **Multi-dimensional Debugging Needs**
   - Script injection, event listening, DevTools, IPC each have their own debugging methods
   - A systematic debugging process and toolset is needed

4. **Common Issues Concentration**
   - Incorrect preload script paths
   - Improperly bound event listeners
   - IPC communication configuration errors
   - These issues require specialized debugging skills

### Why Multi-file Organization

- **Clear Content Classification** - Each topic has its own detailed documentation
- **Easy Maintenance** - Only need to modify corresponding file when updating specific functionality
- **On-demand Reference** - Users can quickly locate needed specific content
- **Avoid Information Overload** - Main document stays concise, details in topic documents

## Recommended Tools

- **Electron DevTools Extension** - Enhanced developer tools
- **electron-devtools-installer** - Install Chrome DevTools extensions
- **electron-debug** - Simplify Electron app debugging
- **devtron** (deprecated, but can be referenced) - Electron-specific DevTools

## Further Reading

- [Electron Official Docs - WebView Tag](https://www.electronjs.org/docs/latest/api/webview-tag)
- [Electron Security Guide](https://www.electronjs.org/docs/latest/tutorial/security)
- [Context Isolation](https://www.electronjs.org/docs/latest/tutorial/context-isolation)
- [IPC Communication](https://www.electronjs.org/docs/latest/tutorial/ipc)
