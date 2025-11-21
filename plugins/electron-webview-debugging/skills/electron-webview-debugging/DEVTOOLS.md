# WebView DevTools Integration and Debugging

This document explains how to use Chrome DevTools to debug Electron WebView Tag.

## DevTools Overview

Electron WebView supports full Chrome DevTools, providing the following debugging capabilities:

| Panel | Function | Main Purpose |
|-----|------|---------|
| **Elements** | DOM inspection and editing | View and modify page structure, CSS styles |
| **Console** | JavaScript console | Execute scripts, view logs, debug code |
| **Sources** | Source code debugging | Set breakpoints, step debugging, view call stack |
| **Network** | Network monitoring | View requests, responses, performance |
| **Performance** | Performance analysis | Analyze rendering performance, find bottlenecks |
| **Memory** | Memory analysis | Detect memory leaks, view heap snapshots |
| **Application** | Application data | View storage, cache, Service Workers |

## Opening DevTools

### Method 1: Using openDevTools() API

This is the most common and direct method:

```javascript
const webview = document.getElementById('myWebview');

// Basic usage: Open DevTools in new window
webview.openDevTools();

// Auto-open after page load complete
webview.addEventListener('did-finish-load', () => {
  webview.openDevTools();
});
```

### Method 2: Using Context Menu

```javascript
// Enable WebView context menu (Inspect Element)
webview.addEventListener('contextmenu', (e) => {
  e.preventDefault();

  const { remote } = require('electron');
  const { Menu, MenuItem } = remote;

  const menu = new Menu();

  // Add "Inspect Element" menu item
  menu.append(new MenuItem({
    label: 'Inspect Element',
    click: () => {
      webview.inspectElement(e.params.x, e.params.y);
    }
  }));

  menu.popup();
});
```

### Method 3: Programmatic Control

```javascript
// Open DevTools based on specific conditions
function openDevToolsIfNeeded() {
  const isDevelopment = process.env.NODE_ENV === 'development';
  const hasDebugFlag = process.argv.includes('--debug');

  if (isDevelopment || hasDebugFlag) {
    webview.openDevTools();
  }
}

webview.addEventListener('did-finish-load', openDevToolsIfNeeded);
```

### ✅ Opening DevTools Best Practice

```javascript
class DevToolsManager {
  constructor(webviewElement) {
    this.webview = webviewElement;
    this.isOpen = false;
    this.setupKeyboardShortcut();
  }

  // Toggle DevTools display
  toggle() {
    if (this.isOpen) {
      this.close();
    } else {
      this.open();
    }
  }

  // Open DevTools
  open() {
    if (!this.isOpen) {
      this.webview.openDevTools();
      this.isOpen = true;
      console.log('DevTools opened');
    }
  }

  // Close DevTools
  close() {
    if (this.isOpen) {
      this.webview.closeDevTools();
      this.isOpen = false;
      console.log('DevTools closed');
    }
  }

  // Set keyboard shortcuts (F12 or Ctrl+Shift+I)
  setupKeyboardShortcut() {
    document.addEventListener('keydown', (e) => {
      // F12
      if (e.key === 'F12') {
        e.preventDefault();
        this.toggle();
      }

      // Ctrl+Shift+I (Windows/Linux) or Cmd+Option+I (Mac)
      if ((e.ctrlKey || e.metaKey) && e.shiftKey && e.key === 'I') {
        e.preventDefault();
        this.toggle();
      }
    });
  }

  // Inspect an element
  inspectElement(x, y) {
    this.webview.inspectElement(x, y);
    this.isOpen = true;
  }
}

// Usage
const devtools = new DevToolsManager(webview);

// Manually open
devtools.open();

// Toggle display
devtools.toggle();

// Inspect specific element
webview.addEventListener('contextmenu', (e) => {
  devtools.inspectElement(e.params.x, e.params.y);
});
```

## DevTools Event Listening

### devtools-opened / devtools-closed

Listen for DevTools opening and closing:

