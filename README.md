# SCapturer

SCapturer is a Windows screenshot utility built for fast, reliable desktop capture and straightforward developer diagnostics.

It supports full virtual-desktop screenshots, rectangular region capture, configurable global hotkeys, background operation, lossless PNG output, and an interactive console interface. The application is written in C# and ships as a self-contained Windows executable.

> SCapturer is a developer utility rather than a general-purpose consumer screenshot application. Its interface exposes runtime state, backend selection, capture timings, and diagnostics directly.

## Features

- Full virtual-desktop capture across multiple monitors
- Rectangular region capture from a cached desktop frame
- Physical-pixel coordinates with Per-Monitor V2 DPI awareness
- Support for mixed-DPI layouts and negative monitor coordinates
- Configurable global hotkeys with validation and rollback
- Reference GDI+ and native GDI + WIC capture backends
- Atomic PNG persistence with explicit disk flush
- Optional clipboard copy, capture sound, and diagnostics
- Single-instance activation and command forwarding
- Hidden background operation and current-user Windows autostart
- Built-in baseline and backend-comparison benchmarks
- Dependency-free logic tests and a Windows reliability harness

## Default hotkeys

| Shortcut | Action |
| --- | --- |
| `Ctrl + Shift + G` | Capture the complete virtual desktop |
| `Ctrl + Shift + S` | Open rectangular region selection |
| `Ctrl + Shift + H` | Show or hide the management console |
| `Ctrl + Shift + Q` | Exit after active file operations finish |

Hotkeys can be changed from the **Hotkeys** page.

## How capture works

### Full desktop

SCapturer captures the complete physical virtual desktop, including monitors positioned to the left or above the primary display.

### Region capture

Region capture uses one desktop frame for the complete interaction:

1. capture the virtual desktop;
2. display a dimmed overlay across all monitors;
3. track the selected rectangle;
4. crop the original cached frame;
5. save the result as PNG;
6. optionally publish it to the clipboard.

The desktop is not captured again while the selection rectangle is moving, so the saved image does not contain the overlay, border, or size label.

## Capture backends

SCapturer includes two Windows capture implementations.

### Reference GDI+

The compatibility backend uses `System.Drawing`, `Graphics.CopyFromScreen`, and GDI+ PNG persistence.

### Native GDI + WIC

The native backend uses:

- `CreateDIBSection` for a top-down BGRA buffer;
- `BitBlt` for desktop acquisition and crop operations;
- Windows Imaging Component for PNG encoding.

The **Diagnostics** page can benchmark both backends and apply the native backend only when it passes the configured performance gate.

See [Backend comparison](docs/BACKEND_COMPARISON.md) and [Performance](docs/PERFORMANCE.md).

## Console interface

The management console contains these pages:

1. Dashboard
2. Capture Settings
3. Hotkeys
4. Save Locations
5. Diagnostics
6. Recent Captures
7. Background and Startup
8. About

Navigation:

| Key | Action |
| --- | --- |
| `↑` / `↓` or `J` / `K` | Move selection |
| `Enter` | Run the selected action |
| `1`–`9`, `0` | Run a visible item directly |
| `Esc` / `Backspace` | Return to the dashboard |
| `R` | Refresh recent captures |
| `F` | Open the selected capture folder |

The interface uses differential rendering, semantic colors, fixed-column runtime telemetry, and a bounded session event feed.

See [Console UI](docs/CONSOLE_UI.md).

## Background operation

SCapturer can run without a visible console window. The same process continues to own:

- global hotkeys;
- the capture worker;
- clipboard publication;
- display-topology observation;
- diagnostics;
- single-instance IPC.

Launching the executable again shows the existing console instead of starting a second independent listener.

Closing the native console window with `X` triggers a hidden background handoff because Windows terminates a process after `CTRL_CLOSE_EVENT`. Normal console hiding through the hotkey or menu remains an in-process window hide operation.

Windows autostart is optional and managed from the **Background and Startup** page.

See [Background and autostart](docs/BACKGROUND_AND_AUTOSTART.md).

## Storage locations

