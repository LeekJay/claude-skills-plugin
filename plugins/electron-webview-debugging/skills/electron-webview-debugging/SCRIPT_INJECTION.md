# WebView Script Injection Debugging

This document explains how to inject and debug JavaScript scripts and CSS styles in Electron WebView Tag.

## Script Injection Methods Overview

Electron WebView provides three main script injection methods:

| Method | Timing | Use Case | Context |
|-----|------|------|--------|
| **preload script** | Before page load | Initialization, API bridging | Isolated context (requires contextBridge) |
| **executeJavaScript()** | After page load | Dynamic script execution | Page context |
| **insertCSS()** | Any time | Style injection | Page styles |

## Method 1: preload Script

### When to Use

- ✅ Need to initialize environment before page scripts run
- ✅ Need to expose secure APIs to the page
- ✅ Need to intercept or modify page global objects
- ✅ Need to establish communication bridge between main process and WebView

### Basic Usage

#### 1. Configure preload attribute in HTML

```html
<webview
  id="myWebview"
  src="https://example.com"
  preload="./preload.js"
  webpreferences="contextIsolation=yes"
></webview>
```

#### 2. Write preload.js script

```javascript
// preload.js
const { contextBridge, ipcRenderer } = require('electron');

// Use contextBridge to safely expose API
contextBridge.exposeInMainWorld('electronAPI', {
  // Expose methods to the page
  sendMessage: (message) => ipcRenderer.send('webview-message', message),
  onReply: (callback) => ipcRenderer.on('main-reply', (event, data) => callback(data)),

  // Expose constants
  appVersion: '1.0.0',
  platform: process.platform
});

// Modify environment before page scripts execute
window.addEventListener('DOMContentLoaded', () => {
  console.log('Preload script executed');
});
```

### Debugging preload Scripts

#### Check preload Path

```bash
# Search for preload attribute configuration
grep -r 'preload=' --include="*.html" --include="*.js"
```

Ensure path is correct:
- ✅ Relative path: `./preload.js` (relative to HTML file)
- ✅ Absolute path: `file:///path/to/preload.js`
- ❌ Wrong: `preload.js` (may not find file)

#### Verify preload is Loaded

Test in WebView's page:

```javascript
// Execute in browser console
console.log(typeof window.electronAPI); // Should output 'object'
console.log(window.electronAPI);        // Should display exposed API
```

#### Common Issues Troubleshooting

**Issue 1: preload script not executed**

```bash
# Check if file exists
ls -la path/to/preload.js

# Check file permissions
stat path/to/preload.js
```

**Issue 2: API not exposed to page**

Check if contextIsolation is enabled:

```html
<!-- ✅ Correct: contextIsolation enabled -->
<webview webpreferences="contextIsolation=yes" preload="./preload.js"></webview>

<!-- ❌ Wrong: Not enabled will cause contextBridge unavailable -->
<webview preload="./preload.js"></webview>
```

**Issue 3: require error**

preload scripts can use Node.js API, but pages cannot:

```javascript
// ✅ Can use in preload.js
const fs = require('fs');
const path = require('path');

// ❌ Cannot use in page scripts (unless nodeintegration=yes, not recommended)
// const fs = require('fs'); // This will error
```

### ✅ Best Practice Example

```javascript
// preload.js - Safe and feature-complete example
const { contextBridge, ipcRenderer } = require('electron');

// Only expose necessary API, don't expose entire ipcRenderer
contextBridge.exposeInMainWorld('electronAPI', {
  // One-way communication (send)
  send: (channel, data) => {
    // Whitelist validation
    const validChannels = ['webview-message', 'webview-request'];
    if (validChannels.includes(channel)) {
      ipcRenderer.send(channel, data);
    }
  },

  // Two-way communication (invoke/handle pattern)
  invoke: async (channel, data) => {
    const validChannels = ['get-data', 'save-data'];
    if (validChannels.includes(channel)) {
      return await ipcRenderer.invoke(channel, data);
    }
  },

  // Receive messages (listen)
  on: (channel, callback) => {
    const validChannels = ['main-reply', 'data-update'];
    if (validChannels.includes(channel)) {
      const subscription = (event, ...args) => callback(...args);
      ipcRenderer.on(channel, subscription);

      // Return unsubscribe function
      return () => {
        ipcRenderer.removeListener(channel, subscription);
      };
    }
  }
});

// Page environment initialization
window.addEventListener('DOMContentLoaded', () => {
  console.log('Preload initialized');

  // Notify main process preload loaded
  ipcRenderer.send('preload-loaded');
});
```

