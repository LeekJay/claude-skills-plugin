# WebView IPC Communication Debugging

This document explains how to debug IPC (Inter-Process Communication) between Electron main process and WebView Tag.

## IPC Communication Overview

Electron WebView IPC communication involves three parts:

```
┌─────────────┐         ┌──────────────────┐         ┌─────────────┐
│  Main       │ ←────→  │  Renderer        │ ←────→  │  WebView    │
│  Process    │   IPC   │  Process         │   IPC   │  (Guest)    │
│             │         │                  │         │             │
│ ipcMain     │         │ ipcRenderer      │         │ ipcRenderer │
│             │         │                  │         │ (in preload)│
└─────────────┘         └──────────────────┘         └─────────────┘
```

## IPC Communication Methods

### Method 1: Through ipc-message Event

WebView can communicate with the main window's renderer process through the `ipc-message` event.

#### Sending Messages from WebView (preload script)

```javascript
// preload.js
const { ipcRenderer } = require('electron');

// Send message to main window's renderer process
ipcRenderer.sendToHost('channel-name', { data: 'hello' });
```

#### Main Window Renderer Process Receiving Messages

```javascript
// renderer.js (main window)
const webview = document.getElementById('myWebview');

webview.addEventListener('ipc-message', (event) => {
  console.log('Received from WebView:', event.channel, event.args);

  if (event.channel === 'channel-name') {
    const data = event.args[0];
    console.log('Data:', data);

    // Reply message
    webview.send('reply-channel', { reply: 'received' });
  }
});
```

#### WebView Receiving Reply (preload script)

```javascript
// preload.js
ipcRenderer.on('reply-channel', (event, data) => {
  console.log('Reply from host:', data);
});
```

### Method 2: Through Main Process Relay

#### WebView → Main Process

```javascript
// preload.js
const { ipcRenderer } = require('electron');

// Send to main process
ipcRenderer.send('webview-to-main', { message: 'hello' });
```

```javascript
// main.js
const { ipcMain } = require('electron');

ipcMain.on('webview-to-main', (event, data) => {
  console.log('Received from WebView:', data);

  // Reply
  event.reply('main-to-webview', { reply: 'hi' });
});
```

```javascript
// preload.js
ipcRenderer.on('main-to-webview', (event, data) => {
  console.log('Reply from main:', data);
});
```

### Method 3: Using invoke/handle (Recommended)

This is the most modern and secure method, supporting async operations and return values.

#### Main Process Setup Handler

```javascript
// main.js
const { ipcMain } = require('electron');

ipcMain.handle('get-data', async (event, params) => {
  console.log('Handle get-data:', params);

  // Execute async operation
  const result = await fetchData(params);

  return result;
});
```

#### WebView Invoke (preload script)

```javascript
// preload.js
const { ipcRenderer, contextBridge } = require('electron');

contextBridge.exposeInMainWorld('electronAPI', {
  getData: async (params) => {
    return await ipcRenderer.invoke('get-data', params);
  }
});
```

#### Use in Page

```javascript
// In WebView page
const result = await window.electronAPI.getData({ id: 123 });
console.log('Result:', result);
```

## Secure IPC Bridging

### ✅ Best Practice: Using contextBridge