| Data | Default location |
| --- | --- |
| Full captures | `%USERPROFILE%\Pictures\SCapturer\Full` |
| Region captures | `%USERPROFILE%\Pictures\SCapturer\Snips` |
| Settings | `%LOCALAPPDATA%\SCapturer\config.json` |
| Diagnostics | `%LOCALAPPDATA%\SCapturer\diagnostics` |

Capture folders can be changed from the console interface.

See [Storage and clipboard](docs/STORAGE_AND_CLIPBOARD.md).

## Requirements

Development and packaging currently target:

- Windows 11 x64
- Windows 10 version 2004 or newer on a best-effort basis
- .NET 8 SDK
- WiX Toolset 3.14 for MSI generation

SCapturer is Windows-specific and depends on Win32 display, hotkey, clipboard, GDI, and WIC APIs.

## Build and run

Build the solution:

```powershell
dotnet build .\SCapturer.sln -c Release
```

Run from source:

```powershell
dotnet run --project .\src\SCapturer.App\SCapturer.App.csproj -c Release
```

Run hidden:

```powershell
dotnet run --project .\src\SCapturer.App\SCapturer.App.csproj -c Release -- --background
```

## Verification

Run the automated logic tests:

```powershell
dotnet run --project .\tests\SCapturer.Tests\SCapturer.Tests.csproj -c Release
```

Run the default Windows reliability workload after a Release build:

```powershell
dotnet run --project .\tools\SCapturer.Reliability\SCapturer.Reliability.csproj -c Release -- --captures 100 --console-cycles 30 --region-cancel-cycles 5 --process-cycles 10
```

Reports are written under `artifacts\reliability`.

See [Reliability](docs/RELIABILITY.md).

## Release packaging

Build the portable ZIP and per-user MSI from the same self-contained `win-x64` publish output:

```powershell
.\scripts\build-release.ps1 -Version 0.1.0
```

Expected output:

```text
dist\release\0.1.0\
├─ SCapturer-v0.1.0-win-x64-portable.zip
├─ SCapturer-v0.1.0-win-x64.msi
├─ RELEASE_NOTES.md
└─ SHA256SUMS.txt
```

The MSI installs for the current user under:

```text
%LOCALAPPDATA%\Programs\X-LAB\SCapturer
```

It creates a Start Menu shortcut, preserves user captures and settings during uninstall, and shuts down a running instance gracefully during maintenance.

See [Packaging](docs/PACKAGING.md) and the [Release checklist](docs/RELEASE_CHECKLIST.md).

## Command-line actions

SCapturer accepts one action per invocation.

| Argument | Action |
| --- | --- |
| `--background` | Start hidden |
| `--show` | Show the current instance |
| `--hide` | Hide the current instance |
| `--toggle-console` | Toggle console visibility |
| `--capture-full` | Queue a full-desktop capture |
| `--capture-region` | Queue region capture |
| `--cancel-region` | Cancel active region selection |
| `--exit` | Exit gracefully |

Maintenance-only arguments are used internally by the MSI package.

## Repository layout

```text
src/
  SCapturer.App/         Executable shell, lifecycle, and console UI
  SCapturer.Core/        Capture, persistence, settings, and diagnostics
tests/
  SCapturer.Tests/       Dependency-free automated logic tests
tools/
  SCapturer.Reliability/ Windows reliability and resource-soak harness
docs/                    Technical documentation
packaging/               Portable and MSI packaging sources
scripts/                 Release automation
legacy/                  Original Batch/PowerShell prototype
```

## Documentation

- [Architecture](docs/ARCHITECTURE.md)
- [Backend comparison](docs/BACKEND_COMPARISON.md)
- [Background and autostart](docs/BACKGROUND_AND_AUTOSTART.md)
- [Console UI](docs/CONSOLE_UI.md)
- [Display test matrix](docs/DISPLAY_TEST_MATRIX.md)
- [Packaging](docs/PACKAGING.md)
- [Performance](docs/PERFORMANCE.md)
- [Release checklist](docs/RELEASE_CHECKLIST.md)
- [Reliability](docs/RELIABILITY.md)
- [Storage and clipboard](docs/STORAGE_AND_CLIPBOARD.md)

## Project history

SCapturer began as a Batch/PowerShell proof of concept. The active implementation is the C# solution in this repository; the original prototype remains under `legacy/` for historical reference.

Created by **XCON** under **X-LAB**.
