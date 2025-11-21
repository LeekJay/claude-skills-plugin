# WebView Event Handling and Listening

This document explains how to listen to and debug various events of Electron WebView Tag.

## WebView Event Categories

Electron WebView provides a rich event system, which can be categorized as follows:

| Event Type | Purpose | Common Events |
|---------|------|---------|
| **Lifecycle Events** | Monitor page load status | `did-start-loading`, `did-finish-load` |
| **Navigation Events** | Monitor page navigation | `will-navigate`, `did-navigate` |
| **Error Events** | Catch loading and runtime errors | `did-fail-load`, `crashed` |
| **Console Events** | Capture console output | `console-message` |
| **Dialog Events** | Handle popups | `new-window`, `close` |
| **Render Events** | Monitor render process | `dom-ready`, `page-title-updated` |

## Lifecycle Events

### Key Lifecycle Event Flow

```
Load Start → did-start-loading
         ↓
   Load Resources
         ↓
  DOM Parsing Complete → dom-ready
         ↓
  Page Load Complete → did-finish-load
         ↓
  Stop Loading Animation → did-stop-loading
```

### did-start-loading

**Trigger Timing**: When WebView starts loading

**Use Cases**:
- ✅ Show loading indicator
- ✅ Initialize loading state
- ✅ Record load start time

```javascript
const webview = document.getElementById('myWebview');

webview.addEventListener('did-start-loading', () => {
  console.log('WebView started loading');

  // Show loading indicator
  document.getElementById('loading-spinner').style.display = 'block';

  // Record start time
  window.loadStartTime = Date.now();
});
```

### dom-ready

**Trigger Timing**: DOM parsing complete, but external resources (images, stylesheets) may not be loaded

**Use Cases**:
- ✅ Earliest timing to safely manipulate DOM
- ✅ Inject scripts that need to execute before resource loading
- ✅ Modify page structure

```javascript
webview.addEventListener('dom-ready', () => {
  console.log('DOM is ready');

  // Inject early script
  webview.executeJavaScript(`
    console.log('Injected at dom-ready');
    document.body.style.opacity = '0';
  `);
});
```

### did-finish-load

**Trigger Timing**: Page and all resources loaded

**Use Cases**:
- ✅ Hide loading indicator
- ✅ Execute scripts that depend on complete page
- ✅ Calculate load time

```javascript
webview.addEventListener('did-finish-load', () => {
  console.log('WebView finished loading');

  // Hide loading indicator
  document.getElementById('loading-spinner').style.display = 'none';

  // Calculate load time
  const loadTime = Date.now() - window.loadStartTime;
  console.log(`Load time: ${loadTime}ms`);

  // Execute script
  webview.executeJavaScript(`
    document.body.style.opacity = '1';
  `);
});
```

### did-stop-loading

**Trigger Timing**: When loading animation stops (may be after did-finish-load)

**Use Cases**:
- ✅ Final load complete confirmation
- ✅ Clean up loading state

```javascript
webview.addEventListener('did-stop-loading', () => {
  console.log('Loading animation stopped');
});
```

### ✅ Lifecycle Events Best Practice

```javascript
class WebViewLifecycleManager {
  constructor(webviewElement) {
    this.webview = webviewElement;
    this.loadState = {
      isLoading: false,
      startTime: null,
      url: null
    };

    this.setupListeners();
  }

  setupListeners() {
    this.webview.addEventListener('did-start-loading', () => {
      this.loadState.isLoading = true;
      this.loadState.startTime = Date.now();
      this.loadState.url = this.webview.getURL();

      console.log(`[Lifecycle] Loading started: ${this.loadState.url}`);
      this.onLoadStart();
    });

    this.webview.addEventListener('dom-ready', () => {
      console.log('[Lifecycle] DOM ready');
      this.onDomReady();
    });

    this.webview.addEventListener('did-finish-load', () => {
      const loadTime = Date.now() - this.loadState.startTime;
      console.log(`[Lifecycle] Loading finished in ${loadTime}ms`);
      this.onLoadFinish(loadTime);
    });

    this.webview.addEventListener('did-stop-loading', () => {
      this.loadState.isLoading = false;
      console.log('[Lifecycle] Loading stopped');
      this.onLoadStop();
    });
  }

  // Hook methods for subclasses or external use
  onLoadStart() {}
  onDomReady() {}
  onLoadFinish(loadTime) {}
  onLoadStop() {}
}

// Usage
const manager = new WebViewLifecycleManager(webview);
manager.onLoadFinish = (loadTime) => {
  console.log(`Custom handler: Loaded in ${loadTime}ms`);
};
```