### ❌ Anti-pattern: Unsafe Practices

```javascript
// ❌ Dangerous: Directly expose ipcRenderer
contextBridge.exposeInMainWorld('electronAPI', {
  ipcRenderer: ipcRenderer  // Allows page to send arbitrary IPC messages!
});

// ❌ Dangerous: Expose require
contextBridge.exposeInMainWorld('require', require);

// ❌ Dangerous: Expose sensitive Node.js APIs
contextBridge.exposeInMainWorld('fs', require('fs'));
contextBridge.exposeInMainWorld('exec', require('child_process').exec);
```

## Method 2: executeJavaScript()

### When to Use

- ✅ Need to dynamically execute scripts after page load
- ✅ Need to retrieve data from the page
- ✅ Need to execute different scripts based on conditions
- ✅ Need to inject code at specific timing

### Basic Usage

```javascript
const webview = document.getElementById('myWebview');

// Wait for page load complete
webview.addEventListener('did-finish-load', () => {
  // Execute simple script
  webview.executeJavaScript('console.log("Hello from injected script")');

  // Execute and get return value
  webview.executeJavaScript('document.title').then(title => {
    console.log('Page title:', title);
  });

  // Execute complex script
  const script = `
    (function() {
      const elements = document.querySelectorAll('a');
      return Array.from(elements).map(a => ({
        text: a.textContent,
        href: a.href
      }));
    })()
  `;

  webview.executeJavaScript(script).then(links => {
    console.log('Found links:', links);
  });
});
```

### Debugging executeJavaScript

#### Check Execution Timing

```javascript
const webview = document.getElementById('myWebview');

// ❌ Wrong: Execute before page loaded
webview.executeJavaScript('console.log("Too early")'); // May fail

// ✅ Correct: Wait for load complete
webview.addEventListener('did-finish-load', () => {
  webview.executeJavaScript('console.log("Perfect timing")');
});
```

#### Handle Errors

```javascript
webview.executeJavaScript('undefined.property')
  .then(result => {
    console.log('Result:', result);
  })
  .catch(error => {
    console.error('Script execution failed:', error);
  });
```

#### Debug Complex Scripts

```javascript
// Use IIFE wrapper to avoid polluting global scope
const script = `
  (function() {
    try {
      // Your code
      const data = document.querySelector('#data');
      return {
        success: true,
        value: data ? data.textContent : null
      };
    } catch (error) {
      return {
        success: false,
        error: error.message
      };
    }
  })()
`;

webview.executeJavaScript(script).then(result => {
  if (result.success) {
    console.log('Data:', result.value);
  } else {
    console.error('Script error:', result.error);
  }
});
```

### ✅ Best Practice Example

```javascript
// Utility function: Safe script execution
function safeExecuteScript(webview, script, timeout = 5000) {
  return Promise.race([
    webview.executeJavaScript(script),
    new Promise((_, reject) =>
      setTimeout(() => reject(new Error('Script execution timeout')), timeout)
    )
  ]);
}

// Usage example
webview.addEventListener('did-finish-load', async () => {
  try {
    const title = await safeExecuteScript(webview, 'document.title');
    console.log('Title:', title);
  } catch (error) {
    console.error('Failed to get title:', error);
  }
});
```

### ❌ Anti-pattern: Common Mistakes

```javascript
// ❌ Error 1: Not waiting for Promise
webview.executeJavaScript('document.title'); // No output
// ✅ Correct:
webview.executeJavaScript('document.title').then(console.log);

// ❌ Error 2: Script syntax error
webview.executeJavaScript('consle.log("typo")'); // console misspelled

// ❌ Error 3: Returning non-serializable value
webview.executeJavaScript('document.querySelector("div")');
// DOM elements cannot be serialized, returns empty object

// ✅ Correct: Return serializable data
webview.executeJavaScript(`
  (function() {
    const div = document.querySelector("div");
    return {
      tagName: div.tagName,
      textContent: div.textContent,
      className: div.className
    };
  })()
`);
```

## Method 3: insertCSS()

### When to Use

- ✅ Need to modify page styles
- ✅ Need to hide or show specific elements
- ✅ Need to override website's original styles
- ✅ Need to implement dark mode or other themes

### Basic Usage

```javascript
const webview = document.getElementById('myWebview');

webview.addEventListener('did-finish-load', () => {
  // Inject CSS
  const css = `
    body {
      font-family: 'Arial', sans-serif !important;
      background-color: #f5f5f5 !important;
    }

    .ads {
      display: none !important;
    }
  `;

  webview.insertCSS(css);
});
```