```javascript
webview.addEventListener('devtools-opened', () => {
  console.log('DevTools opened');
  document.getElementById('devtools-indicator').style.display = 'block';
});

webview.addEventListener('devtools-closed', () => {
  console.log('DevTools closed');
  document.getElementById('devtools-indicator').style.display = 'none';
});
```

### devtools-focused

Listen for DevTools gaining focus:

```javascript
webview.addEventListener('devtools-focused', () => {
  console.log('DevTools focused');
});
```

## Using DevTools to Debug Scripts

### Setting Breakpoints in Sources Panel

1. **Open DevTools**
   ```javascript
   webview.openDevTools();
   ```

2. **Switch to Sources panel**

3. **Find the script file to debug**
   - Page scripts: Usually under `top` or domain name
   - Injected scripts: Under `(no domain)`

4. **Set breakpoints**
   - Click line number to set breakpoint
   - Or add `debugger;` statement in code

### Executing Scripts in Console Panel

```javascript
// Can execute code directly in Console
document.title
document.querySelectorAll('a').length

// Use copy() to copy data to clipboard
copy(document.querySelectorAll('a'))

// Use $0 to reference selected element in Elements panel
$0.style.backgroundColor = 'red'
```

### Using console API for Debugging

```javascript
// Use in injected scripts
webview.executeJavaScript(`
  // Basic logging
  console.log('Log message');
  console.warn('Warning message');
  console.error('Error message');

  // Grouped logging
  console.group('My Group');
  console.log('Item 1');
  console.log('Item 2');
  console.groupEnd();

  // Table display
  console.table([
    { name: 'Alice', age: 25 },
    { name: 'Bob', age: 30 }
  ]);

  // Timing
  console.time('operation');
  // ... some operation
  console.timeEnd('operation');

  // Assertion
  console.assert(1 === 2, '1 is not equal to 2');

  // Counting
  console.count('counter');
  console.count('counter');
`);
```

### ✅ Debugging Scripts Best Practice

```javascript
// Create debugger utility class
class WebViewDebugger {
  constructor(webviewElement) {
    this.webview = webviewElement;
  }

  // Inject debug helper into page
  injectDebugHelper() {
    const helperScript = `
      (function() {
        // Create global debug object
        window.__DEBUG__ = {
          // Get all event listeners
          getEventListeners: function(element) {
            return getEventListeners(element);
          },

          // Find elements
          find: function(selector) {
            return document.querySelectorAll(selector);
          },

          // Highlight element
          highlight: function(element) {
            element.style.outline = '2px solid red';
            setTimeout(() => {
              element.style.outline = '';
            }, 2000);
          },

          // Get element path
          getPath: function(element) {
            const path = [];
            while (element && element.nodeType === Node.ELEMENT_NODE) {
              let selector = element.nodeName.toLowerCase();
              if (element.id) {
                selector += '#' + element.id;
              } else if (element.className) {
                selector += '.' + element.className.split(' ').join('.');
              }
              path.unshift(selector);
              element = element.parentNode;
            }
            return path.join(' > ');
          },

          // Monitor performance
          measurePerformance: function() {
            const perf = window.performance;
            return {
              navigationTiming: perf.timing,
              resources: perf.getEntriesByType('resource'),
              marks: perf.getEntriesByType('mark'),
              measures: perf.getEntriesByType('measure')
            };
          }
        };

        console.log('Debug helper injected. Use window.__DEBUG__');
      })()
    `;

    this.webview.executeJavaScript(helperScript);
  }

  // Execute and return result
  async evaluate(expression) {
    try {
      const result = await this.webview.executeJavaScript(expression);
      console.log('Evaluation result:', result);
      return result;
    } catch (error) {
      console.error('Evaluation error:', error);
      throw error;
    }
  }

  // Monitor console output
  startConsoleMonitoring() {
    this.webview.addEventListener('console-message', (e) => {
      const style = this.getConsoleStyle(e.level);
      console.log(`%c[WebView Console] ${e.message}`, style);
    });
  }

  getConsoleStyle(level) {
    const styles = {
      0: 'color: #666',       // log
      1: 'color: #f59e0b',    // warning
      2: 'color: #ef4444'     // error
    };
    return styles[level] || '';
  }
}

// Usage
const debugger = new WebViewDebugger(webview);

webview.addEventListener('did-finish-load', () => {
  // Inject debug helper
  debugger.injectDebugHelper();

  // Start console monitoring
  debugger.startConsoleMonitoring();

  // Open DevTools
  webview.openDevTools();
});

// Execute debug commands in main process
debugger.evaluate('window.__DEBUG__.measurePerformance()');
```

