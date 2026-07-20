# Reliability validation

SCapturer uses two complementary verification layers:

1. deterministic logic tests that do not require an interactive desktop;
2. a Windows integration and resource-soak harness that exercises the complete application lifecycle.

Both layers use the production code paths. The test projects do not replace the bounded capture coordinator or introduce a separate capture implementation.

## Automated logic tests

Project:

```text
tests\SCapturer.Tests
```

Run:

```powershell
dotnet run --project .\tests\SCapturer.Tests\SCapturer.Tests.csproj -c Release
```

The executable returns exit code `0` only when every case passes.

Current coverage includes:

- hotkey parsing, formatting, validation, and duplicate rejection;
- deep settings snapshots and normalization;
- invalid settings-file backup;
- isolated application paths;
- command-line lifecycle parsing and validation;
- benchmark median and percentile calculations;
- atomic PNG collision handling;
- temporary-file cleanup after encoder failure;
- recent-capture ordering and bounds;
- autostart command construction;
- capture-pipeline work-state semantics.

The project uses a small internal runner instead of external test-framework packages. This keeps clean builds deterministic and allows the same command to run in CI or an offline Windows environment.

## Windows reliability harness

Project:

```text
tools\SCapturer.Reliability
```

Build the solution first:

```powershell
dotnet build .\SCapturer.sln -c Release
```

Run the default workload:

```powershell
dotnet run --project .\tools\SCapturer.Reliability\SCapturer.Reliability.csproj -c Release -- --captures 100 --console-cycles 30 --region-cancel-cycles 5 --process-cycles 10
```

Unless `--app` is supplied, the harness uses:

```text
src\SCapturer.App\bin\Release\net8.0-windows10.0.19041.0\SCapturer.exe
```

A published executable can be tested explicitly:

```powershell
dotnet run --project .\tools\SCapturer.Reliability\SCapturer.Reliability.csproj -c Release -- --app .\dist\publish\win-x64\SCapturer.exe --captures 1000 --console-cycles 100 --region-cancel-cycles 20 --process-cycles 25
```

The harness requires an interactive Windows desktop because full capture and region-overlay cancellation use real display APIs.

## Test isolation

The harness starts SCapturer with internal environment overrides:

```text
SCAPTURER_INSTANCE_SUFFIX
SCAPTURER_DATA_DIRECTORY
SCAPTURER_DISABLE_HOTKEYS=1
SCAPTURER_NONINTERACTIVE=1
```

This provides:

- a unique mutex and named pipe;
- an isolated settings and diagnostics directory;
- isolated Full and Snips capture folders;
- no global-hotkey collision with a normal SCapturer instance;
- non-interactive failure behavior suitable for automation.

Capture and lifecycle actions are sent through the same named-pipe command path used by ordinary secondary invocations.

## Default workload

A standard run performs:

- 5 warm-up full captures;
- 2 console show/hide warm-up cycles;
- 1 region-overlay cancellation warm-up;
- 100 measured full captures;
- 30 console show/hide cycles;
- 5 region-overlay cancellation cycles;
- 10 complete process launch, activation, hide, and graceful-exit cycles.

Full captures use the production persistence path. Region cancellation opens the actual cached-frame overlay and then sends `--cancel-region`; no final Snips PNG may be committed.

Avoid interacting with the overlay while the harness is running.

## Resource sampling

After warm-up, the harness samples:

- GDI object count;
- USER object count;
- process handle count;
- thread count;
- private memory;
- working set.

The baseline is captured only after full capture, IPC, console allocation, WinForms overlay creation, and region cancellation have all executed at least once. This prevents first-use Windows and .NET caches from being treated as leaks.

The final sample is taken after a two-second settling interval.

SCapturer allocates the management console once and reuses that attachment for subsequent hide/show operations. The harness therefore tests repeated visibility changes without intentionally recreating console streams on every cycle.

## Default resource gates

| Resource | Maximum final delta |
| --- | ---: |
| GDI objects | `+8` |
| USER objects | `+8` |
| Process handles | `+32` |
| Threads | `+3` |
| Private memory | greater of `48 MB` or `20%` of baseline |
| Working set | greater of `64 MB` or `30%` of baseline |

These are non-growth gates rather than fixed steady-state budgets. They are intended to detect linear accumulation while allowing runtime caches and normal Windows working-set variation.

## Output

Each run creates:

```text
artifacts\reliability\<timestamp>\
```

Expected contents:

```text
soak-summary.json
resource-samples.jsonl
reliability-report.md
app-data\
captures\
```

Exit codes:

| Code | Meaning |
| ---: | --- |
| `0` | All reliability gates passed |
| `1` | The workload completed, but one or more gates failed |
| `2` | The harness could not start or the environment was invalid |

## Acceptance criteria

A reliability run is acceptable only when:

- the Release solution builds without warnings or errors;
- every automated logic test passes;
- all requested full captures complete;
- cancelled region selections produce no final PNG;
- no `.scapturer.tmp` file remains;
- IPC actions complete successfully;
- every repeated primary process exits gracefully;
- every resource-delta gate passes;
- the generated report is retained as release evidence.

One passing run is not proof against every environment-specific failure. Repeat the workload on relevant Windows versions, display layouts, and packaged builds before release.