```javascript
// preload.js - Secure IPC bridging
const { contextBridge, ipcRenderer } = require('electron');

// Define allowed channel whitelist
const validSendChannels = [
  'webview-message',
  'webview-request',
  'log-message'
];

const validReceiveChannels = [
  'main-reply',
  'data-update',
  'notification'
];

const validInvokeChannels = [
  'get-data',
  'save-data',
  'delete-data'
];

// Expose secure API
contextBridge.exposeInMainWorld('electronAPI', {
  // One-way send (no reply expected)
  send: (channel, data) => {
    if (validSendChannels.includes(channel)) {
      ipcRenderer.send(channel, data);
    } else {
      console.error(`Channel "${channel}" is not allowed`);
    }
  },

  // Two-way communication (invoke/handle pattern)
  invoke: async (channel, data) => {
    if (validInvokeChannels.includes(channel)) {
      return await ipcRenderer.invoke(channel, data);
    } else {
      throw new Error(`Channel "${channel}" is not allowed`);
    }
  },

  // Receive messages
  on: (channel, callback) => {
    if (validReceiveChannels.includes(channel)) {
      const subscription = (event, ...args) => callback(...args);
      ipcRenderer.on(channel, subscription);

      // Return unsubscribe function
      return () => {
        ipcRenderer.removeListener(channel, subscription);
      };
    } else {
      console.error(`Channel "${channel}" is not allowed`);
    }
  },

  // One-time listen
  once: (channel, callback) => {
    if (validReceiveChannels.includes(channel)) {
      ipcRenderer.once(channel, (event, ...args) => callback(...args));
    } else {
      console.error(`Channel "${channel}" is not allowed`);
    }
  }
});
```

### ❌ Anti-pattern: Unsafe Practices

```javascript
// ❌ Dangerous: Directly expose ipcRenderer
contextBridge.exposeInMainWorld('ipc', ipcRenderer);

// ❌ Dangerous: Allow any channel
contextBridge.exposeInMainWorld('electronAPI', {
  send: (channel, data) => ipcRenderer.send(channel, data), // No validation!
});

// ❌ Dangerous: Expose sendSync along with send
contextBridge.exposeInMainWorld('electronAPI', {
  sendSync: (channel, data) => ipcRenderer.sendSync(channel, data), // Can block renderer!
});
```

## IPC Debugging Tools

### Debugger Utility Class

```javascript
// ipc-debugger.js - For main window renderer process
class IPCDebugger {
  constructor(webviewElement) {
    this.webview = webviewElement;
    this.messageLog = [];
    this.maxLogSize = 1000;

    this.setupMessageLogging();
  }

  // Monitor all IPC messages
  setupMessageLogging() {
    this.webview.addEventListener('ipc-message', (event) => {
      const logEntry = {
        direction: 'received',
        channel: event.channel,
        args: event.args,
        timestamp: Date.now()
      };

      this.logMessage(logEntry);
      this.onMessageReceived(logEntry);
    });

    // Intercept sent messages
    const originalSend = this.webview.send.bind(this.webview);
    this.webview.send = (channel, ...args) => {
      const logEntry = {
        direction: 'sent',
        channel: channel,
        args: args,
        timestamp: Date.now()
      };

      this.logMessage(logEntry);
      this.onMessageSent(logEntry);

      return originalSend(channel, ...args);
    };
  }

  // Log message
  logMessage(entry) {
    this.messageLog.push(entry);

    // Limit log size
    if (this.messageLog.length > this.maxLogSize) {
      this.messageLog.shift();
    }

    // Output to console
    const directionIcon = entry.direction === 'sent' ? '→' : '←';
    console.log(`[IPC ${directionIcon}] ${entry.channel}`, entry.args);
  }

  // Hook: message received
  onMessageReceived(entry) {
    // Subclasses can override this method
  }

  // Hook: message sent
  onMessageSent(entry) {
    // Subclasses can override this method
  }

  // Get message log
  getLog() {
    return this.messageLog;
  }

  // Filter log by channel
  filterByChannel(channel) {
    return this.messageLog.filter(entry => entry.channel === channel);
  }

  // Filter log by direction
  filterByDirection(direction) {
    return this.messageLog.filter(entry => entry.direction === direction);
  }

  // Export log
  exportLog() {
    return JSON.stringify(this.messageLog, null, 2);
  }

  // Clear log
  clearLog() {
    this.messageLog = [];
  }

  // Live monitoring (output to UI)
  startLiveMonitor(containerElement) {
    this.onMessageReceived = (entry) => {
      this.appendToMonitor(containerElement, entry);
    };

    this.onMessageSent = (entry) => {
      this.appendToMonitor(containerElement, entry);
    };
  }

  appendToMonitor(container, entry) {
    const div = document.createElement('div');
    div.className = `ipc-message ipc-${entry.direction}`;
    div.innerHTML = `
      <span class="time">${new Date(entry.timestamp).toLocaleTimeString()}</span>
      <span class="direction">${entry.direction === 'sent' ? '→' : '←'}</span>
      <span class="channel">${entry.channel}</span>
      <span class="args">${JSON.stringify(entry.args)}</span>
    `;
    container.appendChild(div);

    // Auto-scroll to bottom
    container.scrollTop = container.scrollHeight;
  }
}

// Usage
const ipcDebugger = new IPCDebugger(webview);

// View log
console.log(ipcDebugger.getLog());

// Filter specific channel
console.log(ipcDebugger.filterByChannel('webview-message'));

// Export log
console.log(ipcDebugger.exportLog());
```

