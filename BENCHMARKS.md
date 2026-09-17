# Braceview benchmarks

How long Braceview takes on real-world-sized files, measured on an ordinary laptop rather than a workstation. The test files are generated (a 55 MB CSV with a million rows, a 20 MB JSON API dump, a 600-section Markdown report) so every run uses the same data.

Measured 2026-09-17 on AMD Ryzen 5 7530U with Radeon Graphics, 15 GB RAM, Windows 11. Desktop: packaged 1.2.1 app, 5 cold starts per scenario, medians. Extension: Chromium 152.0.7977.78 (Electron 44.4.1), 5 runs per page, medians.


## Desktop app

### Cold start to readable content

Time from launching the exe until the content is on screen. Memory is the private memory of all Braceview processes (main, GPU, utility, renderer).

| Scenario | Ready (median) | Range | Memory | DOM nodes |
| --- | --- | --- | --- | --- |
| No file (welcome screen) | 840 ms | 789 ms – 1.31 s | 125 MB | 337 |
| JSON 1.3 KB | 982 ms | 971 ms – 1.07 s | 163 MB | 987 |
| JSON 0.4 MB (1k objects) | 1.14 s | 1.09 s – 1.29 s | 188 MB | 9,591 |
| JSON 20 MB (50k objects) | 1.76 s | 1.69 s – 1.81 s | 363 MB | 8,009 |
| CSV 480 rows | 1.02 s | 1.02 s – 1.21 s | 163 MB | 2,020 |
| CSV 200k rows (11 MB) | 1.29 s | 1.21 s – 1.40 s | 326 MB | 1,692 |
| CSV 1M rows (55 MB) | 2.39 s | 2.31 s – 2.46 s | 682 MB | 1,690 |
| Markdown with diagram | 1.96 s (text 1.10 s) | 1.94 s – 2.23 s | 208 MB | 1,676 |
| Markdown 0.5 MB (600 sections) | 2.14 s | 2.08 s – 2.44 s | 293 MB | 161,843 |

### Interactions on large files

| CSV action | 200k rows (11 MB) | 1M rows (55 MB) |
| --- | --- | --- |
| Sort numeric column | 122 ms | 398 ms |
| Sort text column | 127 ms | 248 ms |
| Search + filter rows | 104 ms | 379 ms |
| Clear filter | 44 ms | 95 ms |
| Column statistics | 87 ms | 347 ms |
| Scroll jump, median | 17 ms | 16 ms |
| Scroll jump, worst | 32 ms | 32 ms |
| Memory after all actions | 716 MB | 890 MB |

| JSON action (20 MB, 50k objects) | Time |
| --- | --- |
| Expand all (tree shows 500 items per level, then "show more") | 75 ms |
| Collapse all | 7 ms |
| Expand to level 3 | 78 ms |
| Search (16701 matches) | 431 ms |
| Next match | 93 ms |
| JSONPath query (4,919 results) | 71 ms |
| Switch to table view | 2 ms |
| Switch to code view | 243 ms |
| Memory after all actions | 675 MB |

| Markdown action (0.5 MB, 600 sections) | Time |
| --- | --- |
| Find in document (2766 matches) | 270 ms |
| Open split editor | 218 ms |
| Memory | 301 MB |

### Idle and size

| Metric | Value |
| --- | --- |
| CPU while idle, 3 files open (15 s) | 0% of one core |
| Memory idle, 3 small files open | 218 MB |
| Installer / portable exe | 108.6 MB / 108.3 MB |
| Microsoft Store package | 159.9 MB |
| macOS dmg (Apple Silicon) | 125.4 MB |
| Installed on disk | 386 MB (Electron runtime + 18 MB app) |
| UI code loaded at start | 1.4 MB JS + 70.4 KB CSS (diagrams 5.3 MB, loaded only when needed) |

## Chrome extension

"Plain" is the browser without Braceview (raw text). "Braceview" is the time until the viewer shows content, measured from navigation start. Memory is the tab's renderer process.

| Page | Plain: ready | Braceview: ready | Plain: memory | Braceview: memory |
| --- | --- | --- | --- | --- |
| JSON API response 1.3 KB | 128 ms | 451 ms | 24 MB | 47 MB |
| JSON 0.4 MB (1k objects) | 274 ms | 708 ms | 47 MB | 92 MB |
| JSON 20 MB (50k objects) | 8.43 s | 9.69 s | 1824 MB | 1295 MB |
| CSV 480 rows (raw file) | 129 ms | 497 ms | 25 MB | 70 MB |
| CSV 200k rows, 11 MB (raw file) | 3.72 s | 5.68 s | 492 MB | 852 MB |
| Markdown report with diagram | 149 ms | 618 ms | 27 MB | 86 MB |
| Markdown diagram rendered | 146 ms | 1.36 s | 26 MB | 85 MB |
| Markdown 0.5 MB (600 sections) | 281 ms | 1.98 s | 44 MB | 198 MB |

**Ordinary web pages** (not JSON/CSV/Markdown): load event 108 ms without vs 111 ms with the extension; tab memory 28 MB vs 29 MB, JS heap 1.1 vs 1.5 MB. Only the 4.7 KB detector runs there.

| Extension size | Value |
| --- | --- |
| Detector (runs on every page) | 4.7 KB |
| Service worker | 0.4 KB |
| Viewer code loaded up front (only on taken-over pages) | 1.41 MB |
| Diagram library (only for Markdown with Mermaid) | 5.03 MB |
| Unpacked / zip | 8.5 MB / 0 MB |

## Why it is fast

1. **CSV sorting** — each column gets one numeric sort key per row, computed once and cached: numbers are parsed once, ISO dates take a fast path, and text is ranked by sorting only its distinct values with one shared `Intl.Collator`. Rows are then placed with a stable counting sort, so there are no per-comparison callbacks.
2. **Column statistics** — rows are read in file order, numbers are sorted natively in a `Float64Array`, the most common values come from one pass, and results are cached until the filter changes.
3. **String hashing** — distinct values are counted with a hash computed in JavaScript rather than by using cell strings as `Map` keys. In Chromium, with a large file loaded, the engine hashes millions of fresh substrings about 30× slower the first time.
4. **Large Markdown** — the sanitizer hands its DOM straight to the page, and long documents let the browser skip layout for off-screen blocks (`content-visibility: auto`).
5. **Little code up front** — both apps are split into ES modules, so about 1 MB of code-editor grammars loads only when a code block needs colouring.
6. **Known slow spot: 20 MB JSON in the extension** — most of that time is the browser receiving and laying out the raw text before the viewer takes over.

## Caveats

- The laptop display was off during the run, so the browser produced no frames: scroll timings cover script + layout, not paint.
- The extension was measured in Electron's Chromium (same engine as Chrome, same extension code), not in Chrome itself.
- Cold starts are medians of 5 launches; each interaction was timed once per run, on a freshly launched app.
- Interaction timings subtract the UI's own input debounce (80 ms stats, 250 ms JSONPath).
