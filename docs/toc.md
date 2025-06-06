# Table Of Contents

## [Developing Adobe CEP Extensions](guide/developing-cep-extensions.md)
- [Development Setup](guide/developing-cep-extensions.md#1-development-setup)
  - [Prerequisites](guide/developing-cep-extensions.md#prerequisites)
  - [Folder Structure](guide/developing-cep-extensions.md#folder-structure)
  - [manifest.xml Configuration](guide/developing-cep-extensions.md#manifestxml-configuration)
  - [Installation & Configuration](guide/developing-cep-extensions.md#installation--configuration)
- [Debugging CEP Extensions](guide/developing-cep-extensions.md#2-debugging-cep-extensions)
- [Best Practices](guide/developing-cep-extensions.md#3-best-practices-for-efficient-cep-development)
- [BOLT-CEP Template](guide/developing-cep-extensions.md#4-hyperbrew-bolt-cep-template)
- [Alternative Frameworks](guide/developing-cep-extensions.md#5-alternative-cep-development-frameworks--tools)
- [Advanced Topics](guide/developing-cep-extensions.md#6-advanced-topics)


## [Getting Started](guide/getting-started.md)
- [Technology Used](guide/getting-started.md#technology-used)
- [Prerequisites](guide/getting-started.md#prerequisites)
- [Development Steps](guide/getting-started.md#development-steps)
  - [1. Decide the folder structure](guide/getting-started.md#1-decide-the-folder-structure)
  - [2. Configure Your Extension in `manifest.xml`](guide/getting-started.md#2-configure-your-extension-in-manifestxml)
  - [3. Download `CSInterface.js`](guide/getting-started.md#3-download-csinterfacejs)
  - [4. Write Your Front-end Code](guide/getting-started.md#4-write-your-front-end-code)
    - [Create HTML Markup](guide/getting-started.md#create-html-markup)
    - [Write Your JavaScript Code](guide/getting-started.md#write-your-javascript-code)
  - [5. Write Your ExtendScript Code](guide/getting-started.md#5-write-your-extendscript-code)
  - [6. Launch your extension in the host app](guide/getting-started.md#6-launch-your-extension-in-the-host-app)
- [Next Steps](guide/getting-started.md#next-steps)
- [Other Resources](guide/getting-started.md#other-resources)

## [Client Side Debugging](guide/debugging.md)
- [Prerequisites](guide/debugging.md#prerequisites)
- [Set the Debug Mode](guide/debugging.md#set-the-debug-mode)
- [Create a `.debug` File](guide/debugging.md#create-a-debug-file)
- [Write Contents for the `.debug` File](guide/debugging.md#write-contents-for-the-debug-file)
- [Debugging in Chrome](guide/debugging.md#debugging-in-chrome)
- [Troubleshooting common issues](guide/debugging.md#troubleshooting-common-issues)
    - [Getting a "not properly signed" alert](guide/debugging.md#getting-a-not-properly-signed-alert)
    - [Getting a blank debug console](guide/debugging.md#getting-a-blank-debug-console)
- [Next Steps](guide/debugging.md#next-steps)
- [Other Resources](guide/debugging.md#other-resources)

## [Exporting Files from the Host App](guide/exporting-files.md)
- [Technology Used](guide/exporting-files.md#technology-used)
- [Prerequisites](guide/exporting-files.md#prerequisites)
- [Configuration](guide/exporting-files.md#configuration)
- [Client-side: HTML Markup](guide/exporting-files.md#client-side-html-markup)
- [Client-side: Creative Cloud host app interaction](guide/exporting-files.md#client-side-creative-cloud-host-app-interaction)
- [Host app: Automation with ExtendScript](guide/exporting-files.md#host-app-automation-with-extendscript)
- [Other Resources](guide/exporting-files.md#other-resources)

## [Network Requests and Responses with Fetch](guide/network-requests.md)
- [Technology Used](guide/network-requests.md#technology-used)
- [Prerequisites](guide/network-requests.md#prerequisites)
- [Configuration](guide/network-requests.md#configuration)
- [Client-side: HTML Markup](guide/network-requests.md#client-side-html-markup)
- [Client-side: Service API interaction](guide/network-requests.md#client-side-service-api-interaction)
- [Client-side: Creative Cloud host app interaction](guide/network-requests.md#client-side-creative-cloud-host-app-interaction)
- [Host app: Automation with ExtendScript](guide/network-requests.md#host-app-automation-with-extendscript)
- [Other Resources](guide/network-requests.md#other-resources)

## [Bolt CEP](tools/bolt-cep.md)
- [Features](tools/bolt-cep.md#features)
- [Dev Requirements](tools/bolt-cep.md#dev-requirements)
- [Compatibility](tools/bolt-cep.md#compatibility)
- [Backers](tools/bolt-cep.md#backers)
- [Tools Built with Bolt CEP](tools/bolt-cep.md#tools-built-with-bolt-cep)
- [Support](tools/bolt-cep.md#support)
  - [Free Support](tools/bolt-cep.md#free-support-)
  - [Paid Priority Support](tools/bolt-cep.md#paid-priority-support-)
- [Using Bolt CEP in Projects](tools/bolt-cep.md#can-i-use-bolt-cep-in-my-free-or-commercial-project)
- [Quick Start](tools/bolt-cep.md#quick-start)
- [Config](tools/bolt-cep.md#config)
- [CEP Panel Structure](tools/bolt-cep.md#cep-panel-structure)
- [ExtendScript](tools/bolt-cep.md#extendscript)

## [ExtendScript Debugger](tools/extendscript-debugger.md)
- [Features](tools/extendscript-debugger.md#features)
  - [Supported Features](tools/extendscript-debugger.md#supported-features)
  - [Unsupported Features](tools/extendscript-debugger.md#unsupported-features)
- [Getting Started](tools/extendscript-debugger.md#getting-started)
  - [Installation](tools/extendscript-debugger.md#installation)
  - [Migration from V1 Versions](tools/extendscript-debugger.md#migration-from-v1-versions)
- [Using the Debugger](tools/extendscript-debugger.md#using-the-debugger)
  - [Running a Script](tools/extendscript-debugger.md#running-a-script)
  - [Debugging a Script](tools/extendscript-debugger.md#debugging-a-script)
  - [Debugging Event Callbacks](tools/extendscript-debugger.md#debugging-event-callbacks)
- [Debugger Configuration](tools/extendscript-debugger.md#debugger-configuration)
  - [Zero Configuration](tools/extendscript-debugger.md#zero-configuration)
  - [Launch Configuration](tools/extendscript-debugger.md#launch-configuration)
  - [Attach and Launch Mode Support](tools/extendscript-debugger.md#attach-and-launch-mode-support)
  - [Recommended Configuration Names](tools/extendscript-debugger.md#recommended-configuration-names)
  - [Compound Launch Configurations](tools/extendscript-debugger.md#compound-launch-configurations)
- [Advanced Configuration](tools/extendscript-debugger.md#advanced-configuration)
- [Debugging](tools/extendscript-debugger.md#debugging)
- [Remote Debugging](tools/extendscript-debugger.md#remote-debugging)
- [VS Code Commands](tools/extendscript-debugger.md#vs-code-commands)
  - [Evaluate Script in Host](tools/extendscript-debugger.md#evaluate-script-in-host)
  - [Custom Key Binding Arguments](tools/extendscript-debugger.md#custom-key-binding-arguments)
  - [Evaluate Script in Attached Host](tools/extendscript-debugger.md#evaluate-script-in-attached-host)
  - [Halt Script in Host](tools/extendscript-debugger.md#halt-script-in-host)
  - [Clear Error Highlights](tools/extendscript-debugger.md#clear-error-highlights)
  - [Export to JSXBin](tools/extendscript-debugger.md#export-to-jsxbin)
- [VS Code Status Bar Buttons](tools/extendscript-debugger.md#vs-code-status-bar-buttons)
  - [Eval in Adobe... Button](tools/extendscript-debugger.md#eval-in-adobe-button)
  - [Halt in Adobe... Button](tools/extendscript-debugger.md#halt-in-adobe-button)
  - [Batch Export to JSXBin](tools/extendscript-debugger.md#batch-export-to-jsxbin)
- [InDesign Server](tools/extendscript-debugger.md#indesign-server-or-when-host-applications-go-rogue)
  - [The registeredSpecifier Property/Argument](tools/extendscript-debugger.md#the-registeredspecifier-propertyargument)
  - [The "Application Specifier Overrides" Extension Setting](tools/extendscript-debugger.md#the-application-specifier-overrides-extension-setting)
- [Common Extensibility Platform (CEP)](tools/extendscript-debugger.md#common-extensibility-platform-cep)
  - [General CEP Debugging Notes](tools/extendscript-debugger.md#general-cep-debugging-notes)
- [Known Issues](tools/extendscript-debugger.md#known-issues)
- [FAQ](tools/extendscript-debugger.md#faq)
- [General Notes](tools/extendscript-debugger.md#general-notes)
- [Resources](tools/extendscript-debugger.md#resources)

## [CEP 12 HTML Extension Cookbook](reference/html-extension-cookbook.md)
- [Overview](reference/html-extension-cookbook.md#overview)
  - [CEP Extensions](reference/html-extension-cookbook.md#cep-extensions)
  - [Extension Types](reference/html-extension-cookbook.md#extension-types)
  - [Applications Integrated with CEP](reference/html-extension-cookbook.md#applications-integrated-with-cep)
  - [Chromium Embedded Framework (CEF)](reference/html-extension-cookbook.md#chromium-embedded-framework-cef)
  - [Browser Features supported by CEP](reference/html-extension-cookbook.md#browser-features-supported-by-cep)
- [Development and Debugging](reference/html-extension-cookbook.md#development-and-debugging)
  - [Migration from CEP 11 to CEP 12](reference/html-extension-cookbook.md#migration-from-cep-11-to-cep-12)
  - [Migration from CEP 10 to CEP 11](reference/html-extension-cookbook.md#migration-from-cep-10-to-cep-11)
  - [Migration from CEP 9 to CEP 10](reference/html-extension-cookbook.md#migration-from-cep-9-to-cep-10)
  - [Migration from CEP 8 to CEP 9](reference/html-extension-cookbook.md#migration-from-cep-8-to-cep-9)
  - [NodeJS](reference/html-extension-cookbook.md#nodejs)

## [Resources and References](reference/cep-resources-and-tools.md)
- [Official Documentation](reference/cep-resources-and-tools.md#official-documentation)
- [Development Frameworks](reference/cep-resources-and-tools.md#development-frameworks)
- [Community Resources](reference/cep-resources-and-tools.md#community-resources)
- [Development Tools](reference/cep-resources-and-tools.md#development-tools)
- [Code Examples](reference/cep-resources-and-tools.md#code-examples)
- [Migration Resources](reference/cep-resources-and-tools.md#migration-resources) 