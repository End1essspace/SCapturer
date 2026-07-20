# Capture backend comparison

SCapturer provides two complete Windows capture backends behind the same `ICaptureBackend` contract. They share the same display-topology model, capture queue, persistence rules, clipboard path, and region-selection workflow.

## Implementations

| Backend | Pixel acquisition | Region crop | PNG encoding |
| --- | --- | --- | --- |
| Reference GDI+ | `Graphics.CopyFromScreen` into `System.Drawing.Bitmap` | GDI+ `DrawImage` | `Bitmap.Save` |
| Native GDI + WIC | `BitBlt` into a top-down 32-bit `CreateDIBSection` | memory-DC `BitBlt` | WIC `IWICBitmapFrameEncode::WritePixels` |

The reference implementation is the compatibility and rollback path. The native implementation keeps pixels in one BGRA buffer that can also be exposed through a non-owning `Bitmap` view for the overlay and clipboard path.

## Backend modes

SCapturer stores one of three modes:

| Mode | Behavior |
| --- | --- |
| `ReferenceGdiPlus` | Always use the reference backend |
| `NativeGdiWic` | Use the native backend when WIC is available; otherwise use the reference backend and expose the fallback reason |
| `Auto` | Prefer the native backend when available; otherwise use the reference backend |

New settings default to `ReferenceGdiPlus`. Users can change the requested mode from **Capture Settings**, and the Diagnostics page can persist the recommendation produced by a comparison run.

Native availability is probed through Windows Imaging Component initialization. A failed probe does not prevent capture because the reference backend remains available.

## Running a comparison

Open **Diagnostics and Benchmark** and select the backend-comparison action.

The application runs the following workload for each backend:

- one warm-up full-desktop capture;
- ten measured full-desktop captures;
- clipboard publication disabled;
- capture sound disabled;
- per-capture diagnostics disabled;
- PNG persistence enabled;
- generated benchmark images deleted after sampling when possible.

Both sides use the same configured destination volume and benchmark settings. Keep the display layout and desktop workload unchanged for the duration of the run; the benchmark does not reserve the machine or freeze display topology between the two series.

Reports are written to:

```text
%LOCALAPPDATA%\SCapturer\diagnostics\benchmarks\backend-comparison_*.json
```

Each report includes environment metadata, both sample sets, summary statistics, the recommended mode, and the exact decision reason.

## Metrics

The comparison evaluates complete capture latency and managed allocation behavior. The report contains:

- median total latency;
- p95 total latency;
- fastest and slowest total latency;
- median pixel-acquisition time;
- median PNG-persistence time;
- average managed bytes allocated;
- average PNG file size.

With the default ten measured samples, SCapturer uses the nearest-rank percentile calculation, so p95 resolves to the slowest sample in that series.

## Decision rule

The native backend is recommended only when both conditions are satisfied:

1. native median total latency is no more than 5% slower than reference;
2. native improves either p95 total latency or average managed allocations by at least 20%.

Improvement is calculated as:

```text
(reference - native) / reference × 100
```

A passing result persists `NativeGdiWic`. A non-passing result persists `ReferenceGdiPlus`.

The decision is intentionally conservative: a lower-level implementation is not selected merely because one isolated phase is faster.

## Interpreting results

Treat the generated recommendation as evidence for the tested machine and display layout, not as a universal ranking.

Before accepting a result:

- close unrelated high-CPU and high-I/O workloads;
- keep the monitor layout unchanged;
- run from the same build configuration;
- compare complete latency, not only pixel acquisition;
- inspect repeated runs when the result is close to a threshold;
- retain the JSON report used for a release decision.

PNG persistence may dominate the end-to-end result on large or visually complex desktops. A faster `BitBlt` path can therefore produce only a small total-latency improvement.

## Correctness requirements

Performance does not override capture correctness. Validate both backends for:

- exact full-desktop and region dimensions;
- pixel-aligned region boundaries;
- negative virtual-desktop coordinates;
- mixed-DPI and multi-monitor layouts;
- cancellation when topology changes during region selection;
- opaque alpha in saved PNG files;
- equivalent clipboard output;
- no linear growth in GDI objects, USER objects, process handles, COM state, or temporary files.

Use [Display validation matrix](DISPLAY_TEST_MATRIX.md) for display-specific checks and [Reliability validation](RELIABILITY.md) for repeated resource testing.

## Rollback

The reference backend is always compiled into the application. Select `ReferenceGdiPlus` from **Capture Settings** to roll back immediately.

Selecting `NativeGdiWic` or `Auto` on a system where WIC initialization fails also produces an explicit reference fallback rather than disabling capture.
