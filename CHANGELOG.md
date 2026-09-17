# Changelog

All notable changes to Braceview are documented here. Versions follow semantic versioning.

## [1.2.0] — 2026-09-17

### Added

- **macOS app** for Apple Silicon and Intel (dmg and zip). Double-clicked files open in the running window, the menu bar has the usual Edit commands, and the window uses native traffic-light controls. Not yet notarized: approve the first launch in System Settings → Privacy & Security.
- **Microsoft Store package** (MSIX/AppX) for Store distribution, signed by Microsoft on submission
- Website and privacy policy at https://viveksthul15.github.io/braceview/

### Changed

- New **Aurora** look: an emerald-to-sky gradient icon and teal accents in light and dark themes
- Mermaid diagrams use matching colours in both themes
- Markdown links use the brand's blue
- Downloads, issues and release notes now live in the public repository github.com/viveksthul15/braceview

## 1.1.0 — 2026-09-17

### Added

- **CSV support** in the desktop app and the extension
  - Virtualized table that stays fast with hundreds of thousands of rows
  - Delimiter detection (comma, semicolon, tab, pipe), header row detection, quoted fields and multi-line cells
  - Column type detection with numeric and date sorting, decimal-comma numbers in semicolon files
  - Search with highlighting and row filtering; filter to a cell's value from the context menu
  - Column statistics panel with distribution chart and most common values
  - Code view with coloured columns; editing and saving in the desktop app
  - Export visible rows as JSON, Markdown table, TSV or CSV
  - Desktop installer registers `.csv`, `.tsv`, `.tab` and `.psv` as "Open with" types
- Files that aren't valid UTF-8 are opened as Windows-1252; UTF-16 files with a byte-order mark are supported; the status bar shows the encoding
- **Chrome extension** for Chrome and Edge
  - Opens JSON responses in the full viewer, including `application/*+json`, JSON Lines, JSON served as
    `text/plain`, and responses with an anti-XSSI prefix
  - Renders `.md` files and `text/markdown` responses, including on pages with strict or sandboxed CSP
  - "View as tree" button when hovering JSON code blocks on web pages, with an inline viewer
  - Viewer / Raw toggle, outline sidebar, theme switch
  - Toolbar popup to turn Braceview off per site; settings page with a first-run welcome

### Changed

- Task list check boxes and external-link arrows no longer use `data:` images

## 1.0.0 — 2026-09-17

First release.

### App

- Tabs with drag-to-reorder, middle-click close and a context menu
- Single instance: opening another file adds a tab to the running window
- Command palette (`Ctrl+K`) over commands, open tabs and recent files
- Sidebar with outline, folder listing and recent files
- Watches open files and reloads them when they change on disk
- Restores the previous session's tabs on startup
- Drag and drop, light/dark/system theme, content zoom, window state persistence
- Windows file associations for JSON and Markdown extensions, installed by the NSIS installer

### JSON

- Tree view with lazy rendering, paging for large collections, expand to level, indent guides,
  type colours, colour swatches and keyboard navigation
- Search across keys and values with match case, regex and filter mode
- Breadcrumb path with copy as JSONPath, JS accessor or jq filter
- Table view with sorting, CSV copy and reveal-in-tree
- JSONPath queries with clickable results
- CodeMirror code view with folding, linting and search
- JSONC and JSON Lines support; best-effort tree for invalid files with error position
- Format, minify, sort keys, save and save-as

### Markdown

- Preview, split and source views with scroll sync
- GitHub-flavored Markdown, alerts, footnotes, emoji, task lists and front matter
- Syntax highlighting, KaTeX math and Mermaid diagrams
- Outline with reading position, `[TOC]`, reading progress, word count and reading time
- Find in document using the CSS Custom Highlight API
- Export to HTML and PDF, print, copy as rich text
- Reading width and font options

[1.2.0]: https://github.com/viveksthul15/braceview/releases/tag/v1.2.0
