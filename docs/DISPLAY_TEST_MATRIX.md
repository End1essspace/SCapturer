# Display validation matrix

This matrix validates SCapturer's physical-coordinate model, cached-frame region overlay, topology invalidation, and recovery across common Windows display configurations.

Run the application from a Release build:

```powershell
dotnet run --project .\src\SCapturer.App\SCapturer.App.csproj -c Release
```

For every successful region capture, inspect the PNG at 100% zoom and compare its first and last pixel rows and columns with the visible selection boundary.

## Record the environment

Capture these details before each test session:

| Field | Record |
| --- | --- |
| Windows version and build | |
| GPU and driver version | |
| Local or Remote Desktop session | |
| Monitor count | |
| Per-monitor resolution | |
| Per-monitor scaling | |
| Primary monitor | |
| Virtual-desktop origin and dimensions | |
| Requested and active capture backend | |
| SCapturer build or commit | |

Retain representative PNG files and note any topology version shown by the console.

## 1. Single-monitor configurations

Test:

- 1920×1080 at 100% scaling;
- 2560×1440 at 125% scaling;
- 3840×2160 at 150% scaling.

Expected:

- console virtual bounds equal the physical desktop resolution;
- a full-desktop PNG has the same dimensions;
- region edges are pixel-aligned;
- the overlay size label matches the saved PNG dimensions;
- the saved image contains no overlay dimming, border, or size label.

## 2. Horizontal multi-monitor layouts

Test:

- primary monitor on the left;
- primary monitor on the right;
- a secondary monitor physically left of the primary;
- different resolutions across monitors;
- mixed scaling such as 100% and 150%;
- a selection spanning a monitor boundary.

Expected:

- virtual origin uses a negative X coordinate when required;
- the overlay covers each monitor exactly once;
- no gap, overlap, duplicated strip, or transparent strip appears;
- cross-monitor dragging remains continuous;
- saved dimensions match the selected physical rectangle;
- the captured pixels align with both monitor edges.

## 3. Vertical and offset layouts

Test:

- a secondary monitor above the primary;
- a secondary monitor below the primary;
- staggered top edges;
- staggered left or right edges;
- selections crossing vertical and offset boundaries.

Expected:

- monitors above the primary produce a negative Y coordinate;
- physical offsets are preserved;
- the overlay does not normalize the desktop to logical DPI units;
- selection clamps only at the complete virtual-desktop boundary;
- uncovered areas outside the real monitor rectangles are not mistaken for additional displays.

## 4. Region selection behavior

For each representative layout:

1. make a small selection on one monitor;
2. select almost the complete virtual desktop;
3. drag from bottom-right to top-left;
4. cancel with `Esc`;
5. cancel through `--cancel-region`;
6. start another selection after cancellation.

Expected:

- drag direction does not change the final rectangle;
- zero-size or accidental click-only selections do not commit a PNG;
- user cancellation produces no final PNG;
- external cancellation produces no final PNG;
- the overlay closes cleanly;
- the next region capture works without restarting SCapturer.

## 5. Topology changes during region capture

Open the overlay, then perform each applicable change:

- change a monitor resolution;
- change scaling;
- disconnect a secondary monitor;
- reconnect a monitor;
- change the primary display;
- rotate a display;
- rearrange monitors in Windows Settings.

Expected:

- active region selection is cancelled;
- status reports that display topology changed;
- no region PNG is committed;
- no `.scapturer.tmp` file remains;
- the next overlay uses the refreshed physical bounds;
- the topology version advances after the new configuration stabilizes.

## 6. Full capture during topology changes

Trigger full capture while changing resolution, reconnecting a display, or rearranging monitors.

Expected:

- the capture completes against one stable topology or retries once;
- a topology that changes again causes a visible failure rather than an unbounded retry;
- no partial or dimensionally inconsistent PNG is committed;
- a subsequent capture succeeds after topology stabilizes.

## 7. Power and session transitions

Test:

- sleep and resume;
- lock and unlock;
- user logon where applicable;
- Remote Desktop connection;
- Remote Desktop disconnection and return to the console session.

Expected:

- display topology is invalidated and refreshed;
- local or remote session state updates;
- an active overlay is cancelled during the transition;
- global hotkeys continue working after the session is usable;
- no application restart is required;
- the next capture uses the current desktop dimensions.

Remote Desktop results may differ from local-console results because Windows can expose a different virtual display topology. Record both environments separately.

## 8. Backend parity

Repeat representative single-monitor, mixed-DPI, negative-coordinate, and cross-monitor region tests with:

- `ReferenceGdiPlus`;
- `NativeGdiWic`.

Expected:

- both backends produce identical dimensions;
- region boundaries map to the same physical pixels;
- saved PNG files are fully opaque;
- native fallback is visible if WIC is unavailable;
- switching backends does not require an application restart.

## 9. Resource stability

After warm-up, repeat:

- 20 display-topology changes;
- 50 region selections cancelled before commit;
- 50 successful region captures across at least two monitor boundaries.

Observe the process with the reliability harness, Task Manager, or Process Explorer.

Expected:

- GDI and USER object counts do not grow linearly;
- process handles and threads return near their post-warmup range;
- only one capture worker remains active;
- no orphan overlay window remains;
- no stale temporary file remains in the capture directories;
- normal capture still works after the sequence.

Use [Reliability validation](RELIABILITY.md) for the automated resource gates.

## Pass criteria

A display configuration passes only when:

- full and region PNG dimensions are exact;
- selected boundaries are pixel-aligned;
- negative coordinates are preserved;
- topology changes cancel unsafe region operations;
- full capture never commits a mixed-topology image;
- recovery requires no process restart;
- both capture backends meet the same correctness contract;
- repeated changes do not show linear resource growth.

Document failures with the environment record, requested and active backend, console status, topology version, and an affected PNG when one was safely produced.