## Using Network Panel

### Monitoring Network Requests

1. **Open DevTools and switch to Network panel**

2. **Reload page to capture requests**
   ```javascript
   webview.reload();
   ```

3. **Filter request types**
   - XHR: View AJAX requests
   - JS: View script files
   - CSS: View stylesheet files
   - Img: View images
   - Doc: View documents

4. **View request details**
   - Headers: Request and response headers
   - Preview: Response content preview
   - Response: Raw response
   - Timing: Request timeline

### Programmatic Network Monitoring

While you can't directly access the Network panel from main process, you can monitor through webRequest API:

```javascript
// In main process
const { session } = require('electron');

// Get WebView's session
const ses = session.fromPartition('persist:myapp');

// Listen to all requests
ses.webRequest.onBeforeRequest((details, callback) => {
  console.log('Request:', details.url);
  callback({});
});

// Listen to request completion
ses.webRequest.onCompleted((details) => {
  console.log('Completed:', details.url, details.statusCode);
});

// Listen to request failure
ses.webRequest.onErrorOccurred((details) => {
  console.error('Failed:', details.url, details.error);
});
```

## Using Performance Panel

### Recording Performance Analysis

1. **Open DevTools Performance panel**

2. **Start recording**
   - Click record button (circle)
   - Or press Ctrl+E (Cmd+E)

3. **Execute operations to analyze**

4. **Stop recording**

5. **Analyze results**
   - View FPS chart
   - View main thread activity
   - View network requests
   - View screenshot timeline

### Programmatic Performance Monitoring

```javascript
// Inject performance monitoring script
webview.executeJavaScript(`
  (function() {
    // Monitor FPS
    let lastTime = performance.now();
    let frames = 0;

    function measureFPS() {
      frames++;
      const currentTime = performance.now();

      if (currentTime >= lastTime + 1000) {
        const fps = Math.round((frames * 1000) / (currentTime - lastTime));
        console.log('FPS:', fps);

        frames = 0;
        lastTime = currentTime;
      }

      requestAnimationFrame(measureFPS);
    }

    requestAnimationFrame(measureFPS);

    // Monitor long tasks
    const observer = new PerformanceObserver((list) => {
      for (const entry of list.getEntries()) {
        console.warn('Long Task:', entry.duration.toFixed(2) + 'ms');
      }
    });

    observer.observe({ entryTypes: ['longtask'] });
  })()
`);
```

## Using Memory Panel

### Detecting Memory Leaks

1. **Open DevTools Memory panel**

2. **Take heap snapshot**
   - Select "Heap snapshot"
   - Click "Take snapshot"

3. **Take another snapshot after executing operations**

4. **Compare snapshots**
   - Select "Comparison" view
   - View memory growth

### Programmatic Memory Monitoring

```javascript
// Inject memory monitoring script
webview.executeJavaScript(`
  (function() {
    if (performance.memory) {
      setInterval(() => {
        const memory = performance.memory;
        console.log('Memory:', {
          usedJSHeapSize: (memory.usedJSHeapSize / 1048576).toFixed(2) + 'MB',
          totalJSHeapSize: (memory.totalJSHeapSize / 1048576).toFixed(2) + 'MB',
          jsHeapSizeLimit: (memory.jsHeapSizeLimit / 1048576).toFixed(2) + 'MB'
        });
      }, 5000);
    } else {
      console.warn('performance.memory not available');
    }
  })()
