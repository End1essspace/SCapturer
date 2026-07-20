# SCapturer v{{VERSION}}

Released: {{DATE}}

SCapturer is a Windows screenshot developer utility by **XCON**, built under **X-LAB**. It provides full virtual-desktop capture, cached-frame region capture, configurable global hotkeys, background operation, and detailed runtime diagnostics.

## Included in this release

- lossless full-desktop capture across the physical Windows virtual desktop;
- rectangular region capture from one cached desktop frame;
- Per-Monitor V2, mixed-DPI, and negative-coordinate handling;
- Reference GDI+ and Native GDI + WIC capture backends;
- atomic PNG persistence with storage fallback and bounded clipboard retry;
- configurable global hotkeys;
- recent-capture browsing;
- background operation, single-instance command forwarding, and opt-in autostart;
- backend benchmarks and Windows reliability validation;
- portable `win-x64` package and per-user MSI installer.

## Downloads

| File | Use |
| --- | --- |
| `SCapturer-v{{VERSION}}-win-x64-portable.zip` | Portable deployment |
| `SCapturer-v{{VERSION}}-win-x64.msi` | Per-user Windows installation |
| `SHA256SUMS.txt` | SHA-256 verification |

Both distributions contain the same self-contained `SCapturer.exe`. A separate .NET runtime is not required.

## System requirements

- Windows 11 x64;
- Windows 10 version 2004 or newer on a best-effort basis;
- an interactive desktop session for screen capture.

## Installation

### Portable

1. Extract the ZIP to a stable folder.
2. Run `SCapturer.exe`.
3. Keep the executable in that location when using Windows autostart.

Portable extraction does not enable autostart or make installer-owned registry changes.

### MSI

Run:

```text
SCapturer-v{{VERSION}}-win-x64.msi
```

The MSI installs for the current user without administrator elevation:

```text
%LOCALAPPDATA%\Programs\X-LAB\SCapturer
```

It creates the Start Menu shortcut `X-LAB\SCapturer`. Windows autostart remains disabled until enabled from **Background and Startup**.

## Default hotkeys

| Shortcut | Action |
| --- | --- |
| `Ctrl + Shift + G` | Capture the complete virtual desktop |
| `Ctrl + Shift + S` | Capture a selected rectangular region |
| `Ctrl + Shift + H` | Show or hide the management console |
| `Ctrl + Shift + Q` | Exit gracefully |

Hotkeys can be changed inside SCapturer.

## Upgrade and uninstall

The MSI requests graceful shutdown before repair, upgrade, or uninstall.

Upgrade and repair preserve:

- settings;
- screenshots;
- diagnostics;
- the user's autostart preference.

Uninstall removes the application, Start Menu shortcut, and SCapturer autostart registration. It preserves screenshots, settings, and diagnostics.

Removing a portable copy only requires exiting SCapturer and deleting its folder. User data remains in the normal SCapturer data locations.

## License

SCapturer is distributed under the MIT License.

The complete license text is included in the repository, portable ZIP, and MSI installation.

## Verification

Verify downloaded artifacts against `SHA256SUMS.txt` before use.

This release was built from one verified self-contained `win-x64` publish output and packaged as both portable ZIP and per-user MSI.

## Known limitations

- no application icon is configured in this release;
- Windows 10 support is best effort;
- Remote Desktop and unusual mixed-DPI layouts can expose environment-specific Windows capture behavior.