## Navigation Events

### will-navigate

**Trigger Timing**: Before user clicks link or program calls navigation API

**Use Cases**:
- ✅ Intercept navigation
- ✅ Validate target URL
- ✅ Block access to specific domains

```javascript
webview.addEventListener('will-navigate', (e) => {
  console.log('Will navigate to:', e.url);

  // Intercept navigation (prevent redirect)
  if (e.url.includes('blocked-domain.com')) {
    e.preventDefault();
    console.log('Navigation blocked');
  }
});
```

### did-navigate

**Trigger Timing**: After navigation completes

**Use Cases**:
- ✅ Update URL display
- ✅ Record navigation history
- ✅ Update navigation button state

```javascript
webview.addEventListener('did-navigate', (e) => {
  console.log('Navigated to:', e.url);

  // Update address bar
  document.getElementById('url-bar').value = e.url;

  // Update forward/back buttons
  updateNavigationButtons();
});

function updateNavigationButtons() {
  document.getElementById('back-btn').disabled = !webview.canGoBack();
  document.getElementById('forward-btn').disabled = !webview.canGoForward();
}
```

### did-navigate-in-page

**Trigger Timing**: In-page navigation (e.g., anchor jumps, History API)

**Use Cases**:
- ✅ Monitor single-page application route changes
- ✅ Track anchor jumps

```javascript
webview.addEventListener('did-navigate-in-page', (e) => {
  if (e.isMainFrame) {
    console.log('In-page navigation:', e.url);
  }
});
```

### ✅ Navigation Events Best Practice

```javascript
// URL whitelist validation
const allowedDomains = ['example.com', 'trusted-site.com'];

webview.addEventListener('will-navigate', (e) => {
  const url = new URL(e.url);
  const isAllowed = allowedDomains.some(domain =>
    url.hostname === domain || url.hostname.endsWith(`.${domain}`)
  );

  if (!isAllowed) {
    e.preventDefault();
    console.warn(`Navigation blocked: ${e.url}`);

    // Show warning
    showWarning(`This link is not allowed: ${url.hostname}`);
  }
});

// Navigation history tracking
const navigationHistory = [];

webview.addEventListener('did-navigate', (e) => {
  navigationHistory.push({
    url: e.url,
    timestamp: Date.now(),
    isMainFrame: e.isMainFrame
  });

  // Limit history size
  if (navigationHistory.length > 50) {
    navigationHistory.shift();
  }
});
```

## Error Events

### did-fail-load

**Trigger Timing**: Page load failure

**Use Cases**:
- ✅ Show error page
- ✅ Log errors
- ✅ Retry loading