`);
```

## Remote Debugging

### Enable Remote Debugging Port

Configure at main process startup:

```javascript
// main.js
const { app } = require('electron');

app.commandLine.appendSwitch('remote-debugging-port', '9222');

app.whenReady().then(() => {
  console.log('Remote debugging available at http://localhost:9222');
});
```

### Connect to Remote DevTools

1. **Start application**

2. **Visit in Chrome browser**
   ```
   http://localhost:9222
   ```

3. **Select WebView to debug**

4. **Use full Chrome DevTools**

### ✅ Remote Debugging Best Practice

```javascript
// Only enable remote debugging in development environment
if (process.env.NODE_ENV === 'development') {
  app.commandLine.appendSwitch('remote-debugging-port', '9222');
}

// Or control through command line arguments
if (process.argv.includes('--remote-debug')) {
  const port = process.argv.find(arg => arg.startsWith('--remote-debug-port='))
    ?.split('=')[1] || '9222';
  app.commandLine.appendSwitch('remote-debugging-port', port);
}
```

## DevTools Extensions

### Installing Chrome DevTools Extensions

```javascript
// main.js
const { app, session } = require('electron');
const path = require('path');

app.whenReady().then(() => {
  // Method 1: Load extension from local
  session.defaultSession.loadExtension(
    path.join(__dirname, 'devtools-extensions/react-devtools')
  );

  // Method 2: Use electron-devtools-installer
  const { default: installExtension, REACT_DEVELOPER_TOOLS } = require('electron-devtools-installer');

  installExtension(REACT_DEVELOPER_TOOLS)
    .then((name) => console.log(`Added Extension: ${name}`))
    .catch((err) => console.log('An error occurred: ', err));
});
```

### Common Extensions

- **React Developer Tools** - React application debugging
- **Vue.js devtools** - Vue application debugging
- **Redux DevTools** - Redux state management debugging

## Troubleshooting

### DevTools Won't Open

```bash
# Check if WebView loaded
webview.getURL()

# Check if already open
webview.isDevToolsOpened()

# Try closing and reopening
webview.closeDevTools();
setTimeout(() => webview.openDevTools(), 100);
```

### Can't Find Script in Sources Panel

```javascript
// Ensure script has sourceURL
webview.executeJavaScript(`
  (function() {
    // Your code
  })()
  //# sourceURL=my-injected-script.js
`);
```

### Breakpoints Not Working in DevTools

```javascript
// Ensure breakpoint set in correct context
// preload script: In (isolated world)
// page script: In main context
// executeJavaScript: In main context
```

## Debugging Tips Summary

### ✅ Efficient Debugging Workflow

1. **Open DevTools first**
   ```javascript
   webview.openDevTools();
   ```

2. **Set keyboard shortcuts for quick toggle**
   ```javascript
   document.addEventListener('keydown', (e) => {
     if (e.key === 'F12') {
       webview.isDevToolsOpened() ?
         webview.closeDevTools() :
         webview.openDevTools();
     }
   });
   ```

3. **Use Console panel for quick testing**
   ```javascript
   // Execute in Console
   $0 // Currently selected element
   $$ // Shorthand for querySelectorAll
   copy() // Copy to clipboard
   ```

4. **Use Network panel to troubleshoot request issues**
   - View failed requests
   - Check request headers and responses
   - Analyze request timing

5. **Use Performance panel to optimize performance**
   - Record page load
   - Find long tasks
   - Optimize rendering performance

6. **Regularly check Memory panel**
   - Take heap snapshots
   - Compare memory changes
   - Find memory leaks

## Recommended Tools and Resources

- **Chrome DevTools Official Documentation** - https://developer.chrome.com/docs/devtools/
- **electron-devtools-installer** - Install Chrome extensions
- **electron-debug** - Simplify debugging configuration
- **Lighthouse** - Performance and quality audit tool
