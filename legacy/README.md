# Legacy prototype

`screen_hotkey_listener.bat` is the original Batch/PowerShell proof of concept that preceded the current SCapturer application.

It is preserved to document the project's origin and early technical experiments. It is not part of the supported build, packaging, or release path.

## What the prototype did

The script combined Batch, PowerShell, dynamically compiled C#, Windows Forms, and Win32 hotkey registration in one file.

Its behavior included:

- creating a Startup-folder shortcut for automatic launch;
- extracting an embedded PowerShell payload to `%TEMP%`;
- registering `Ctrl+Shift+G` and `Ctrl+Shift+Q`;
- polling the same key combinations as a fallback;
- capturing the complete Windows virtual desktop;
- copying the captured bitmap to the clipboard;
- playing a notification sound;
- running through a hidden Windows Forms listener.

The prototype copied screenshots to the clipboard only. It did not provide the current application's PNG persistence, region overlay, configurable settings, diagnostics, backend selection, bounded pipeline, lifecycle controls, or installer integration.

## Status

The file is historical and unsupported:

- do not use it as the normal SCapturer launcher;
- do not include it in portable or MSI packages;
- do not treat its Startup shortcut as the current autostart mechanism;
- do not use it as the reference for current capture or lifecycle behavior.

The active implementation is the C# solution rooted at:

```text
SCapturer.sln
```

Current source code is under:

```text
src/SCapturer.App
src/SCapturer.Core
```

Project documentation begins at the repository-root [`README.md`](../README.md).

## Why it remains in the repository

Keeping the prototype provides a compact record of the migration from a single-file automation script to a structured Windows application. It can also help explain early implementation choices, but changes to the production application must be made in the active C# projects.

No maintenance or compatibility guarantees apply to the legacy script.
