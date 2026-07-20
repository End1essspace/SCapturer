SCapturer v{{VERSION}}
====================

Windows screenshot developer utility by XCON, built under X-LAB.

PACKAGE CONTENTS
----------------
SCapturer.exe
README.txt
LICENSE

SCapturer is self-contained. A separate .NET installation is not required.

FIRST RUN
---------
1. Extract both files to a permanent folder.
2. Run SCapturer.exe.
3. Use the management console to review capture, hotkey, storage,
   diagnostics, and background settings.

Keep SCapturer.exe in a stable location when Windows autostart is enabled.
Moving the executable makes the existing autostart registration stale;
SCapturer can repair it from Background and Startup.

DEFAULT HOTKEYS
---------------
Ctrl+Shift+G  Capture the complete virtual desktop
Ctrl+Shift+S  Capture a selected rectangular region
Ctrl+Shift+H  Show or hide the management console
Ctrl+Shift+Q  Exit SCapturer gracefully

Hotkeys can be changed from the Hotkeys page.

BACKGROUND OPERATION
--------------------
Hiding the console does not stop capture services or global hotkeys.

To show an existing background instance:
- press Ctrl+Shift+H; or
- run SCapturer.exe again.

Closing the native console window with X performs a background handoff.
The visible console process closes, and SCapturer resumes hidden.

STORAGE
-------
Full captures:
%USERPROFILE%\Pictures\SCapturer\Full

Region captures:
%USERPROFILE%\Pictures\SCapturer\Snips

Settings:
%LOCALAPPDATA%\SCapturer\config.json

Diagnostics:
%LOCALAPPDATA%\SCapturer\diagnostics

Capture folders can be changed from Save Locations.

COMMAND-LINE ACTIONS
--------------------
SCapturer.exe --background
SCapturer.exe --show
SCapturer.exe --hide
SCapturer.exe --toggle-console
SCapturer.exe --capture-full
SCapturer.exe --capture-region
SCapturer.exe --cancel-region
SCapturer.exe --exit

SCapturer accepts one action per invocation. When an instance is already
running, the new process forwards the action and exits.

LICENSE
-------
SCapturer is distributed under the MIT License.

The complete license text is included in the LICENSE file.

UPDATING THE PORTABLE COPY
--------------------------
1. Exit SCapturer with Ctrl+Shift+Q or SCapturer.exe --exit.
2. Replace SCapturer.exe with the newer portable executable.
3. Start SCapturer again.

Settings, screenshots, diagnostics, and the current autostart preference are
stored outside the portable folder and remain available after replacement.

PORTABLE REMOVAL
----------------
1. Exit SCapturer gracefully.
2. Disable autostart from Background and Startup when it is enabled.
3. Delete the portable folder.

Removing the portable files does not delete screenshots, settings, or
diagnostics.
