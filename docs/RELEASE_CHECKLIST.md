# Release checklist

Complete this checklist for every public SCapturer release. Do not publish artifacts until all applicable items pass.

## Release record

- [ ] version: `________________`;
- [ ] source commit: `________________`;
- [ ] Git tag: `v________________`;
- [ ] validation machine and Windows build recorded;
- [ ] validation date recorded;
- [ ] release owner recorded.

## Source and verification

- [ ] working tree is clean;
- [ ] local `main` is synchronized with `origin/main`;
- [ ] version uses `major.minor.patch`;
- [ ] release notes describe the version being published;
- [ ] repository and distributed packages identify the license as MIT;
- [ ] Release build completes with zero warnings and zero errors;
- [ ] deterministic logic tests pass;
- [ ] default Windows reliability workload passes;
- [ ] reliability report is retained as release evidence;
- [ ] required display-layout checks are complete.

## Release build

Run without skip switches:

```powershell
.\scripts\build-release.ps1 -Version <version>
```

Confirm:

- [ ] the script exits successfully;
- [ ] no `-SkipTests`, `-SkipReliability`, or `-SkipMsi` switch was used;
- [ ] `dist\release\<version>` contains exactly the expected release files;
- [ ] generated release notes contain the correct version and date;
- [ ] temporary `dist\work\<version>` staging data was removed after success.

## Artifact integrity

Expected files:

```text
SCapturer-v<version>-win-x64-portable.zip
SCapturer-v<version>-win-x64.msi
RELEASE_NOTES.md
SHA256SUMS.txt
```

Validate:

- [ ] portable ZIP exists, opens, and is non-empty;
- [ ] MSI exists and is non-empty;
- [ ] `SHA256SUMS.txt` lists every distributable artifact;
- [ ] recalculated SHA-256 values match the checksum file;
- [ ] executable Product, Company, Author, FileVersion, and ProductVersion metadata are correct;
- [ ] executable file version is `<version>.0`;
- [ ] no `.dll`, `.pdb`, `.deps.json`, or `.runtimeconfig.json` file is distributed beside the executable;
- [ ] omission of an application icon is intentional for this release.

## Portable package

The ZIP root must contain only:

```text
SCapturer.exe
README.txt
LICENSE
```

Validate from a clean extraction directory:

- [ ] `README.txt` contains the correct version;
- [ ] `LICENSE` contains the complete MIT license text and the correct copyright holder;
- [ ] application starts without a separately installed .NET runtime;
- [ ] full-desktop capture works;
- [ ] region capture works;
- [ ] clipboard and sound settings work;
- [ ] recent captures open correctly;
- [ ] `Ctrl+Shift+H` hides and shows the console;
- [ ] closing the console with `X` resumes the listener in the background;
- [ ] launching the same executable again shows the existing console;
- [ ] autostart can be enabled and disabled;
- [ ] moving the executable produces a visible stale-autostart state;
- [ ] SCapturer can repair the stale registration;
- [ ] removing the portable folder leaves screenshots, settings, and diagnostics intact.

## MSI clean installation

Validate on a machine or user profile without an installed SCapturer MSI:

- [ ] installation succeeds without administrator elevation;
- [ ] install directory is `%LOCALAPPDATA%\Programs\X-LAB\SCapturer`;
- [ ] Start Menu shortcut is `X-LAB\SCapturer`;
- [ ] no desktop shortcut is created;
- [ ] Installed apps or Programs and Features shows one SCapturer entry;
- [ ] publisher is `X-LAB`;
- [ ] installed executable opens the management console;
- [ ] installed `LICENSE` is present beside the executable;
- [ ] full and region capture work;
- [ ] background mode works;
- [ ] second-instance activation works;
- [ ] autostart remains disabled until enabled inside SCapturer.

## MSI repair

- [ ] start SCapturer and hide the console;
- [ ] run MSI repair;
- [ ] the running instance exits gracefully before file replacement;
- [ ] repair completes without a reboot request;
- [ ] application starts normally afterward;
- [ ] settings remain intact;
- [ ] screenshots and diagnostics remain intact;
- [ ] the previous autostart preference is preserved.

## MSI upgrade

Validate with the immediately preceding published MSI:

- [ ] install the previous version;
- [ ] change at least one setting;
- [ ] enable autostart;
- [ ] leave SCapturer running in the background;
- [ ] install the new MSI;
- [ ] the previous process exits gracefully;
- [ ] upgrade completes without a reboot or locked-file prompt;
- [ ] the new executable reports the correct version;
- [ ] settings remain unchanged;
- [ ] screenshots and diagnostics remain unchanged;
- [ ] autostart remains enabled and points to the installed executable;
- [ ] only one installed-app entry remains;
- [ ] installing the older MSI over the newer version is blocked.

## MSI uninstall

- [ ] start SCapturer in the background;
- [ ] uninstall through Windows;
- [ ] SCapturer exits gracefully;
- [ ] installed executable is removed;
- [ ] installed `LICENSE` is removed;
- [ ] Start Menu shortcut is removed;
- [ ] installer-owned empty directories are removed;
- [ ] SCapturer Run-key value is removed;
- [ ] screenshots remain;
- [ ] `%LOCALAPPDATA%\SCapturer\config.json` remains;
- [ ] diagnostics remain;
- [ ] reinstalling SCapturer loads the preserved configuration.

## Publication

Before creating the public GitHub release:

- [ ] release commit is pushed;
- [ ] annotated or signed tag `v<version>` points to the verified commit;
- [ ] release title is `SCapturer v<version>`;
- [ ] generated `RELEASE_NOTES.md` is used as the release description;
- [ ] portable ZIP is attached;
- [ ] MSI is attached;
- [ ] `SHA256SUMS.txt` is attached;
- [ ] attachment names exactly match the release notes;
- [ ] the release is reviewed in draft form before publication.

## Final sign-off

- [ ] every required item above is complete;
- [ ] known limitations are stated in the release notes;
- [ ] download and checksum verification was repeated from the published release;
- [ ] repository status is clean after publication.