### Main Process IPC Debugging

```javascript
// main.js
const { ipcMain } = require('electron');

class MainIPCDebugger {
  constructor() {
    this.messageLog = [];
    this.setupInterceptors();
  }

  setupInterceptors() {
    // Intercept ipcMain.on
    const originalOn = ipcMain.on.bind(ipcMain);
    ipcMain.on = (channel, listener) => {
      console.log(`[IPC Main] Registered listener for: ${channel}`);

      const wrappedListener = (event, ...args) => {
        this.logMessage('received', channel, args);
        return listener(event, ...args);
      };

      return originalOn(channel, wrappedListener);
    };

    // Intercept ipcMain.handle
    const originalHandle = ipcMain.handle.bind(ipcMain);
    ipcMain.handle = (channel, listener) => {
      console.log(`[IPC Main] Registered handler for: ${channel}`);

      const wrappedListener = async (event, ...args) => {
        this.logMessage('received', channel, args);
        const result = await listener(event, ...args);
        this.logMessage('replied', channel, [result]);
        return result;
      };

      return originalHandle(channel, wrappedListener);
    };
  }

  logMessage(direction, channel, args) {
    const entry = {
      direction,
      channel,
      args,
      timestamp: Date.now()
    };

    this.messageLog.push(entry);
    console.log(`[IPC Main ${direction}] ${channel}`, args);
  }

  getLog() {
    return this.messageLog;
  }

  exportLog() {
    return JSON.stringify(this.messageLog, null, 2);
  }
}

// Enable debugging
const mainDebugger = new MainIPCDebugger();

// Export log before app quits
app.on('before-quit', () => {
  const fs = require('fs');
  fs.writeFileSync('ipc-log.json', mainDebugger.exportLog());
});
```

## Common IPC Issues Debugging

### Issue 1: Message Not Received

#### Checklist

```javascript
// 1. Confirm preload script loaded
webview.addEventListener('dom-ready', () => {
  webview.executeJavaScript('typeof window.electronAPI')
    .then(type => {
      if (type === 'undefined') {
        console.error('electronAPI not exposed! Check preload script.');
      }
    });
});

// 2. Verify channel names match
// Sender
ipcRenderer.send('my-channel', data);

// Receiver (ensure names match exactly)
ipcMain.on('my-channel', (event, data) => { /* ... */ });

// 3. Check if event listener correctly bound
webview.addEventListener('ipc-message', (event) => {
  console.log('Listener attached, received:', event.channel);
});
```

### Issue 2: Data Serialization Failed

```javascript
// ❌ Wrong: Sending non-serializable data
ipcRenderer.send('channel', {
  element: document.querySelector('div'), // DOM elements cannot be serialized
  func: () => {}                          // Functions cannot be serialized
});

// ✅ Correct: Only send serializable data
ipcRenderer.send('channel', {
  html: document.querySelector('div').innerHTML,
  data: 'string or number or object'
});
```

### Issue 3: Message Sent at Wrong Time

