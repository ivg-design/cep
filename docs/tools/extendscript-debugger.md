---
title: ExtendScript Debugger
description: A VS Code Extension that enables debugging ExtendScript scripts and extensions in Adobe's ExtendScript-enabled applications.
category: Tools
---

# ExtendScript Debugger Extension for VS Code

A VS Code Extension that enables debugging ExtendScript scripts and extensions in Adobe's ExtendScript-enabled applications.

::: warning Apple Silicon Users
The extension does not run in native builds of VS Code but it does work with Intel/universal builds using Rosetta. Please see the [Known Issues](#known-issues) section for information on how to use the extension.
:::

## Features

### Supported Features
- Breakpoints
  - Conditional Breakpoints
  - Expression Condition
  - Hit Count
- Exception Breakpoints
  - Caught Exceptions
- Logpoints
- Variables View
  - Local and Global Scope
  - Modify variables
- Watch View
- Call Stack View
- Debug Actions
  - Continue / Pause
  - Step Over
  - Step Into
  - Step Out
  - Restart
  - Disconnect / Stop
- Debug Console
  - Expression Evaluation
  - Expression Evaluation of Code on Hover
  - Script Evaluation and Halting
- Export ExtendScript to JSXBin

::: note
Changes to the Caught Exceptions setting while a script is running or stopped at a breakpoint will only apply to scopes created after the setting is changed.
:::

::: note
Expression Evaluation of Code on Hover requires an active debug session.
:::

### Unsupported Features
- Profiling Support
- Object Model Viewer (OMV)
- Auto-Completion
- Scripts Panel

## Getting Started

### Installation
Install the extension through the usual means.

The extension requires VS Code v1.62 or newer.

### Migration from V1 Versions
The ExtendScript Debugger V2 is a complete rewrite of the V1 version. The internals received a complete overhaul that substantially increased stability, performance, flexibility, and improved compatibility with native VS Code features. As a result of this overhaul, the launch configuration properties have changed. The following is a table of properties that have been renamed:

| V1 | V2 |
|----|----|
| targetSpecifier | hostAppSpecifier |
| program | script |
| excludes | hiddenTypes |

The `engineName` and `debugLevel` properties remain unchanged. All other V1 properties have been removed and will be ignored. Please see the [Advanced Configuration](#advanced-configuration) section for a full listing of configuration properties available in V2.

Additionally, the manner in which the extension operates has changed dramatically. It is highly recommended that you read the following section as it provides an overview of how to use this extension with three common use cases. 

## Using the Debugger

This extension was designed to support a wide variety of use cases. Three common use cases are outlined below to provide guidance on how certain features may be used.

### Running a Script
The extension supports running (evaluating) a script in a host application without an active debug session. This functionality may be triggered via the `Evaluate Script in Host...` command or by clicking the `Eval in Adobe...` button.

### Debugging a Script
The most direct way to debug a script in the extension is to start a debug session configured with a launch request type. Such a debug session will:
1. Connect to the specified host engine
2. Inform it that the debugger is active
3. Trigger the script evaluation

Any breakpoints or uncaught exceptions will cause VS Code to show the debug state and enable interacting with the host application. Once the script evaluation is complete, the debug session will clean itself up and shut down.

### Debugging Event Callbacks
Debugging ExtendScript triggered via a callback (e.g. with ScriptUI or CEP) requires that the debugger be connected when that script is run. A launch debug session is only active when the originating script is processed by the host application and will therefore miss debug messages from the host application in these circumstances.

To debug ExtendScript triggered by callbacks:
1. Start a debug session configured with an attach request type
2. The session will connect to the specified host engine
3. Inform it that the debugger is active
4. Wait for debug messages from the connected engine

Any breakpoints or uncaught exceptions encountered by the host engine while your attach debug session is active will cause VS Code to show the debug state and enable interacting with the host application. This debug session will remain active until explicitly disconnected or stopped.

::: tip Additional Notes
- Scripts may also be evaluated while an attach debug session is active
- CEP environments in particular may benefit from a compound launch configuration to debug both scripting environments simultaneously
- If your scripts are loaded into an application through a symbolic link (most commonly seen in CEP workflows), the `aliasPath` configuration property may be required to enable debugging features
:::

## Debugger Configuration

The ExtendScript Debugger is capable of debugging with and without a standard launch configuration.

### Zero Configuration
If you have not yet defined a `launch.json` file in your project, the Run and Debug view will show the "Run and Debug" button. If you click this button while a file recognized as "JavaScript" (.js) or "JavaScript React" (.jsx) is the active file in VS Code, you will be given the option to select "ExtendScript" in the list of debugger options in the dropdown.

When you select "ExtendScript", you will be asked to select a Debugging Mode. Once you have made your choice, simply select your target host application and, if applicable, the target engine and the debug session will start.

### Launch Configuration
Setting up a launch configuration allows you to simplify and customize the process of starting a debug session. To start, please follow the standard steps to initialize a launch configuration.

When you add a new ExtendScript configuration to your `launch.json`, it will contain only the minimum settings required to start debugging. Shown below is the default attach request configuration:

```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "type": "extendscript-debug",
            "request": "attach",
            "name": "Attach to ExtendScript Engine"
        }
    ]
}
```

With the configuration above, you can select the "Attach To ExtendScript Engine" option in VS Code's Debug and Run view and initiate a debug session. Because no host application is specified, the extension will look up installed applications and show a list from which you may select a target. If your selected application is running and supports multiple engines, the extension will then ask which engine should be debugged. Once your selections are made, the debug session will begin.

See the [Advanced Configuration](#advanced-configuration) section for an explanation of available configuration options.

### Attach and Launch Mode Support
VS Code supports two debug session request types: attach and launch. Both are supported by the ExtendScript Debugger extension and have distinct meaning:

- **attach Requests**: Launch configurations with request type `attach` will connect to an ExtendScript engine running within a host application and provide it with a listing of active breakpoint information. Once this connection is established, breakpoints and debug logging will be active for any script processed by the engine, whether triggered from within the host application itself or by an Evaluation process from VS Code. The connection will remain open until it is explicitly closed using the "Disconnect" or "Stop" debug actions.

- **launch Requests**: Launch configurations with request type `launch` will connect to an ExtendScript engine running within a host application, provide it with a listing of active breakpoint information, and then trigger the evaluation of a specified script. The resulting debug session will remain active as long as the script evaluation process does not end or until the session is explicitly closed using the "Stop" or "Disconnect" debug actions.

The main difference between attach and launch requests is that the life of a launch mode debug session is tied directly to the length of time it takes the host engine to process the script. For short scripts, this may result in VS Code appearing to "flash" into and out of the "debugger active" state. This does not happen for attach configurations because the debug session connection is not tied to any specific script evaluation. For this reason, it is highly recommended that attach request debug sessions be used when debugging scripts that contain asynchronous callbacks (e.g. ScriptUI or CEP).

### Recommended Configuration Names
VS Code currently uses a "play" button for both attach and launch mode debug configurations. This can lead to confusion as many users expect that pressing a "play" button will result in their script being run. When an attach mode configuration is selected, this will not happen. As such, we recommend that users name their attach mode configurations starting with "Attach to" and launch mode configurations starting with "Launch in" (or the like) to avoid this ambiguity.

### Compound Launch Configurations
The ExtendScript Debugger extension supports Compound Launch Configurations. Scenarios where this may be of interest include:

#### Debugging CEP contexts
CEP extensions support two main scripting contexts: a JavaScript context and an ExtendScript context. These contexts have the ability to interact via certain message passing APIs. A compound configuration would allow you to start debug sessions in both the JavaScript and ExtendScript contexts with a single click.

Example configuration:
```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "type": "chrome",
            "request": "attach",
            "name": "[Attach] CEP JavaScript",
            "port": 7777,   // <-- Whatever debug port you have configured.
            "webRoot": "${workspaceRoot}",
            "pathMapping": {
                "/": "${workspaceRoot}"
            }
        },
        {
            "type": "extendscript-debug",
            "request": "attach",
            "name": "[Attach] CEP ExtendScript",
            "hostAppSpecifier": "premierepro-22.0"
        }
    ],
    "compounds": [
        {
            "name": "[Compound] Debug CEP",
            "configurations": [
                "[Attach] CEP JavaScript",
                "[Attach] CEP ExtendScript"
            ]
        }
    ]
}
```

#### Debugging Engine to Engine Messaging
When passing messages back and forth between multiple ExtendScript engines in the same or different host applications, it may be helpful to debug both engines simultaneously. Compound Launch Configurations make this easy:

```json
{
    "version": "0.2.0",
    "configurations": [
        {
            "type": "extendscript-debug",
            "request": "attach",
            "name": "Attach to InDesign Coordinator",
            "hostAppSpecifier": "indesign-17.064",
            "engineName": "MyCoordinator"
        },
        {
            "type": "extendscript-debug",
            "request": "attach",
            "name": "Attach to Premiere Pro",
            "hostAppSpecifier": "premierepro-22.0"
        }
    ],
    "compounds": [
        {
            "name": "[Compound] Debug ID-to-PPro",
            "configurations": [
                "Attach to InDesign Coordinator",
                "Attach to Premiere Pro"
            ]
        }
    ]
}
```

::: tip
The name fields in launch configurations are entirely customizable and are not used by the ExtendScript Debugger. The use of "[Attach]", "[Launch]", "[Compound]", "Attach to", and "Launch in" within the name fields is simply intended to assist with configuration disambiguation.
:::

## Advanced Configuration

The following configuration options are accepted by both attach and launch debug configurations:

| Property | Type | Description | Default Value |
|----------|------|-------------|---------------|
| hostAppSpecifier | string | The application specifier of the host application to debug. | "" |
| engineName | string | The name of the engine to target. | "" |
| hiddenTypes | string[] | An array of data types and class names that should be hidden in the Variables view. Valid names are: "undefined", "null", "boolean", "number", "string", "object", "this", "prototype", "builtin", "Function", any valid ExtendScript class name. The string "this" hides the this object. The string "prototype" hides all elements from the prototype chain, and the string "builtin" hides all elements that are part of the core ExtendScript language. | [] |
| aliasPath | string | The absolute path to a file system alias (symbolic link) for the root directory loaded by a host application. If scripts are loaded into the host application through a symbolic link, then debugger features are unlikely to work by default. This is due to a mismatch between the script paths used by VS Code (actual paths) and those used by the host application (symbolic link paths). The aliasPath property enables the ExtendScript Debugger extension to rewrite paths using the symbolic link path, thereby matching the host application's view. For this feature to work, the root directory loaded into VS Code must be the directory targeted by the symbolic link. This workflow is most commonly used in CEP projects. | "" |
| registeredSpecifier | string | A secondary application specifier that matches what the host application registered as its specifier during installation as opposed to the one it actually uses for BridgeTalk communication. NOTE: This is used only in very specific circumstances to assist in host application connections. At present, only InDesign Server connections make use of this. | "" |

The following configuration options are accepted by launch debug configurations only:

| Property | Type | Description | Default Value |
|----------|------|-------------|---------------|
| script | string | The absolute path to the script to debug in the host application. If not specified, the contents of the active editor will be used. | "" |
| bringToFront | boolean | Whether to bring the host application to front when starting the debug session or not. | false |
| debugLevel | number | The debugging level: 0 - No debugging. 1 - Break on breakpoints, errors, or exceptions. 2 - Stop at the first executable line. | 1 |

::: note
If the script property is specified, then only the saved state of the script will be evaluated in the host application. In other words, unsaved modifications are ignored when a script is specified using this property.
::: 

## Debugging

Starting an ExtendScript attach mode debug session in VS Code does not run any scripts in the connected host application - it merely informs the host application that a debugger is ready to debug. There are many ways to trigger a script to run while an attach mode debug session is active:

1. Run a script in the host application directly (specifics are determined by the host application)
2. Load a CEP extension in a host application that supports debugging CEP contexts
3. Interact with a UI element that triggers some ExtendScript to run in a host application (e.g. in a CEP extension or ScriptUI within an app that supports debugging such contexts)
4. Run the `ExtendScript: Evaluate Script in Host...` command
5. Run the `ExtendScript: Evaluate Script in Attached Host...` command

Any breakpoint, error, or exception encountered while an ExtendScript debug session is active will cause VS Code to show the debug state and enable interacting with the host application.

::: note
The above does not apply to launch mode debug sessions because launch mode debug sessions will run a script automatically when started.
:::

## Remote Debugging

The ExtendScript Debugger extension does not currently support Remote Debugging.

## VS Code Commands

The ExtendScript Debugger extension adds the following Commands to VS Code:

- `Evaluate Script in Host...`
- `Evaluate Script in Attached Host...`
- `Halt Script in Host...`
- `Clear Error Highlights...`
- `Export As Binary...`

These commands may be triggered from the Command Palette or by custom key bindings. The key bindings approach provides access to several configuration options for the `Evaluate Script in Host...` and `Evaluate Script in Attached Host...` commands.

### Evaluate Script in Host

The ExtendScript Debugger extension provides an `Evaluate Script in Host...` command which enables VS Code to instruct a host application to evaluate a given script within a specific engine. If an attach mode debug session is active for the specified engine when the command is triggered, then any configured breakpoints may cause the script to pause. If a launch mode debug session is active for the specified engine when the command is triggered, then the command will fail as the extension allows only a single script evaluation to be active in an engine at any given time. If no attach mode debug session is active when the command is triggered, then breakpoints are ignored.

When triggered, the command will show a list of installed host applications from which to select a target. If the host application has multiple ExtendScript engines running, a second list showing these options will appear. After the application and engine have been selected, the contents of the currently active editor will be sent to the host application for evaluation.

::: tip Notes
- This command will fail if the host application is not yet running. Please ensure that the desired host application is running before triggering this command.
- Upon successful evaluation, the final result will be shown in an information message box.
- If an error occurs during evaluation, any details regarding the error will appear in an error message box.
- The file does not need to be saved in order to be sent to the host application for evaluation.
:::

### Custom Key Binding Arguments

The most flexible way to trigger the `Evaluate Script in Host...` command is to create a custom key binding. When you configure a custom key binding, you not only gain the ability to more quickly trigger the command, but you have the ability to specify the available command arguments.

The command string is `extension.extendscript-debug.evalInHost` and the available command arguments include:

| Argument | Type | Description | Default Value |
|----------|------|-------------|---------------|
| hostAppSpecifier | string | Specifier for the host application within which to evaluate the file. If not specified, a prompt will appear. | "" |
| engineName | string | Name of the engine to target. Will use the default engine if not specified. If two or more engines are available, a prompt will appear. | "" |
| script | string | Absolute path to the script file to evaluate. If not specified, the contents of the active editor will be used. | "" |
| bringToFront | boolean | Whether to bring the host application to front when evaluating or not. | false |
| debugLevel | number | The debugging level: 0 - No debugging. 1 - Break on breakpoints, errors, or exceptions. 2 - Stop at the first executable line. This only applies when evaluating while a debug session with the host application is active. | 1 |
| registeredSpecifier | string | A secondary specifier that matches what the host application registered as its specifier during installation as opposed to the one it actually uses for BridgeTalk communication. NOTE: This is used only in very specific circumstances to assist in host application connections. At present, only InDesign Server connections make use of this. | "" |

Example of a configured key binding:

```json
{
    "key": "cmd+alt+j",
    "command": "extension.extendscript-debug.evalInHost",
    "args": {
        "hostAppSpecifier": "indesign-16.064",
        "engineName": "MyCoordinator",
        "debugLevel": 2,
    }
}
```

When triggered, the command above would cause the contents of the currently active editor to be sent to the "MyCoordinator" engine in InDesign and, if a debug session was active, pause on the first executable line of the script.

::: note
If the script argument is specified, then only the saved state of the file will be evaluated in the host application. In other words, unsaved modifications are ignored when a path is specified using this feature.
::: 

### Evaluate Script in Attached Host

The `Evaluate Script in Attached Host...` command is very similar to the `Evaluate Script in Host...` command. The difference is that instead of requiring that you specify a specific host application and engine, the command will evaluate the script in an engine for which an attach mode debug session is active. How the command operates depends upon how many attach mode debug sessions are active when the command is triggered:

- **Zero Active Sessions**: The command will attempt to start an attach mode debug session and will ask which host application and, if applicable, engine to target. Once the debug session starts, the script evaluation will begin.
- **One Active Session**: The command will immediately evaluate the script in the engine targeted by the active debug session.
- **Multiple Active Sessions**: The command will present a list of the active attach mode debug sessions and allow you to select which to target for script evaluation.

The command string is `extension.extendscript-debug.evalInAttachedHost`. The command supports the same set of configurable command arguments as `Evaluate Script in Host...` except that the `hostAppSpecifier`, `engineName`, and `registeredSpecifier` fields are ignored (they are supplied directly by the debug session).

### Halt Script in Host

The `Halt Script in Host...` command enables you to halt (terminate/stop/end) any active evaluation process that was initially started with the `Evaluate Script in Host...` command. If there is only a single active evaluation process when this command is triggered, then that evaluation will be halted. If there are multiple active evaluation processes when the command is triggered, then the command will show a list of active evaluation processes that may be halted. Selecting one from this list will halt the selected evaluation.

The command string is `extension.extendscript-debug.haltInHost` and there are no configurable command arguments.

### Clear Error Highlights

Triggering the `Clear Error Highlights...` command will clean up any active ExtendScript error highlights.

The command string is `extension.extendscript-debug.clearErrorHighlights` and there are no configurable command arguments.

### Export to JSXBin

You can export your .js and .jsx scripts to JSXBin by right-clicking the editor window of a .js or .jsx file and selecting `Export As Binary...`. A file name suggestion will be provided. If you proceed with the export, the results will be saved in the same directory as your currently opened file. You can enter a complete path for the output if desired.

The command string is `extension.extendscript-debug.exportToJSXBin` and there are no configurable command arguments.

## VS Code Status Bar Buttons

The ExtendScript Debugger extension adds two new buttons to the Status Bar that appear/disappear based on context.

### Eval in Adobe... Button

This button appears when a document either:
- recognized by VS Code as javascript, javascriptreact, or extendscript, or
- has a file extension of .jsxbin

is focused. Clicking this button triggers the `Evaluate Script in Host...` command. Once a target host application/engine combination is chosen, the contents of the focused document will be evaluated within it.

When an attach mode debug session is active, the `Eval in Adobe...` button changes to read `Eval in Adobe [name of application] (engine)...`. Clicking this button will evaluate the focused script in the application being debugged.

### Halt in Adobe... Button

This button appears when a script evaluation is triggered from within VS Code. If only a single evaluation process is actively running, then the button will read `Halt in Adobe [name of application] (engine)...` and clicking it will immediately halt that evaluation process.

If more than one evaluation process is active, then the button will read `Halt in Adobe...` and clicking it will open a list showing all active evaluation processes. Selecting a process from this list will cause that process to halt.

### Batch Export to JSXBin

By using the `Export As Binary...` command as described above, you can export one file at a time. However you can also batch export your .js and .jsx files to JSXBin using a script provided with the extension.

1. Ensure that you have NodeJS installed
2. Locate the script at the following location:
   - Mac: `$HOME/.vscode/extensions/<adobe.extendscript-debug extension directory>/public-scripts/exportToJSXBin.js`
   - Windows: `%USERPROFILE%\.vscode\extensions\<adobe.extendscript-debug extension directory>\public-scripts\exportToJSXBin.js`
3. Run the following command from terminal:
   ```bash
   node <Path to exportToJSXBin.js> [options] [filename/directory]
   ``` 

## InDesign Server (or When Host Applications Go Rogue)

During installation InDesign Server registers itself incorrectly for communication with other applications (including debuggers). The result is that the debugger is able to access certain information about the application but it fails to make any connections to running instances. To correct for this, the ExtendScript Debugger contains several configuration options and settings to enable the extension's features to correctly interface with InDesign Server instances.

The ExtendScript Debugger extension supports per-configuration properties and "global" settings to work around the issue.

### The registeredSpecifier Property/Argument

A `registeredSpecifier` property may be specified in debug configurations and custom key binding arguments for the `Evaluate Script in Host...` command. This property refers to the specifier that the host application registers for itself during installation. When this property is present, the `hostAppSpecifier` is used for communicating with the host application while the `registeredSpecifier` is used to resolve application metadata (like it's "Display Name"). For most applications, the registered specifier is the same one used for communication so specifying the value is unnecessary.

An example pairing might look like:

```json
"hostAppSpecifier": "indesignserver_myconfig-17.064",
"registeredSpecifier": "indesignserver-17.0",
```

Properly setting the `registeredSpecifier` debug configuration property or key binding argument will allow only that specific configuration to work. See below for a more "global" solution.

### The "Application Specifier Overrides" Extension Setting

The "Application Specifier Overrides" extension setting enables full control over how the extension interprets any `hostAppSpecifier` value it encounters. With proper configuration, this setting will enable the default `Evaluate Script in Host...` command and related features to work with "default" InDesign Server instances (those started without a custom port or configuration). It also enables you to skip adding the `registeredSpecifier` property/argument in configurations.

The setting expects an array of objects with the following properties:

| Property | Type | Description | Default Value |
|----------|------|-------------|---------------|
| appSpecRegExp | string | A JavaScript regular expression value that is used to test against "Host Application Specifier" values for applicability. This applies to any custom hostAppSpecifier used in either debug configurations or custom key binding arguments. Proper declaration of the regular expression will allow custom application instances to resolve as expected. | "" |
| registeredSpecifier | string | The specifier that the host application registers for itself during installation. | "" |
| commsSpecifier | string | The specifier by which the "default" application instance will communicate. | "" |

Example:

```json
"extendscript.advanced.applicationSpecifierOverrides": [
    {
        "appSpecRegExp": "indesignserver[_a-z0-9]*-17",
        "registeredSpecifier": "indesignserver-17.0",
        "commsSpecifier": "indesignserver-17.064"
    }
]
```

The `appSpecRegExp` in the example above will successfully match against a `hostAppSpecifier` with value "indesignserver_myconfig-17.064" and will instruct any configuration in which it was found to use the `registeredSpecifier` value of "indesignserver-17.0". Additionally, if the `Evaluate Script in Host...` command is run without custom arguments, then the extension will match the `registeredSpecifier` value of "indesignserver-17.0" against the specifier it automatically uses, find that they are the same, and then use the `commsSpecifier` value of "indesignserver-17.064" for communication. If a "default" InDesign Server 2022 instance is running, then the script evaluation process will proceed as expected.

::: tip
This feature can be used to point the `Eval in Adobe...` button and base `Evaluate Script in Host...` command to a specific instance. This is only recommended if you always use the same application instance (e.g. InDesign Server port and configuration settings). In such cases, simply specify the full instance specifier for the `commsSpecifier` property.
:::

## Common Extensibility Platform (CEP)

CEP extensions run scripts in one of two separate contexts:

1. An ExtendScript engine. This engine is provided by the host application and supports the host application's DOM.
2. A JavaScript engine. This engine is provided by CEF (which is part of the CEP runtime) and supports the HTML DOM used for the extension's UI.

Of these two contexts (and where supported by the host application), the ExtendScript Debugger extension can be used to debug scripts run in the ExtendScript engine context (#1 above).

::: note
The JavaScript engine context (#2 above) can be debugged using VS Code's built-in JavaScript debugger. Once configured properly, both contexts can be debugged simultaneously using a Compound Launch Configuration.
:::

Unfortunately, no two host applications implement the ExtendScript engine context in the same way. Some are easy to debug, some require special configuration to work with, and some (e.g. Photoshop) simply do not support debugging CEP ExtendScript engines.

This section describes how to use the ExtendScript Debugger extension to debug CEP ExtendScript contexts where such functionality is supported by the host application.

### General CEP Debugging Notes

#### Loading Scripts
There are three different ways to ask a Host Application to evaluate ExtendScript scripts in CEP:

1. The CSXS Manifest's `<ScriptPath>` Element
2. The `//@include` Preprocessor Directive
3. The `$.evalFile()` Function

Loading a script with option #1 treats the file as an unnamed script. As suggested here:

> An unnamed script is assigned a unique name generated from a number.

::: note
Some host applications don't even assign a number for scripts loaded in this manner. The script's name evaluates to the empty string.
:::

Such scripts are effectively anonymous scripts and complicate the debugging experience. Unfortunately, when a breakpoint or exception is encountered in such a script, the ExtendScript Debugger extension has no way to connect that message to a file in the project. In such circumstances, a new unsaved file will be shown with the break state.

Scripts loaded with either option #2 or #3 as listed above do not suffer from this issue. In these circumstances, host applications provide the ExtendScript Debugger extension with the information required to connect breakpoints or exceptions to the correct source file in the project and the debugger will operate as expected.

Please see the following sections for examples on how to load CEP ExtendScript scripts in ways that support an improved debugging experience.

#### Treating the `<ScriptPath>` Script as a Loader
In this scenario, you might create a file called "loader.jsx" and specify the file in your CSXS Manifest's `<ScriptPath>`. This file itself would have the same problem outlined above. However, the purpose of this script is simply to load the scripts that actually contain your business logic.

Note that the `<ScriptPath>` script is not automatically invoked by the host application by default. One event that causes the evaluation of this script is when `CSInterface.evalScript()` is called for the first time. There may be other events that also trigger the specified script to evaluate.

::: warning
Photoshop (amongst others?) does not support loading scripts in this manner. Attempts to process other files from the one specified within the `<ScriptPath>` element will fail. Photoshop also does not support debugging CEP ExtendScript.
:::

There are two ways to use this approach:

1. Preprocessor directives. Scripts loaded using the `//@include` preprocessor directive maintain all of the information necessary to communicate with the debugger. Example:
   ```javascript
   //@include "helloworld.jsx"
   //@include "someotherscript.jsx"
   ```

2. Using the `$.evalFile()` function. Scripts loaded using the `$.evalFile()` function maintain all of the information necessary to communicate with the debugger. Example:
   ```javascript
   $.evalFile($.includePath + "/helloworld.jsx");
   $.evalFile($.includePath + "/someotherscript.jsx");
   ```

#### Loading ExtendScript from the JavaScript Context
In this scenario, you trigger the `$.evalFile()` function from the browser context via CEP's `CSInterface.evalScript()` function. For this option, you might add the following logic to a `<script>` block in your extension's HTML somewhere:

```javascript
const csInterface = new CSInterface();

// Use the extension path reported by the CSInterface.
const path = csInterface.getSystemPath(SystemPath.EXTENSION);
csInterface.evalScript(`$.evalFile("${path}/host/helloworld.jsx")`);
csInterface.evalScript(`$.evalFile("${path}/host/someotherscript.jsx")`);
```

Or, alternatively (for non-Photoshop [and possibly others?] applications):

```javascript
const csInterface = new CSInterface();

// Use the include path set for the CEP's ExtendScript context.
csInterface.evalScript(`$.evalFile($.includePath + "/host/helloworld.jsx")`);
csInterface.evalScript(`$.evalFile($.includePath + "/host/someotherscript.jsx")`);
```

::: warning
Photoshop (amongst others?) does not support the second alternative. The value of `$.includePath` resolves to the empty string.
:::

#### Reloading CEP Extensions and ExtendScript
Restarting a CEP extension will also cause it to reevaluate the ExtendScript. Another way to reevaluate the ExtendScript is to use the ExtendScript Debugger extension's various methods of running a script. Note that this may cause oddities as the ExtendScript engine itself isn't reset - you're just overwriting the existing variable and function definitions. This can cause surprising issues if you're not careful.

#### Manually Enabling Debugging
Some host applications run their CEP ExtendScript engine in a manner that constantly resets the "debug level" of the engine to "No Debugging". When this happens, an attached debug session may appear to ignore breakpoints and exceptions. Fortunately, it is possible to work around this issue by manually setting the debug level of the ExtendScript engine with the `$.level` property. Setting this property to either 1 or 2 will reenable the debugging features of the engine.

There are two approaches to manually adjusting the debug level so that you can debug your CEP callback scripts. Depending upon your workflow, one of these may be more flexible for you than the other:

1. Set `$.level` inside the ExtendScript callback function. With this approach, you simply set the value at the top of the function you wish to debug. See:
   ```javascript
   function DoSomething()
   {
       $.level = 1;

       // Breakpoints will work here.
   }
   ```

2. Set the `$.level` when calling the ExtendScript callback function. ExtendScript functions triggered from CEP browser contexts use the `CSInterface.evalScript()` API. We can set the `$.level` value from within this interface as follows:
   ```javascript
   csInterface.evalScript(`$.level = 1; DoSomething();`);
   ```

## Known Issues

1. **Apple Silicon Support**: The Extension fails to work on Apple devices using Apple Silicon (e.g. M1 processors). Internally, the extension interfaces with a special library that handles communication with host applications. This library is currently Intel-only. To successfully run the extension on Apple devices running Apple Silicon, VS Code itself must be run with Rosetta. Please see Apple's documentation for information on how to configure a universal build of VS Code to run using Rosetta. Alternatively, you can download the Intel-specific build of VS Code and run it directly.

2. **Windows on ARM**: The Extension fails to work on Windows on ARM devices. Use of the ExtendScript Debugger on Windows on ARM devices is not supported at this time.

3. **Bring Target to Front**: Bring Target to Front does not work for certain host applications. Certain host applications on certain operating systems may ignore the extension's request to come forward. A possible workaround is to add `BridgeTalk.bringToFront("host-app-specifier")` to the top of the script you wish to evaluate.

4. **InDesign Server Connection**: The debugger fails to connect to InDesign Server. The ExtendScript Debugger extension fails to recognize that InDesign Server is running. This is due to a BridgeTalk registration issue in InDesign Server itself. See the [InDesign Server](#indesign-server-or-when-host-applications-go-rogue) section for information on how to work around this issue.

5. **this Object in Variables View**: The `this` object does not appear in the Variables view. All ExtendScript engines contain a bug that causes the implicit `this` variable to display incorrect contents when viewed from all but the top stack frame in a given call stack (only the implicit `this` for the top stack frame is ever resolved). For consistency, the implicit `this` variable is not listed. If you need to view the contents of the implicit `this` in any context, you may do so by adding `var _this = this;` to your script. The `_this` variable will appear in the Variables view and allow you to inspect the contents of the implicit `this` as expected.

   This issue also affects the Debug Console. Entering `this` into the Debug Console will only ever refer to the implicit `this` resolved in the context of the top stack frame.

6. **Binary Values**: Unencoded binary values may break the underlying debugger protocol. All host applications have a known bug where attempts to send binary-encoded data to the debugger will fail. This typically results in missing Debug Console output or an empty Variables view. Scenarios where you may encounter this issue include when attempting to view the results of a binary file `read()` operation or when writing binary values directly in ExtendScript. The following script, for instance, will trigger this issue:

   ```javascript
   var x = "\0";             // String representation of "NULL"
   $.writeln("x is: " + x);  // Write the value of `x` to the Debug Console: does nothing
   $.bp();                   // Ask the debugger to break: the Variables view will show an error
   ```

   To work around this issue and "see" the contents of the problematic variable, you may encode it using `encodeURI()`, `toSource()`, or by using a `btoa()` polyfill. For example:

   ```javascript
   var x = encodeURI("\0");  // Encoded string representation of "NULL"
   $.writeln("x is: " + x);  // Write the value of `x` to the Debug Console: prints "x is: %00"
   $.bp();                   // Ask the debugger to break: the Variables view works as expected
   ```

   ::: note
   This issue is present in all ExtendScript debuggers, including the original ESTK.
   :::

## FAQ

### Can debug sessions or the Evaluate Script in Host... command be configured to launch the host application?
The ExtendScript Debugger extension does not currently support launching host applications.

### How do I halt evaluation of a script that wasn't started from VS Code?
There are currently two options:

1. Connect an attach mode debug session to the host engine within which the script is evaluating. Once connected use the "Stop" debug action to simultaneously end the debug session and halt the script. The "Disconnect" button can be converted into a "Stop" button by holding the Alt/option key.

2. Attempt to evaluate a script (e.g. with a command or by starting a launch mode debug session) in the host engine within which the script is evaluating. The active evaluation will be detected and you will be offered the option to halt the active evaluation process and retry evaluating the script you specified. 

## General Notes

1. If the ExtendScript Toolkit (ESTK) connects to a host application, then the ExtendScript Debugger extension will no longer be able to function correctly as a debugger. Restarting the host application is enough to fix this issue.

2. A single host application can only be debugged by a single VS Code window. If two or more VS Code windows attempt to maintain debug sessions with a single host application at the same time, only the last one to connect will work.

3. A single VS Code window can manage multiple debug sessions with multiple host application/engine combinations at the same time.

4. Once a VS Code Window connects to a host application, it will begin acting as the de facto debugger for all future debugging purposes (until another VS Code window connects). For host applications that support multiple engines, this may mean that an engine with no active debug session triggers a breakpoint and notifies the ExtendScript Debugger extension about the break. In these cases, the extension will attempt to notify you and offer you the ability to attach a debug session.

5. Changes to the "Caught Exceptions" breakpoint while a script is evaluating (e.g. stopped at a breakpoint) will only apply to newly created scopes (stack frames).

6. When an `Evaluate Script in Host...` command is run without an active debug session and fails with an error status, the extension will highlight the line of source code reported with the error. You may clear these highlights in one of the following manners:
   - Focus another source file such that the source file with a highlight becomes a background tab in VS Code
   - Close and reopen the source file
   - Start another script evaluation
   - Start a debug session
   - If the "Show Result Messages" setting is enabled, dismiss the relevant error message (if hidden, click the notification bell in the right side of the status bar)
   - Run the `Clear Error Highlights...` command

7. The ExtendScript Debugger extension ignores the `#target` and `#targetengine` preprocessor directives. The extension will always use either the configured `hostAppSpecifier` and `engineName` settings or, if not otherwise specified, those dynamically chosen in the relevant UI.

8. When disconnecting from a Debug Session:
   - The host engine is instructed to unregister all VS Code breakpoints and continue evaluation
   - Any script-based breakpoints (e.g. `debugger` or `$.bp()`) subsequently encountered by the host engine will cause the host engine to pause and await communication from a debugger. When this occurs, the extension will attempt to notify you and offer you the ability to attach a debug session to investigate the breakpoint details. If you dismiss or otherwise miss this notification, you may either halt the evaluation process or manually connect an attach mode debug session.

## Resources

- [Official VS Code Debugging Documentation](https://code.visualstudio.com/docs/editor/debugging)
- [Extensions / Add-ons Development Forum](https://community.adobe.com/t5/extensibility/ct-p/ct-extensibility) 