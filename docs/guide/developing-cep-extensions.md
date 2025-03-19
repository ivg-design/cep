# Developing Adobe CEP Extensions for After Effects – Technical Guide

## Table of Contents

1. [Development Setup](#1-development-setup)  
   - [Prerequisites](#prerequisites)
   - [Folder Structure](#folder-structure)
   - [manifest.xml Configuration](#manifestxml-configuration)
   - [Installation & Configuration](#installation--configuration)
2. [Debugging CEP Extensions](#2-debugging-cep-extensions)  
   - [Client-Side Debugging (Panel UI)](#client-side-debugging-panel-ui)
   - [Server-Side Debugging (ExtendScript)](#server-side-debugging-extendscript)
   - [Troubleshooting Debugging](#troubleshooting-debugging)
3. [Best Practices for Efficient CEP Development](#3-best-practices-for-efficient-cep-development)  
   - [Minimize Host Round-Trips](#minimize-host-round-trips)
   - [Optimize ExtendScript Execution](#optimize-extendscript-execution)
   - [Efficient UI Updates](#efficient-ui-updates)
   - [Common Pitfalls (and Solutions)](#common-pitfalls-and-solutions)
   - [Performance Optimizations Summary](#performance-optimizations-summary)
   - [Security Considerations](#security-considerations)
4. [Hyperbrew BOLT-CEP Template](#4-hyperbrew-bolt-cep-template)  
   - [Overview & Features](#overview--features)
   - [Installation & Usage Guide](#installation--usage-guide)
   - [Real-World Examples](#real-world-examples)
   - [Common Issues and How to Resolve Them](#common-issues-and-how-to-resolve-them)
5. [Alternative CEP Development Frameworks & Tools](#5-alternative-cep-development-frameworks--tools)  
   - [Battle Axe's Brutalism (with Bombino)](#battle-axes-brutalism-with-bombino)
   - [CEP + Parcel (cep-bundler / fusepilot)](#cep--parcel-cep-bundler--fusepilot)
   - [Yeoman Generators (CEP)](#yeoman-generators-cep)
   - [CEP Extension Toolkit / Legacy Methods](#cep-extension-toolkit--legacy-methods)
   - [Other Tools](#other-tools)
6. [Advanced Topics](#6-advanced-topics)  
   - [Automation & CI/CD for CEP Extensions](#automation--cicd-for-cep-extensions)
   - [Distributing & Updating CEP Extensions](#distributing--updating-cep-extensions)
   - [UXP vs. CEP (Future-Proofing Your Extensions)](#uxp-vs-cep-future-proofing-your-extensions)

---

> **Note**: This guide complements the existing documentation in the `/docs/guide` directory. For specific topics, refer to:
> - [Getting Started Guide](getting-started.md) - Basic setup and first extension
> - [Debugging Guide](debugging.md) - Detailed debugging workflows
> - [Network Requests](network-requests.md) - Making HTTP requests from CEP
> - [Exporting Files](exporting-files.md) - File I/O operations

## 1. Development Setup

### Prerequisites

To develop Adobe CEP extensions for After Effects, ensure you have the following: 

- **Adobe After Effects (CC 2014 or later)** – CEP (Common Extensibility Platform) panels are supported in After Effects CC 2014 and above. After Effects 2022/2023 use CEP 11 (Chromium 88, Node 15) under the hood, and both Windows and macOS (Intel & Apple Silicon) are supported (on Apple Silicon, CEP panels run natively in AE).

- **Enable Debug Mode** – During development, allow unsigned extensions to load. Set the Player Debug Mode flag:
  - Windows: Add a reg key `PlayerDebugMode`=`1` under `HKEY_CURRENT_USER/Software/Adobe/CSXS.<version>` (e.g. CSXS.10 for CEP 10)
  - macOS: Run: `defaults write com.adobe.CSXS.<version> PlayerDebugMode 1`
  
  > **Note**: Use the CEP version matching your AE release (AE 2022+ uses CSXS.11). After enabling, **restart** After Effects. On macOS, you may need to kill the `cfprefsd` process or log out/in for the change to take effect.
  
  For more details on debug mode setup, see our [Debugging Guide](debugging.md#set-the-debug-mode).

- **Node.js and NPM/Yarn** – Install Node.js (v16+ recommended) if you plan to use modern toolchains or frameworks (React, Vue, TypeScript, etc.). Yarn Classic (v1.x) is often required by certain CEP build tools (e.g. BOLT-CEP).

- **Code Editor / IDE** – Use an editor like **Visual Studio Code** for coding HTML/CSS/JS and ExtendScript. Install the **ExtendScript Debugger** extension for VSCode to debug host scripts (ExtendScript) within After Effects.

- **CEP Resources** – Download the Adobe CEP SDK/Resources which include the **CSInterface.js** library and sample code. The CSInterface JavaScript library simplifies communication between the panel (JS) and After Effects (ExtendScript). In many setups, this is included or bundled, but be aware of it for manual projects.

- **After Effects Scripting Permissions** – In After Effects, enable **"Allow Scripts to Write Files and Access Network"** (Under *Edit > Preferences > Scripting & Expressions*). This allows your extension's scripts to access the file system and network if needed.

For a more detailed setup walkthrough, see our [Getting Started Guide](getting-started.md#prerequisites).

### Folder Structure

A CEP extension is a folder containing your panel's files and a manifest. The **minimum required** structure is: a `CSXS` folder with a `manifest.xml`, and your HTML/JS/CSS and ExtendScript code files. A recommended structure is to separate "client" (panel UI) and "host" (ExtendScript) code:

```
my-extension/
├── CSXS/
│   └── manifest.xml
├── client/
│   ├── index.html
│   ├── style.css
│   ├── index.js
│   └── lib/
│       └── CSInterface.js
└── host/
    └── index.jsx
```

- **`CSXS/manifest.xml`** – Extension manifest (XML format) defining the extension's identity, supported host apps/versions, and panel settings. *(This file is required; without it, After Effects will not recognize your extension.)*

- **`client/`** – Front-end assets (HTML, CSS, JS). For example: `index.html`, `style.css`, `index.js`, and the `CSInterface.js` library. This is the code that runs in a Chromium Embedded browser inside After Effects.

- **`host/`** – Host scripts (ExtendScript `.jsx` files) that run in After Effects. For example: `index.jsx` containing ExtendScript code to be invoked for AE-specific functionality.

This structure cleanly separates UI code from scripting logic. However, the structure is flexible as long as the manifest paths are configured correctly. You may choose a different layout (some frameworks bundle everything under one `dist` folder), but ensure the manifest's references to files are updated accordingly.

For more details on file organization and best practices, see our [Getting Started Guide](getting-started.md#1-decide-the-folder-structure).

### manifest.xml Configuration

The `CSXS/manifest.xml` is the heart of the extension configuration. At minimum, it must specify:
- An Extension Bundle ID
- An Extension ID
- Target Host Application(s) with version range
- Required CEP runtime version
- Entry points (HTML and ExtendScript files)

Key fields in **manifest.xml** for an After Effects CEP panel include:

- **ExtensionBundleId** – A unique reverse-domain identifier for your extension bundle (e.g. `com.mycompany.myextension`).

- **Extension Id** – A unique ID for the specific extension (panel). Often this is `ExtensionBundleId.panel` (e.g. `com.mycompany.myextension.panel`). This ID will appear twice (in ExtensionList and later in the `<Extension Id="...">` node).

- **Version** – Your extension version (e.g. `1.0.0`). Keep this in sync with your releases.

- **HostList** – The Adobe apps that will host this extension. Use the host's code and a version range. For After Effects, the host code is `"AEFT"`. For example:
  ```xml
  <Host Name="AEFT" Version="[16.0,99.0]"/>
  ```
  This indicates the panel is compatible with AE 16.0 and up (16.0 was CC 2019). Using a range or `"*"`, you can future-proof the manifest so the panel doesn't disappear when AE is updated.

- **RequiredRuntime** – Indicates the CEP runtime version required. For CEP 11 (used in AE 2022+), use:
  ```xml
  <RequiredRuntime Name="CSXS" Version="11.0"/>
  ```
  For broader compatibility you might use a slightly lower version (CEP 9 or 10), but ensure it's supported by your code.

- **MainPath** – The relative path to your panel's HTML file (e.g. `client/index.html`). This file is loaded in the panel UI.

- **ScriptPath** – The relative path to your ExtendScript file (e.g. `host/index.jsx`). This will be automatically loaded into After Effects when the panel opens.

- **CEFCommandLine** (optional) – Command-line switches for the embedded Chromium engine. For instance, you can enable Node.js, or allow remote debugging. By default, Node is enabled in CEP panels.

- **Extension UI Settings** – Within `<UI>`, configure how your panel appears:
  - `<Menu>My Extension</Menu>` (name in the Window > Extensions menu)
  - `<Geometry>` (panel default width/height)
  - `<Icons>` (icon path)
  - `<Language>` etc.
  For a panel, `Type="Panel"` is typical. Use `<AutoVisible>true</AutoVisible>` if you want the panel to auto-open when AE starts.

Below is a simplified example of a manifest for an After Effects panel:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ExtensionManifest ExtensionBundleId="com.mycompany.myextension" ExtensionBundleVersion="1.0.0" Version="11.0">
  <ExtensionList>
    <Extension Id="com.mycompany.myextension.panel" Version="1.0.0" />
  </ExtensionList>
  <ExecutionEnvironment>
    <HostList>
      <Host Name="AEFT" Version="[16.0,99.0]"/>
    </HostList>
    <LocaleList>
      <Locale Code="All"/>
    </LocaleList>
    <RequiredRuntimeList>
      <RequiredRuntime Name="CSXS" Version="11.0"/>
    </RequiredRuntimeList>
  </ExecutionEnvironment>
  <DispatchInfoList>
    <DispatchInfo>
      <UI>
        <Type>Panel</Type>
        <Menu>My Extension</Menu>
        <Geometry>
          <Size Width="300" Height="200"/>
        </Geometry>
      </UI>
      <Resources>
        <MainPath>client/index.html</MainPath>
        <ScriptPath>host/index.jsx</ScriptPath>
      </Resources>
    </DispatchInfo>
  </DispatchInfoList>
</ExtensionManifest>
```

This manifest would register a panel named "My Extension" under After Effects' **Window > Extensions** menu. It targets AE CC 2019 and later, requires CEP 11, and loads `client/index.html` with its ExtendScript in `host/index.jsx`.

For more detailed manifest options and examples, see our [Getting Started Guide](getting-started.md#2-configure-your-extension-in-manifestxml).

### Installation & Configuration

During development, you can load your extension by placing its folder in the appropriate extensions directory. The locations are: 

- **macOS:** 
  - User-level: `~/Library/Application Support/Adobe/CEP/extensions`
  - System-wide: `/Library/Application Support/Adobe/CEP/extensions`

- **Windows:** 
  - User-level: `%USERPROFILE%\AppData\Roaming\Adobe\CEP\extensions`
  - System-wide: `%ProgramFiles(x86)%\Common Files\Adobe\CEP\extensions`

For development, the **user-level** path is recommended (no admin rights needed). Create the `CEP/extensions` folder if it doesn't exist. Copy your extension folder (containing the `CSXS` subfolder) into this directory. After Effects will scan this on launch to populate the Extensions menu.

> **Note**: For more details on installation paths and troubleshooting, see our [Getting Started Guide](getting-started.md#6-launch-your-extension-in-the-host-app).

**Debug mode:** As noted in Prerequisites, ensure you've enabled PlayerDebugMode=1 on your system. If not, After Effects will refuse to load your extension with an error about it not being properly signed. When debug mode is on, you can ignore signature warnings during development.

Now launch After Effects. You should find your extension under **Window > Extensions** (in newer versions it might be under "Extensions (Legacy)" since CEP is the legacy platform). Click your extension's name to open the panel. If everything is configured correctly, a panel window will appear (it might be blank if you haven't added UI yet). 

If you **don't see your extension** in the menu, double-check the following common issues:

1. The `manifest.xml` is in a `CSXS` folder at the root of your extension folder. The extension folder name can be anything, but it should match what you placed in the extensions directory.

2. The manifest's Host entry for After Effects covers your AE version. For example, if your manifest only has `Version="16.0"` and you're on After Effects 2023 (version 23.x), it will not show up. Using a range like `[16.0,99.0]` or `"*"` ensures it includes newer versions.

3. PlayerDebugMode is truly set (on macOS, remember to restart or kill prefs cache as mentioned). If not set, AE will list the extension but show an error on opening, or it may not list it at all if unsigned.

4. The extension is in the correct directory. On Windows, note the distinction between `%AppData%` (user Roaming) vs. `%ProgramFiles(x86)%` (system). On macOS, ensure it's in your user Library if you didn't install system-wide.

For detailed troubleshooting steps and common issues, see our [Debugging Guide](debugging.md#troubleshooting-common-issues).

## 2. Debugging CEP Extensions

During development, robust debugging is critical. You'll typically debug two contexts:
- **Client-side**: The panel's HTML/JS running in Chrome Embedded Framework
- **Server-side**: The ExtendScript running in After Effects

For comprehensive debugging workflows, see our dedicated [Debugging Guide](debugging.md). Here's a quick overview of the key concepts:

### Client-Side Debugging (Panel UI)

CEP panels run on an embedded Chromium browser engine, so you can use Chrome DevTools to debug HTML, CSS, and JavaScript. To set this up:

1. Create a `.debug` file in your extension root with your panel's debug configuration
2. Enable remote debugging in Chrome
3. Use Chrome DevTools for live debugging, console logs, and network monitoring

For step-by-step instructions and troubleshooting, see [Client Side Debugging](debugging.md#client-side-debugging-panel-ui).

### Server-Side Debugging (ExtendScript)

For debugging the ExtendScript (JSX) that runs in After Effects:

1. Install the Adobe ExtendScript Debugger extension in VS Code
2. Set up a launch configuration for After Effects
3. Use breakpoints and step-through debugging in your JSX files

For detailed ExtendScript debugging setup, see [Server Side Debugging](debugging.md#server-side-debugging-extendscript).

### Troubleshooting Debugging

Common debugging issues include:
- Panel UI not loading (gray panel)
- DevTools connection problems
- ExtendScript debugger attachment issues

See our [Troubleshooting Guide](debugging.md#troubleshooting-common-issues) for solutions to these and other debugging challenges.

## 3. Best Practices for Efficient CEP Development

Developing CEP extensions for After Effects involves both web development and scripting. Here are best practices and common pitfalls to keep your extension efficient, stable, and secure.

### Minimize Host Round-Trips

One of the biggest performance considerations is the communication between the panel (JS) and After Effects (ExtendScript). Each call to `CSInterface.evalScript()` invokes After Effects to run some ExtendScript and possibly return a result. These calls are relatively slow (millisecond-scale) and can block the UI.

> **Important**: `CSInterface.evalScript()` is essentially synchronous – it will block the JavaScript thread until the ExtendScript finishes (even if you use a callback, the UI thread in CEP is halted while the host app runs the script).

To reduce the number of evalScript calls:

1. **Batch operations:** If you need several pieces of data from AE, retrieve them in one JavaScript call:
   ```javascript
   // Instead of this (two round-trips):
   evalScript("getCompName()")
   evalScript("getCompDuration()")
   
   // Do this (one round-trip):
   evalScript("getCompInfo()") // Returns JSON with both name and duration
   ```

2. **Avoid tight loops calling ExtendScript:** If you have a loop in JS that calls an ExtendScript function each iteration, that will be very slow. Instead:
   ```javascript
   // Instead of this:
   layers.forEach(layer => {
     evalScript(`processLayer("${layer}")`)
   })
   
   // Do this:
   evalScript(`processLayers(${JSON.stringify(layers)})`)
   ```

3. **Use Callbacks or Promises** for asynchronous flows:
   ```javascript
   // Using callback
   CSInterface.evalScript('getProjectData()', result => {
     // Handle result when ready
   })
   
   // Using Promise wrapper
   function evalScriptPromise(script) {
     return new Promise((resolve, reject) => {
       CSInterface.evalScript(script, result => {
         if (result === 'EvalScript error') reject(new Error(result))
         else resolve(result)
       })
     })
   }
   ```

For more examples of efficient communication patterns, see our [Network Requests Guide](network-requests.md).

### Optimize ExtendScript Execution

Long-running ExtendScript code will freeze After Effects (and by extension, your panel) until it completes. To optimize:

1. **Use `app.scheduleTask()`** for long operations:
   ```javascript
   // Process items in chunks
   function processChunk(items, startIndex, chunkSize) {
     for (var i = startIndex; i < startIndex + chunkSize && i < items.length; i++) {
       // Process item
     }
     if (startIndex + chunkSize < items.length) {
       app.scheduleTask('processChunk(items, ' + (startIndex + chunkSize) + ', ' + chunkSize + ')', 100)
     }
   }
   ```

2. **Leverage AE's native functions** whenever possible:
   ```javascript
   // Instead of pixel-by-pixel manipulation in ExtendScript (slow)
   // Use AE's built-in effects and methods
   ```

3. **Use persistent ExtendScript engine** for caching:
   ```javascript
   // In your JSX file
   #targetengine "myPersistentEngine"
   
   // Cache expensive computations
   var cachedData = null
   function getData() {
     if (!cachedData) {
       cachedData = expensiveComputation()
     }
     return cachedData
   }
   ```

For more ExtendScript optimization techniques, see our [Getting Started Guide](getting-started.md#5-write-your-extendscript-code).

### Efficient UI Updates

The panel's UI is essentially a webpage. Follow these best practices:

1. **Throttle frequent updates:**
   ```javascript
   let throttleTimer
   function updateStatus(text) {
     if (throttleTimer) return
     throttleTimer = setTimeout(() => {
       document.getElementById('status').textContent = text
       throttleTimer = null
     }, 100) // Update max 10 times per second
   }
   ```

2. **Virtual DOM / Frameworks:** If using React, Vue, or Svelte (via BOLT-CEP or other frameworks), leverage their optimizations.

3. **Caching in UI:**
   ```javascript
   // Cache expensive data
   const projectDataCache = {
     data: null,
     timestamp: 0,
     maxAge: 5000, // 5 seconds
     
     async get() {
       if (!this.data || Date.now() - this.timestamp > this.maxAge) {
         this.data = await evalScriptPromise('getProjectData()')
         this.timestamp = Date.now()
       }
       return this.data
     }
   }
   ```

4. **Idle timing for updates:**
   ```javascript
   // Check project state on idle
   let idleCallback
   function scheduleProjectCheck() {
     if (idleCallback) return
     idleCallback = requestIdleCallback(() => {
       checkProjectState()
       idleCallback = null
     })
   }
   ```

For more UI optimization techniques, see our [Getting Started Guide](getting-started.md#4-write-your-front-end-code).

### Common Pitfalls (and Solutions)

Here are common issues you might encounter and how to solve them:

1. **Extension not appearing in menu:**
   ```
   Problem: Extension doesn't show up in Window > Extensions menu
   Solution: Check manifest version range and installation path
   ```
   See [Getting Started Guide](getting-started.md#6-launch-your-extension-in-the-host-app) for detailed troubleshooting.

2. **"Extension could not be loaded because it was not properly signed" error:**
   ```
   Problem: Extension fails to load with signing error
   Solution: Enable PlayerDebugMode=1 for development
   For distribution: Sign the extension with a certificate
   ```
   See [Debugging Guide](debugging.md#getting-a-not-properly-signed-alert) for details.

3. **`app` or other AE objects undefined in JS:**
   ```javascript
   // Wrong: This won't work in panel JS
   app.project.activeItem
   
   // Correct: Use evalScript to access AE objects
   CSInterface.evalScript('app.project.activeItem.name', name => {
     console.log(name)
   })
   ```

4. **Scope and lifetime of ExtendScript code:**
   ```javascript
   // In JSX file
   #targetengine "main"
   
   // Variables persist in engine while it lives
   var persistentData = {}
   
   // But are gone if AE closes/reloads
   // Solution: Re-initialize on panel launch
   function initializeData() {
     if (!persistentData.initialized) {
       persistentData = { initialized: true, /* ... */ }
     }
   }
   ```

5. **Cross-platform issues:**
   ```javascript
   // Wrong: Direct path concatenation
   const path = folder + "\\" + filename
   
   // Correct: Use File API
   const file = new File(folder + "/" + filename) // Forward slash works on both platforms
   // Or better:
   const file = new File(folder).fsName + new File().fs.getSeparator() + filename
   ```

For more troubleshooting tips, see our [Debugging Guide](debugging.md#troubleshooting-common-issues).

### Performance Optimizations Summary

Key points for maintaining good performance:

1. **Minimize JS-ExtendScript Communication:**
   - Batch operations into fewer `evalScript` calls
   - Cache results when possible
   - Use asynchronous patterns for better UI responsiveness

2. **Optimize ExtendScript:**
   - Break up long operations with `app.scheduleTask`
   - Use native AE functions over custom implementations
   - Cache expensive computations in persistent engine

3. **Efficient UI Updates:**
   - Throttle frequent updates
   - Use modern frameworks' optimizations
   - Implement smart caching
   - Use idle callbacks for background tasks

For detailed performance tips, see our [Network Requests Guide](network-requests.md#optimizing-performance).

### Security Considerations

CEP extensions run with high privileges on a user's machine. Follow these security practices:

1. **Code Signing:**
   ```bash
   # Sign your extension for distribution
   ZXPSignCmd -sign <inputDir> <output.zxp> <p12Certificate> <p12Password>
   
   # Verify the signature
   ZXPSignCmd -verify <output.zxp>
   ```

2. **HTTPS and Network Security:**
   ```javascript
   // Always use HTTPS for external requests
   fetch('https://api.example.com/data', {
     // ... options
   })
   
   // Handle network errors
   try {
     const response = await fetch(url)
     if (!response.ok) throw new Error(`HTTP error: ${response.status}`)
   } catch (error) {
     console.error('Network request failed:', error)
   }
   ```

3. **Content Security Policy:**
   ```html
   <!-- In your panel's HTML -->
   <meta http-equiv="Content-Security-Policy" content="default-src 'self' https:; script-src 'self' 'unsafe-inline' 'unsafe-eval'; style-src 'self' 'unsafe-inline';">
   ```

4. **Node.js Security:**
   ```javascript
   // Be careful with file system operations
   const fs = require('fs')
   
   // Always validate paths
   const path = require('path')
   const safePath = path.normalize(userProvidedPath)
   if (safePath.startsWith(basePath)) {
     // Safe to proceed
   }
   ```

5. **Prevent Script Injection:**
   ```javascript
   // Wrong: Direct string concatenation
   evalScript('alert("' + userInput + '")')
   
   // Correct: Proper escaping
   evalScript(`alert("${userInput.replace(/["\\]/g, '\\$&')}")`)
   
   // Better: Use JSON for data passing
   evalScript(`processData(${JSON.stringify(userInput)})`)
   ```

For more security best practices, see our [Network Requests Guide](network-requests.md#security-considerations).

## 4. Hyperbrew BOLT-CEP Template

[BOLT-CEP](https://github.com/hyperbrew/bolt-cep) is a modern boilerplate for building CEP extensions, aimed at making development faster and easier. It integrates a robust build system with support for popular front-end frameworks and streamlines the CEP setup.

### Overview & Features

1. **Lightning-Fast HMR (Hot Module Replacement):**
   ```bash
   # Start development server with HMR
   yarn dev
   ```
   Your panel will live-reload instantly as you save changes, without needing to manually refresh or reload the extension.

2. **Multi-Framework UI Support:**
   ```bash
   # Create new project with your preferred framework
   yarn create bolt-cep my-extension --template react
   # Or
   yarn create bolt-cep my-extension --template vue
   # Or
   yarn create bolt-cep my-extension --template svelte
   ```

3. **Modern JavaScript in Both Layers:**
   ```typescript
   // In your panel (TypeScript)
   interface CompInfo {
     name: string
     duration: number
   }
   
   // In ExtendScript (also TypeScript)
   function getCompInfo(): CompInfo {
     const comp = app.project.activeItem
     return {
       name: comp.name,
       duration: comp.duration
     }
   }
   ```

4. **Optimized Bundling:**
   ```bash
   # Build for production
   yarn build
   
   # Create signed ZXP
   yarn zxp
   ```

### Installation & Usage Guide

1. **Create a New Project:**
   ```bash
   # Using yarn
   yarn create bolt-cep my-extension
   
   # Using npm
   npm init bolt-cep my-extension
   ```

2. **Install Dependencies:**
   ```bash
   cd my-extension
   yarn
   ```

3. **Configure Your Extension:**
   ```typescript
   // cep.config.ts
   export default {
     bundleId: 'com.mycompany.myextension',
     hostList: {
       AEFT: {
         version: '[16.0,99.9]'  // AE CC 2019 and up
       }
     },
     panels: {
       main: {
         name: 'My Extension',
         width: 500,
         height: 600
       }
     }
   }
   ```

4. **Development Workflow:**
   ```bash
   # Start dev server
   yarn dev
   
   # Your extension will be available in AE
   # Changes to files trigger instant updates
   ```

5. **Building for Production:**
   ```bash
   # Create production build
   yarn build
   
   # Package as ZXP
   yarn zxp
   
   # Create distribution zip
   yarn zip
   ```

For more details on project setup, see our [Getting Started Guide](getting-started.md).

### Real-World Examples

BOLT-CEP is used in production by various tools:

1. **Battle Axe's Rubberhose 3:**
   ```typescript
   // Example of BOLT-CEP with React and TypeScript
   import { useEffect, useState } from 'react'
   import { evalTS } from '@bolt-cep/core'
   
   function RiggingPanel() {
     const [bones, setBones] = useState([])
     
     useEffect(() => {
       evalTS('getBones()').then(setBones)
     }, [])
     
     return (
       <div className="panel">
         {bones.map(bone => (
           <BoneControl key={bone.id} data={bone} />
         ))}
       </div>
     )
   }
   ```

2. **Klutz GPT:**
   ```typescript
   // Example of BOLT-CEP with Vue
   <template>
     <div class="ai-panel">
       <prompt-input @submit="generateAnimation" />
       <results-view :animations="results" />
     </div>
   </template>
   
   <script setup>
   import { ref } from 'vue'
   import { evalTS } from '@bolt-cep/core'
   
   const results = ref([])
   
   async function generateAnimation(prompt) {
     const animation = await evalTS('createFromPrompt', { prompt })
     results.value.push(animation)
   }
   </script>
   ```

### Common Issues and How to Resolve Them

1. **Apple Silicon Build Issues:**
   ```bash
   # Run Node in x64 mode for JSXBIN support
   arch -x86_64 yarn build
   
   # Or disable JSXBIN in config
   # cep.config.ts
   export default {
     jsxBin: 'off'  // For development
   }
   ```

2. **Yarn vs npm:**
   ```bash
   # Ensure using Yarn Classic
   yarn set version classic
   
   # Check version
   yarn --version  # Should be 1.x
   ```

3. **Multiple Adobe Apps:**
   ```typescript
   // cep.config.ts
   export default {
     hostList: {
       AEFT: { version: '[16.0,99.9]' },
       PHXS: { version: '[20.0,99.9]' }
     }
   }
   ```

For more troubleshooting tips, see our [Debugging Guide](debugging.md).

## 5. Alternative CEP Development Frameworks & Tools

While BOLT-CEP is a powerful all-in-one solution, there are other frameworks and tools to consider for CEP extension development. Each has its own philosophy and tooling.

### Battle Axe's Brutalism (with Bombino)

[Brutalism](https://github.com/battleaxedotco/brutalism) is a component-based UI framework specifically designed for Adobe panels. It provides pre-styled components that mimic Adobe's UI.

**Key Features:**
```javascript
// Example Brutalism component usage
<template>
  <div class="panel">
    <Button @click="doThing">Action</Button>
    <Dropzone @drop="handleDrop" />
    <ColorPicker v-model="color" />
  </div>
</template>

<script>
import { Button, Dropzone, ColorPicker } from 'brutalism'

export default {
  components: { Button, Dropzone, ColorPicker },
  data: () => ({
    color: '#FF0000'
  }),
  methods: {
    doThing() {
      // Automatically handles theme/scaling
    }
  }
}
</script>
```

**Comparison to Bolt:**
- Brutalism is a *UI framework*, Bolt is a *build system*
- Brutalism assumes Vue.js usage
- Can be used together (Bolt for build, Brutalism for UI)

### CEP + Parcel (cep-bundler / fusepilot)

For a simpler build setup, some tools use Parcel bundler:

```bash
# Using cep-bundler
npm install @adobe-extension-tools/cep-bundler

# Example config
{
  "name": "my-extension",
  "version": "1.0.0",
  "scripts": {
    "build": "cep-bundler build",
    "dev": "cep-bundler watch"
  }
}
```

**Key Features:**
- Zero configuration bundling
- Automatic dependency handling
- Live reload support

**Comparison to Bolt:**
- Simpler setup but fewer features
- Manual manifest management
- No integrated TypeScript for JSX

### Yeoman Generators (CEP)

Older but still functional scaffolding tools:

```bash
# Install Yeoman and a CEP generator
npm install -g yo generator-cep-vue-cli

# Generate a new extension
yo cep-vue-cli

# Structure created:
my-extension/
├── CSXS/
├── src/
│   ├── components/
│   └── main.js
└── jsx/
    └── main.jsx
```

**Comparison to Bolt:**
- Older build tools (Webpack 4, etc.)
- Basic live-reload (no HMR)
- Manual debug setup required

### CEP Extension Toolkit / Legacy Methods

The traditional way of building CEP extensions:

```html
<!-- Direct HTML/JS approach -->
<!DOCTYPE html>
<html>
<head>
  <script src="CSInterface.js"></script>
  <script src="jquery.js"></script>
</head>
<body>
  <button onclick="doThing()">Click Me</button>
  <script>
    function doThing() {
      var csInterface = new CSInterface()
      csInterface.evalScript('app.project.activeItem.name')
    }
  </script>
</body>
</html>
```

**Comparison to Modern Tools:**
- No build step (direct browser loading)
- Manual dependency management
- No hot reload or modern JS features

### Other Tools

1. **CEP-Sparker:**
   ```bash
   # Quick CEP project setup
   npx cep-sparker init my-extension
   ```

2. **Bombino (Standalone):**
   ```bash
   # Install Bombino CLI
   npm i -g bombino
   
   # Create extension from template
   bombino create my-extension --template quasar
   ```

3. **Adobe UXP (Future):**
   ```javascript
   // Modern UXP API (different from CEP)
   import { core, app } from 'uxp'
   
   // Different manifest format
   {
     "id": "my.extension",
     "name": "My Extension",
     "version": "1.0.0",
     "host": {
       "app": "AE",
       "minVersion": "22.0.0"
     }
   }
   ```

For a comparison of features across frameworks:

| Feature | BOLT-CEP | Brutalism | Parcel | Yeoman |
|---------|----------|-----------|---------|---------|
| Hot Reload | ✅ HMR | ✅ Basic | ✅ Basic | ❌ |
| TypeScript | ✅ Full | ✅ Vue | ❌ | ❌ |
| UI Components | ❌ | ✅ Adobe-style | ❌ | ❌ |
| Framework Choice | ✅ Multiple | Vue only | ✅ Any | Generator specific |
| Build Tools | Vite | Vue CLI | Parcel | Varies |
| Debug Setup | ✅ Automatic | Manual | Manual | Manual |

For more details on choosing a framework, see our [Getting Started Guide](getting-started.md#technology-used).

## 6. Advanced Topics

### Automation & CI/CD for CEP Extensions

Automate your build and packaging process using continuous integration:

1. **Scripting the Build:**
   ```bash
   # Example build script (package.json)
   {
     "scripts": {
       "build": "vite build",
       "sign": "zxpsignCmd -sign ./dist ./dist/MyExtension.zxp certificate.p12",
       "verify": "zxpsignCmd -verify ./dist/MyExtension.zxp",
       "release": "npm run build && npm run sign && npm run verify"
     }
   }
   ```

2. **GitHub Actions Workflow:**
   ```yaml
   # .github/workflows/release.yml
   name: Release
   on:
     release:
       types: [created]
   
   jobs:
     build:
       runs-on: ubuntu-latest
       steps:
         - uses: actions/checkout@v2
         - uses: actions/setup-node@v2
         
         - name: Install dependencies
           run: yarn install
           
         - name: Build extension
           run: yarn build
           
         - name: Sign ZXP
           run: |
             echo "${{ secrets.CERTIFICATE_B64 }}" | base64 -d > cert.p12
             yarn sign
           env:
             CERT_PASSWORD: ${{ secrets.CERT_PASSWORD }}
             
         - name: Upload artifact
           uses: actions/upload-release-asset@v1
           with:
             upload_url: ${{ github.event.release.upload_url }}
             asset_path: ./dist/MyExtension.zxp
             asset_name: MyExtension.zxp
             asset_content_type: application/octet-stream
   ```

3. **Automated Testing:**
   ```javascript
   // Example ExtendScript test
   #target aftereffects
   
   function testCreateComp() {
     var comp = app.project.items.addComp('Test', 1920, 1080, 1, 10, 30)
     if (!comp) throw new Error('Failed to create comp')
     return 'Pass'
   }
   
   // Run tests
   try {
     testCreateComp()
     $.write('Tests passed\n')
   } catch (e) {
     $.write('Test failed: ' + e.message + '\n')
   }
   ```

### Distributing & Updating CEP Extensions

Multiple distribution options are available:

1. **Adobe Exchange:**
   ```json
   // exchange.json
   {
     "products": [{
       "name": "My Extension",
       "version": "1.0.0",
       "host": "AE",
       "hostMinVersion": "16.0",
       "hostMaxVersion": "99.9",
       "downloadURL": "https://example.com/download/MyExtension.zxp",
       "updateURL": "https://example.com/updates/check.json"
     }]
   }
   ```

2. **Self-hosted Distribution:**
   ```javascript
   // In your panel, check for updates
   async function checkForUpdates() {
     const response = await fetch('https://example.com/updates/check.json')
     const { version } = await response.json()
     
     if (version > currentVersion) {
       // Show update notification
       const updateURL = `https://example.com/download/${version}/MyExtension.zxp`
       CSInterface.openURLInDefaultBrowser(updateURL)
     }
   }
   ```

3. **Enterprise Distribution:**
   ```powershell
   # Windows deployment script
   $extensionPath = "$env:ProgramFiles\Common Files\Adobe\CEP\extensions"
   Copy-Item ".\MyExtension" $extensionPath -Recurse
   
   # Enable debug mode for testing
   New-ItemProperty -Path "HKCU:\Software\Adobe\CSXS.11" -Name "PlayerDebugMode" -Value "1" -PropertyType String
   ```

### UXP vs. CEP (Future-Proofing Your Extensions)

Adobe is transitioning to UXP (Unified Extensibility Platform) for future extensions. Here's how to prepare:

1. **Architectural Considerations:**
   ```javascript
   // CEP approach (current)
   const csInterface = new CSInterface()
   csInterface.evalScript('app.project.activeItem.name', name => {
     console.log(name)
   })
   
   // UXP approach (future)
   import { app } from 'uxp'
   const name = await app.activeDocument.name
   console.log(name)
   ```

2. **Code Organization for Future Migration:**
   ```typescript
   // Separate business logic from platform-specific code
   // platform/cep.ts
   export async function getActiveItemName(): Promise<string> {
     return new Promise(resolve => {
       const csInterface = new CSInterface()
       csInterface.evalScript('app.project.activeItem.name', resolve)
     })
   }
   
   // platform/uxp.ts
   export async function getActiveItemName(): Promise<string> {
     const { app } = await import('uxp')
     return app.activeDocument.name
   }
   
   // Use in your components
   import { getActiveItemName } from './platform/cep'  // or './platform/uxp'
   ```

3. **Feature Parity Considerations:**
   ```javascript
   // CEP features that need alternatives in UXP
   
   // CEP: Direct file system access
   const fs = require('fs')
   fs.readFileSync('path/to/file')
   
   // UXP: File system access via APIs
   const fs = require('uxp').storage.localFileSystem
   const file = await fs.getFileForOpening()
   const contents = await file.read()
   ```

4. **Gradual Migration Strategy:**
   ```typescript
   // config.ts
   export const isUXP = Boolean(globalThis.require?.('uxp'))
   
   // Wrapper for platform-specific features
   export class PlatformBridge {
     static async getActiveItem() {
       if (isUXP) {
         const { app } = await import('uxp')
         return app.activeDocument
       } else {
         return new Promise(resolve => {
           const csInterface = new CSInterface()
           csInterface.evalScript('app.project.activeItem', resolve)
         })
       }
     }
   }
   ```

For more details on UXP migration, see Adobe's [UXP Documentation](https://www.adobe.io/photoshop/uxp/).

---

This concludes our comprehensive guide to developing Adobe CEP Extensions. For specific topics, refer to our other guides:
- [Getting Started Guide](getting-started.md) - Basic setup and first extension
- [Debugging Guide](debugging.md) - Detailed debugging workflows
- [Network Requests](network-requests.md) - Making HTTP requests from CEP
- [Exporting Files](exporting-files.md) - File I/O operations

For the latest updates and community support, visit the [Adobe CEP Resources](https://github.com/Adobe-CEP/CEP-Resources) repository. 