```javascript
// ❌ Wrong: Sending message before preload loaded
const webview = document.getElementById('myWebview');
webview.send('early-message', data); // May fail

// ✅ Correct: Wait for did-finish-load
webview.addEventListener('did-finish-load', () => {
  webview.send('ready-message', data);
});
```

### Issue 4: Memory Leak (Not Removing Listeners)

```javascript
// ❌ Wrong: Repeatedly adding listeners
function setupWebView() {
  webview.addEventListener('ipc-message', handler);
}
setupWebView();
setupWebView(); // handler will be called twice!

// ✅ Correct: Remove old listener or use reference
function setupWebView() {
  webview.removeEventListener('ipc-message', handler);
  webview.addEventListener('ipc-message', handler);
}

// Or
let unsubscribe;

function setupWebView() {
  if (unsubscribe) unsubscribe();

  webview.addEventListener('ipc-message', handler);
  unsubscribe = () => {
    webview.removeEventListener('ipc-message', handler);
  };
}
```

## Complete IPC Communication Example

### Scenario: WebView Requests Data and Receives Updates

#### 1. Main Process (main.js)

```javascript
const { app, ipcMain } = require('electron');

// Mock data storage
let appData = {
  users: [],
  settings: {}
};

// Handle data fetch request
ipcMain.handle('get-data', async (event, params) => {
  console.log('[Main] get-data request:', params);

  // Mock async data fetching
  await new Promise(resolve => setTimeout(resolve, 100));

  return {
    success: true,
    data: appData[params.type]
  };
});

// Handle data save request
ipcMain.handle('save-data', async (event, params) => {
  console.log('[Main] save-data request:', params);

  appData[params.type] = params.data;

  // Notify all WebViews data updated
  BrowserWindow.getAllWindows().forEach(win => {
    win.webContents.send('data-updated', {
      type: params.type,
      data: params.data
    });
  });

  return { success: true };
});
```

#### 2. Preload Script (preload.js)

```javascript
const { contextBridge, ipcRenderer } = require('electron');

// Whitelist
const validInvokeChannels = ['get-data', 'save-data'];
const validReceiveChannels = ['data-updated'];

contextBridge.exposeInMainWorld('electronAPI', {
  // Get data
  getData: async (type) => {
    return await ipcRenderer.invoke('get-data', { type });
  },

  // Save data
  saveData: async (type, data) => {
    return await ipcRenderer.invoke('save-data', { type, data });
  },

  // Listen for data updates
  onDataUpdated: (callback) => {
    const handler = (event, data) => callback(data);
    ipcRenderer.on('data-updated', handler);

    // Return unsubscribe function
    return () => {
      ipcRenderer.removeListener('data-updated', handler);
    };
  }
});
```

#### 3. WebView Page (page.html)

```html
<!DOCTYPE html>
<html>
<head>
  <title>WebView Page</title>
</head>
<body>
  <h1>Data Manager</h1>
  <button id="loadBtn">Load Data</button>
  <button id="saveBtn">Save Data</button>
  <pre id="output"></pre>

  <script>
    const output = document.getElementById('output');

    // Load data
    document.getElementById('loadBtn').addEventListener('click', async () => {
      try {
        const result = await window.electronAPI.getData('users');
        output.textContent = JSON.stringify(result, null, 2);
      } catch (error) {
        output.textContent = 'Error: ' + error.message;
      }
    });

    // Save data
    document.getElementById('saveBtn').addEventListener('click', async () => {
      const newData = [
        { id: 1, name: 'Alice' },
        { id: 2, name: 'Bob' }
      ];

      try {
        const result = await window.electronAPI.saveData('users', newData);
        output.textContent = 'Saved: ' + JSON.stringify(result, null, 2);
      } catch (error) {
        output.textContent = 'Error: ' + error.message;
      }
    });

    // Listen for data updates
    const unsubscribe = window.electronAPI.onDataUpdated((data) => {
      console.log('Data updated:', data);
      output.textContent = 'Updated: ' + JSON.stringify(data, null, 2);
    });

    // Unsubscribe on page unload
    window.addEventListener('beforeunload', () => {
      unsubscribe();
    });
  </script>
</body>
</html>
```