```javascript
webview.addEventListener('did-fail-load', (e) => {
  // e.errorCode: Error code
  // e.errorDescription: Error description
  // e.validatedURL: Failed URL

  if (e.errorCode !== -3) { // -3 is user cancel, no need to handle
    console.error('Load failed:', {
      code: e.errorCode,
      description: e.errorDescription,
      url: e.validatedURL
    });

    // Show error page
    showErrorPage(e.errorDescription);
  }
});

function showErrorPage(errorMessage) {
  const errorHTML = `
    <!DOCTYPE html>
    <html>
    <head>
      <style>
        body { font-family: Arial; text-align: center; padding: 50px; }
        .error { color: #d32f2f; }
      </style>
    </head>
    <body>
      <h1 class="error">Page Load Failed</h1>
      <p>${errorMessage}</p>
      <button onclick="location.reload()">Reload</button>
    </body>
    </html>
  `;

  webview.loadURL(`data:text/html,${encodeURIComponent(errorHTML)}`);
}
```

### crashed

**Trigger Timing**: WebView renderer process crashed

**Use Cases**:
- ✅ Restart WebView
- ✅ Log crashes
- ✅ Notify user

```javascript
webview.addEventListener('crashed', () => {
  console.error('WebView crashed!');

  // Log crash
  logCrash({
    url: webview.getURL(),
    timestamp: Date.now()
  });

  // Show crash prompt
  const shouldReload = confirm('Page crashed, reload?');
  if (shouldReload) {
    webview.reload();
  }
});
```

### unresponsive

**Trigger Timing**: WebView renderer process unresponsive

**Use Cases**:
- ✅ Wait for response recovery
- ✅ Prompt user
- ✅ Provide terminate option

```javascript
webview.addEventListener('unresponsive', () => {
  console.warn('WebView is unresponsive');

  const shouldWait = confirm('Page unresponsive, continue waiting?');
  if (!shouldWait) {
    webview.stop();
    webview.reload();
  }
});

webview.addEventListener('responsive', () => {
  console.log('WebView is responsive again');
});
```

### ✅ Error Handling Best Practice

```javascript
class WebViewErrorHandler {
  constructor(webviewElement) {
    this.webview = webviewElement;
    this.errorCount = 0;
    this.crashCount = 0;
    this.maxRetries = 3;

    this.setupErrorHandlers();
  }

  setupErrorHandlers() {
    // Load failure handler
    this.webview.addEventListener('did-fail-load', (e) => {
      if (e.errorCode === -3) return; // User cancel

      this.errorCount++;
      console.error(`[Error ${this.errorCount}] Load failed:`, e);

      if (this.errorCount < this.maxRetries) {
        setTimeout(() => {
          console.log(`Retrying (${this.errorCount}/${this.maxRetries})...`);
          this.webview.reload();
        }, 2000);
      } else {
        this.showPermanentError(e);
      }
    });

    // Crash handler
    this.webview.addEventListener('crashed', () => {
      this.crashCount++;
      console.error(`[Crash ${this.crashCount}]`);

      if (this.crashCount < this.maxRetries) {
        setTimeout(() => {
          console.log('Reloading after crash...');
          this.webview.reload();
        }, 1000);
      } else {
        this.showCrashError();
      }
    });

    // Reset counters on successful load
    this.webview.addEventListener('did-finish-load', () => {
      this.errorCount = 0;
      this.crashCount = 0;
    });
  }

  showPermanentError(error) {
    console.error('Max retries reached');
    // Show error UI
  }

  showCrashError() {
    console.error('Too many crashes');
    // Show crash UI
  }
}
```

## Console Events

### console-message

**Trigger Timing**: Console output from WebView page

**Use Cases**:
- ✅ Capture page logs
- ✅ Debug page code
- ✅ Monitor errors and warnings

```javascript
webview.addEventListener('console-message', (e) => {
  // e.level: 0=log, 1=warning, 2=error
  // e.message: Message content
  // e.line: Line number
  // e.sourceId: Source file

  const levelMap = {
    0: 'LOG',
    1: 'WARN',
    2: 'ERROR'
  };

  console.log(`[WebView ${levelMap[e.level]}] ${e.message} (${e.sourceId}:${e.line})`);
});
```

### ✅ Console Monitoring Best Practice

```javascript
class WebViewConsoleMonitor {
  constructor(webviewElement) {
    this.webview = webviewElement;
    this.logs = [];
    this.maxLogs = 1000;

    this.setupConsoleMonitor();
  }

  setupConsoleMonitor() {
    this.webview.addEventListener('console-message', (e) => {
      const logEntry = {
        level: this.getLevelName(e.level),
        message: e.message,
        source: e.sourceId,
        line: e.line,
        timestamp: Date.now()
      };

      // Add to log
      this.logs.push(logEntry);
      if (this.logs.length > this.maxLogs) {
        this.logs.shift();
      }

      // Handle by level
      switch (e.level) {
        case 2: // ERROR
          this.onError(logEntry);
          break;
        case 1: // WARN
          this.onWarning(logEntry);
          break;
        default: // LOG
          this.onLog(logEntry);
      }
    });
  }

  getLevelName(level) {
    const map = { 0: 'LOG', 1: 'WARN', 2: 'ERROR' };
    return map[level] || 'UNKNOWN';
  }

  onError(entry) {
    console.error(`[WebView Error] ${entry.message}`);
    // Can send error report
  }

  onWarning(entry) {
    console.warn(`[WebView Warning] ${entry.message}`);
  }

  onLog(entry) {
    console.log(`[WebView Log] ${entry.message}`);
  }

  // Export logs
  exportLogs() {
    return JSON.stringify(this.logs, null, 2);
  }

  // Clear logs
  clearLogs() {
    this.logs = [];
  }

  // Search logs
  searchLogs(keyword) {
    return this.logs.filter(log =>
      log.message.toLowerCase().includes(keyword.toLowerCase())
    );
  }
}

// Usage
const consoleMonitor = new WebViewConsoleMonitor(webview);

// Export logs
console.log(consoleMonitor.exportLogs());

// Search errors
const errors = consoleMonitor.searchLogs('error');
```

## Other Important Events

### page-title-updated

**Trigger Timing**: Page title updated

```javascript
webview.addEventListener('page-title-updated', (e) => {
  console.log('Title updated:', e.title);
  document.title = e.title; // Sync window title
});
```

### page-favicon-updated

**Trigger Timing**: Page icon updated

```javascript
webview.addEventListener('page-favicon-updated', (e) => {
  console.log('Favicon updated:', e.favicons);
  // e.favicons is an array of icon URLs
  if (e.favicons.length > 0) {
    document.querySelector('link[rel="icon"]').href = e.favicons[0];
  }
});
```

### new-window

**Trigger Timing**: Page requests to open new window (e.g., target="_blank")

```javascript
webview.addEventListener('new-window', (e) => {
  console.log('New window requested:', e.url);

  // Prevent opening new window, load in current WebView
  e.preventDefault();
  webview.loadURL(e.url);
});
```

## Complete Event Listening Example

```javascript
class WebViewEventManager {
  constructor(webviewElement) {
    this.webview = webviewElement;
    this.setupAllListeners();
  }

  setupAllListeners() {
    // Lifecycle
    this.webview.addEventListener('did-start-loading', () =>
      this.log('Loading started'));
    this.webview.addEventListener('did-finish-load', () =>
      this.log('Loading finished'));

    // Navigation
    this.webview.addEventListener('will-navigate', (e) =>
      this.log(`Will navigate to: ${e.url}`));
    this.webview.addEventListener('did-navigate', (e) =>
      this.log(`Navigated to: ${e.url}`));

    // Errors
    this.webview.addEventListener('did-fail-load', (e) =>
      this.error(`Load failed: ${e.errorDescription}`));
    this.webview.addEventListener('crashed', () =>
      this.error('WebView crashed!'));

    // Console
    this.webview.addEventListener('console-message', (e) =>
      this.log(`Console [${e.level}]: ${e.message}`));

    // Others
    this.webview.addEventListener('page-title-updated', (e) =>
      this.log(`Title: ${e.title}`));
    this.webview.addEventListener('new-window', (e) => {
      e.preventDefault();
      this.log(`New window blocked: ${e.url}`);
    });
  }

  log(message) {
    console.log(`[WebView] ${message}`);
  }

  error(message) {
    console.error(`[WebView] ${message}`);
  }
}

// Usage
const eventManager = new WebViewEventManager(
  document.getElementById('myWebview')
);
```

## Troubleshooting

### Event Not Triggered

```bash
# Check if WebView element exists
document.getElementById('myWebview') !== null

# Check if event listener is correctly bound
webview.addEventListener('did-finish-load', () => console.log('test'));

# Confirm if WebView is loading
webview.isLoading()
```

### Event Triggered Multiple Times

```javascript
// ❌ Wrong: Duplicate binding
function setupWebView() {
  webview.addEventListener('did-finish-load', handler);
}
setupWebView();
setupWebView(); // handler will be called twice!

// ✅ Correct: Remove old listener or use once
function setupWebView() {
  webview.removeEventListener('did-finish-load', handler);
  webview.addEventListener('did-finish-load', handler);
}

// Or use once option (if supported)
webview.addEventListener('did-finish-load', handler, { once: true });
```

## Recommended Tools

- **Chrome DevTools** - View console output and network requests
- **Electron DevTools Extensions** - Enhanced debugging capabilities
- **Logging libraries** - e.g., winston, pino for log recording