### Debugging CSS Injection

#### Verify CSS Takes Effect

Open WebView's DevTools, check Elements panel:

```javascript
webview.openDevTools();
```

In DevTools:
1. Select Elements tab
2. Check if injected `<style>` tag exists in `<head>`
3. Check if styles are overridden by other rules

#### Use !important to Increase Priority

```css
/* ✅ Use !important to ensure style takes effect */
.element {
  color: red !important;
}

/* ❌ May be overridden by site's original styles */
.element {
  color: red;
}
```

### ✅ Best Practice Example

```javascript
// Dark mode example
function enableDarkMode(webview) {
  const darkModeCSS = `
    html {
      filter: invert(1) hue-rotate(180deg);
    }

    img, video, [style*="background-image"] {
      filter: invert(1) hue-rotate(180deg);
    }
  `;

  webview.insertCSS(darkModeCSS);
}

// Hide ads example
function hideAds(webview) {
  const adBlockCSS = `
    [class*="ad-"],
    [id*="ad-"],
    [class*="advertisement"],
    iframe[src*="doubleclick"],
    iframe[src*="googlesyndication"] {
      display: none !important;
      visibility: hidden !important;
      opacity: 0 !important;
      width: 0 !important;
      height: 0 !important;
    }
  `;

  webview.insertCSS(adBlockCSS);
}
```

## Combined Usage: Complete Example

```javascript
const webview = document.getElementById('myWebview');

// 1. Set preload script
webview.setAttribute('preload', './preload.js');

// 2. Inject CSS after page load
webview.addEventListener('did-finish-load', () => {
  // Inject styles
  webview.insertCSS(`
    body {
      margin: 20px;
      padding: 20px;
    }
  `);

  // Inject script
  const script = `
    (function() {
      // Add custom functionality
      window.customFeature = {
        getData: () => document.title,
        setTitle: (title) => { document.title = title; }
      };

      console.log('Custom feature injected');
    })()
  `;

  webview.executeJavaScript(script);
});

// 3. Listen for console messages
webview.addEventListener('console-message', (e) => {
  console.log(`[WebView Console] ${e.level}: ${e.message}`);
});
```

## Security Considerations

### ⚠️ Script Injection Security Rules

1. **Validate injected content**
   ```javascript
   // ❌ Dangerous: Directly inject user input
   webview.executeJavaScript(`alert("${userInput}")`);

   // ✅ Safe: Escape or validate input
   const safeInput = userInput.replace(/["'\\]/g, '\\$&');
   webview.executeJavaScript(`alert("${safeInput}")`);
   ```

2. **Use preload instead of nodeintegration**
   ```html
   <!-- ❌ Dangerous -->
   <webview nodeintegration="yes"></webview>

   <!-- ✅ Safe -->
   <webview preload="./preload.js" webpreferences="contextIsolation=yes"></webview>
   ```

3. **Limit API exposure scope**
   ```javascript
   // ❌ Dangerous: Expose too much functionality
   contextBridge.exposeInMainWorld('api', {
     exec: require('child_process').exec,
     readFile: require('fs').readFileSync
   });

   // ✅ Safe: Only expose necessary, restricted functionality
   contextBridge.exposeInMainWorld('api', {
     readConfig: () => ipcRenderer.invoke('read-config'),
     saveData: (data) => ipcRenderer.invoke('save-data', data)
   });
   ```

## Troubleshooting Checklist

### Preload Script Issues
- [ ] Check if preload file path is correct
- [ ] Confirm contextIsolation is enabled
- [ ] Verify contextBridge API is correctly exposed
- [ ] Check if preload script has syntax errors
- [ ] Check console for preload-related errors

### executeJavaScript Issues
- [ ] Confirm execution after did-finish-load
- [ ] Check if script has syntax errors
- [ ] Verify return value is serializable
- [ ] Add .catch() to handle errors
- [ ] Use DevTools to verify script logic

### insertCSS Issues
- [ ] Check if CSS syntax is correct
- [ ] Use !important to increase priority
- [ ] Verify styles take effect in DevTools
- [ ] Check if overridden by other styles
- [ ] Verify selectors correctly match elements

## Recommended Tools

- **Chrome DevTools** - Inspect injected scripts and styles
- **Electron Fiddle** - Test WebView functionality
- **ESLint** - Check preload script syntax
- **Prettier** - Format injected code