#### 4. Main Window Renderer Process (renderer.js)

```javascript
const webview = document.getElementById('myWebview');

// Set preload script
webview.setAttribute('preload', './preload.js');

// Load page
webview.setAttribute('src', './page.html');

// Listen for WebView IPC messages (if needed)
webview.addEventListener('ipc-message', (event) => {
  console.log('[Renderer] IPC message from WebView:', event.channel, event.args);
});
```

## Performance Optimization

### Reduce IPC Call Frequency

```javascript
// ❌ Not good: Frequent calls
setInterval(() => {
  window.electronAPI.saveData('position', getCurrentPosition());
}, 100); // Save every 100ms

// ✅ Better: Batch processing
let positionBuffer = [];
let saveTimer = null;

function queueSave(position) {
  positionBuffer.push(position);

  if (saveTimer) clearTimeout(saveTimer);

  saveTimer = setTimeout(async () => {
    await window.electronAPI.saveData('positions', positionBuffer);
    positionBuffer = [];
  }, 1000); // Batch save after 1 second
}

setInterval(() => {
  queueSave(getCurrentPosition());
}, 100);
```

### Use Transferable Objects (if applicable)

For large data transfers, consider using SharedArrayBuffer or other optimization methods.

## Troubleshooting Checklist

### IPC Communication Debugging Checklist

- [ ] Confirm preload script path is correct
- [ ] Verify contextBridge API is correctly exposed
- [ ] Check channel names match on send and receive sides
- [ ] Confirm message sent timing (after did-finish-load)
- [ ] Verify sent data can be serialized
- [ ] Check for listener leaks
- [ ] Check console for error messages
- [ ] Use IPC debugging tools to log message flow

### Debug Commands

```javascript
// Check if electronAPI exposed
webview.executeJavaScript('typeof window.electronAPI')
  .then(console.log);

// Test simple IPC call
webview.executeJavaScript(`
  window.electronAPI.getData('test')
    .then(result => console.log('IPC test result:', result))
    .catch(err => console.error('IPC test error:', err));
`);
```

## Security Considerations

### ⚠️ IPC Security Rules

1. **Always use whitelist to validate channels**
   ```javascript
   const validChannels = ['get-data', 'save-data'];
   if (!validChannels.includes(channel)) {
     throw new Error('Invalid channel');
   }
   ```

2. **Don't directly expose ipcRenderer**
   ```javascript
   // ❌ Dangerous
   contextBridge.exposeInMainWorld('ipc', ipcRenderer);

   // ✅ Safe
   contextBridge.exposeInMainWorld('api', {
     getData: () => ipcRenderer.invoke('get-data')
   });
   ```

3. **Validate and sanitize input data**
   ```javascript
   ipcMain.handle('save-data', async (event, params) => {
     // Validate input
     if (!params || typeof params.type !== 'string') {
       throw new Error('Invalid parameters');
     }

     // Sanitize input
     const sanitizedData = sanitize(params.data);

     // Process...
   });
   ```

4. **Limit callable functionality**
   ```javascript
   // Only expose necessary functionality
   contextBridge.exposeInMainWorld('electronAPI', {
     // ✅ Restricted file reading
     readConfig: () => ipcRenderer.invoke('read-config'),

     // ❌ Don't expose arbitrary file reading
     // readFile: (path) => ipcRenderer.invoke('read-file', path)
   });
   ```

## Recommended Tools and Libraries

- **electron-log** - Enhanced logging
- **electron-store** - Persistent storage (reduce IPC calls)
- **typed-emitter** - Type-safe event emitters
- **RxJS** - Reactive programming library, suitable for complex message streams
