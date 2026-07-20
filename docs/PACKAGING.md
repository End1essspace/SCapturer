# Release packaging

SCapturer ships two Windows x64 distributions from the same verified publish output:

| Artifact | Purpose |
| --- | --- |
| `SCapturer-v<version>-win-x64-portable.zip` | Portable use without an installer |
| `SCapturer-v<version>-win-x64.msi` | Per-user installation with Start Menu integration |

Both distributions contain the same self-contained, single-file `SCapturer.exe`.

## Release identity

| Field | Value |
| --- | --- |
| Product | SCapturer |
| Publisher | X-LAB |
| Author | XCON |
| Version format | `major.minor.patch` |
| Runtime | .NET 8, self-contained |
| Architecture | `win-x64` |
| Primary target | Windows 11 x64 |
| Compatibility target | Windows 10 version 2004 or newer, best effort |

An application icon is not configured in the current package. This is intentional and does not affect the packaging layout.

## Build requirements

The release machine needs:

- Windows;
- the .NET 8 SDK;
- WiX Toolset 3.14 for MSI generation.

The generated executable includes the .NET runtime, so target computers do not need a separate .NET installation.

The release script looks for `candle.exe` and `light.exe` in `PATH`, the `WIX` environment variable, and standard WiX 3.14 or 3.11 installation directories.

## Build a release candidate

Run from the repository root:

```powershell
.\scripts\build-release.ps1 -Version 0.1.0
```

A normal release build performs:

1. clean;
2. restore;
3. Release build with the requested version;
4. automated logic tests;
5. the default Windows reliability workload;
6. self-contained single-file publish;
7. portable package creation;
8. MSI compilation and linking;
9. release-note generation;
10. SHA-256 generation;
11. artifact validation.

A release candidate must be built without skip switches.

### Development switches

The following switches are available for local iteration:

| Switch | Effect |
| --- | --- |
| `-SkipTests` | Skip the deterministic logic-test executable |
| `-SkipReliability` | Skip the Windows reliability workload |
| `-SkipMsi` | Build the portable package without compiling the MSI |

Example:

```powershell
.\scripts\build-release.ps1 -Version 0.1.0 -SkipReliability -SkipMsi
```

Artifacts produced with any skip switch are not complete release candidates.

## Output

A complete build creates:

```text
dist\release\0.1.0\
├─ SCapturer-v0.1.0-win-x64-portable.zip
├─ SCapturer-v0.1.0-win-x64.msi
├─ RELEASE_NOTES.md
└─ SHA256SUMS.txt
```

Temporary publish, portable-staging, and WiX object directories are kept under `dist\work\<version>` during the build and removed after success.

## Publish contract

The publish profile is stored at:

```text
src\SCapturer.App\Properties\PublishProfiles\win-x64.pubxml
```

It configures:

- `Release`;
- `win-x64`;
- self-contained deployment;
- single-file publishing;
- native-library extraction from the executable;
- trimming disabled;
- ReadyToRun disabled;
- debug symbols disabled.

The .NET SDK can still emit optional portable PDB files for referenced projects. The release script removes `*.pdb` files from the staging directory and then enforces this invariant:

```text
The deployable publish directory contains exactly one non-empty file:
SCapturer.exe
```

The script also verifies that the executable file version matches `<version>.0`.

## Portable package

The portable ZIP contains exactly these files at its root:

```text
SCapturer.exe
README.txt
LICENSE
```

The included `LICENSE` file contains the complete MIT license text. Extraction does not modify the registry or enable autostart.

For stable autostart behavior, keep `SCapturer.exe` in a permanent folder. When the executable moves, SCapturer identifies the existing Run-key command as stale and can repair it from **Background and Startup**.

To remove the portable copy:

1. exit SCapturer gracefully;
2. delete the portable folder.

Screenshots, settings, and diagnostics remain in their user-data locations.

## MSI package

WiX source:

```text
packaging\windows\SCapturer.wxs
```

The MSI is permanently scoped to the current user:

| Property | Behavior |
| --- | --- |
| Elevation | Not required |
| Install directory | `%LOCALAPPDATA%\Programs\X-LAB\SCapturer` |
| Start Menu shortcut | `X-LAB\SCapturer` |
| Desktop shortcut | Not created |
| Autostart | Remains opt-in inside SCapturer |

The application component uses an HKCU installer marker as its Windows Installer key path. Per-user installation and Start Menu directories are removed when they become empty.

WiX validation remains enabled except for `ICE91`. That warning is suppressed deliberately because the package is explicitly and permanently `perUser`; it describes a hypothetical per-machine use of the same directory tree.

## Maintenance commands

The installer uses internal command-line actions before replacing or removing the executable:

| Operation | Command | Autostart behavior |
| --- | --- | --- |
| Repair or major upgrade | `SCapturer.exe --shutdown-for-update` | Preserved |
| Real uninstall | `SCapturer.exe --prepare-uninstall` | Removed |

Both commands request graceful shutdown through the normal single-instance channel and wait for the instance mutex to be released. The maintenance process returns a failure code when shutdown does not complete within 45 seconds.

These arguments are installer-facing operations, not normal interactive commands.

## Upgrade policy

The package uses:

- one stable `UpgradeCode`;
- a generated `ProductCode` for each build;
- semantic three-part versions such as `0.1.0`, `0.1.1`, `0.2.0`, and `1.0.0`.

Installing a newer MSI performs a major upgrade. Installing an older MSI over a newer version is blocked.

Repair and upgrade preserve the user's autostart preference. The executable path remains stable across versions.

## Uninstall data policy

Uninstall removes:

- the installed executable;
- the installed `LICENSE` file;
- the Start Menu shortcut;
- empty installer-owned directories;
- the SCapturer current-user autostart value.

Uninstall preserves user-owned data:

```text
%USERPROFILE%\Pictures\SCapturer\
%LOCALAPPDATA%\SCapturer\config.json
%LOCALAPPDATA%\SCapturer\diagnostics\
```

The installer must never silently delete screenshots, settings, or diagnostic evidence.

## Manual executable publish

To publish only the self-contained executable:

```powershell
dotnet publish .\src\SCapturer.App\SCapturer.App.csproj -p:PublishProfile=win-x64 -p:Version=0.1.0 -p:AssemblyVersion=0.1.0.0 -p:FileVersion=0.1.0.0 -p:InformationalVersion=0.1.0 -o .\dist\publish\win-x64
```

This command does not create the portable ZIP, MSI, release notes, checksums, or release-verification evidence.

## Release verification

Before publishing, complete [Release checklist](RELEASE_CHECKLIST.md). Keep the following as release evidence:

- the exact source commit;
- automated test output;
- the reliability report;
- portable and MSI validation results;
- confirmation that both distributions include the MIT license text;
- `RELEASE_NOTES.md`;
- `SHA256SUMS.txt`.
