# What's New

A plain-language, timelined summary of new and meaningfully improved capabilities across
**wafermap** and **tsmap** — for people using the app, or building on the library, not for
contributors. Only genuinely user/adopter-facing capabilities are listed here: internal
refactors, dependency bumps, docs-only fixes, and process/tooling changes are skipped, even
when they're a real fix worth having. For the complete technical record of everything that
shipped, see each project's own changelog:
[wafermap](https://github.com/wafertools/wafermap/blob/main/CHANGELOG.md) ·
[tsmap](https://github.com/wafertools/tsmap/blob/main/CHANGELOG.md).

Covers everything since both projects moved to the `wafertools` GitHub org and
`@wafertools` npm scope (2026-08-01) — see each changelog directly for anything earlier.

---

## 2026-08-28 — Bin definitions, wafer diameter & edge-exclusion overrides, reports open in-app

**tsmap** can now read and edit **bin definitions** — human-readable names for hard/soft bin
numbers, plus which ones count as pass. STDF/ATDF already carry this in their own HBR/SBR
records; CSV/JSON/Parquet, which have no such record at all, can now load definitions from a
CSV via a new **Load bin definitions…** button in the column mapping step, and any format can
save/load them afterwards from **Lot ▾ → Bin definitions…**.

A new **Lot ▾ → Diameter & edge exclusion…** dialog lets you override the wafer size tsmap
infers from the data and add an edge-exclusion band — dies within it are dropped from yield
and shown dimmed on the map — useful when the data is too sparse for a reliable automatic
guess. Both are also settable at launch via `--wafer-diameter`/`--edge-exclusion`. A new
**Help → Definitions file formats…** dialog saves a filled-in example of any of tsmap's three
definitions-file types (tests, splits, bins) without needing to load data first.

**wafermap**'s summary and lot reports now open inside the app instead of a separate browser
tab — closing a longstanding issue where the report could open successfully but sit hidden in
an unfocused, sometimes-minimized window, looking like the button had silently done nothing.
The new in-app view has its own Print/Save-as-PDF button, plus an "Open as full page" link for
anyone who wants a real separate page. The built-in help guide picked up the same in-app
Print/PDF button, a combined table of contents that no longer reads oddly across a host app's
own guide content, and a find-in-page search box for the one case where a browser's own Ctrl+F
can't reach it (an embedded desktop app like tsmap).

CSV exports (die lists, test/bin/split tables) are now protected against formula injection — a
cell value starting with `=`, `+`, `-`, or `@`, sourced from metadata this library didn't
originate (an operator's free-text note, a MES field), used to be read as a live formula by
Excel/Sheets/LibreOffice on open; it's now safely escaped.

*See: [tsmap v0.1.31](https://github.com/wafertools/tsmap/blob/main/CHANGELOG.md#0131--2026-08-28),
[wafermap v0.26.0](https://github.com/wafertools/wafermap/blob/main/CHANGELOG.md#0260--2026-08-28)*

## 2026-08-23 — A live demo of launching tsmap from your own page; dark themes readable again

If you're wiring tsmap into a data-selection tool of your own, there's now a **[live
demo](https://wafertools.github.io/tsmap/demos/open-from-link.html)** of the launch-from-a-link
flow: pick a sample dataset and get real, working links for both the browser build
(`?dataUrl=`) and the desktop app (`tsmap://open?url=...`), plus the equivalent command line.
Previously this was described in the docs but there was nothing to click.

**tsmap**'s three dark themes (Dark, Nord, Solarized Dark) had borders and secondary text
that measured as low as 1.04:1 against their own backgrounds — effectively invisible rather
than deliberately subtle, with the empty-state "Supports STDF, ATDF…" subtitle the most
visible casualty. Every one of those values has been re-picked per theme and checked against
each surface it actually renders on. Light themes were unaffected. tsmap also picked up a new
app and tab icon — a wafer glyph in place of the old placeholder.

Two loading fixes: a gzipped sample file could fail to open in the **browser build** with
"The compressed data was not valid" (the server had already decompressed it, and tsmap
decompressed it a second time); and a sparsely-positioned wafer whose dies all landed in one
row or column could render visibly stretched, non-square dies — cosmetic only, die counts,
bins and yield were always correct.

*See: [tsmap v0.1.28](https://github.com/wafertools/tsmap/blob/main/CHANGELOG.md#0128--2026-08-23),
[wafermap v0.23.1](https://github.com/wafertools/wafermap/blob/main/CHANGELOG.md#0231--2026-08-18)*

## 2026-08-16 — Filter large batches before loading; wafer maps handle data with no position info

**tsmap** can now scan a large batch of files — a whole directory of lots, say — for their
lot/wafer metadata *before* loading any of them, and show the results in a sortable,
filterable table. Pick just the files you actually want from many, rather than loading
everything and sorting it out afterward. The scan reads only header-level information (lot
ID, part type, tester, wafer count, dates, etc), so it stays fast even across a large batch. A
filter can be saved and reloaded, and the last one you used is remembered automatically.

Both **wafermap** and **tsmap** now handle wafer test data that has no reported die x/y positions
at all — a wafer-number-only test log, or a lot where some dies report a position and others
don't. This used to be silently dropped; it's now kept, with the wafer shown as a bin
breakdown or value histogram instead of a fabricated map (which would risk being misread as
real spatial data), plus a full exportable die list. Yield, bin counts, and per-test
statistics are unaffected either way — only spatial analysis (rings, quadrants, clustering)
doesn't apply to a die with no position.

Also: a pass over keyboard and screen-reader accessibility across wafermap's summary panel,
chart legends, and die-list tables.

*See: [tsmap v0.1.27](https://github.com/wafertools/tsmap/blob/main/CHANGELOG.md#0127--2026-08-16),
[wafermap v0.23.0](https://github.com/wafertools/wafermap/blob/main/CHANGELOG.md#0230--2026-08-16)*

## 2026-08-15 — Parquet support, load data straight from a URL, native file associations

**tsmap** now opens `.parquet` files alongside STDF/ATDF/CSV/JSON, through the same column
mapping used for CSV/JSON — useful if your data pipeline already lands in a columnar
format rather than raw test-log files.

If your data lives behind an API or on a server rather than on your own filesystem, tsmap
can now fetch and load it directly: `tsmap --url <url> --url-format <format>` on desktop, or
a `?dataUrl=&dataFormat=` link for the browser build — no need to download a file by hand
first. A caller application (your own data-selection tool, a script) can hand tsmap a URL and
have it open immediately. Header-based authentication is supported on desktop for APIs that
need it. A `tsmap://open?url=...` link lets a web page launch the desktop app directly with
data to load, and on Windows/macOS/Linux tsmap can now register itself as the default handler
for `.stdf`/`.atdf`/`.parquet` files, so double-clicking one in a file manager just opens it.

*See: [tsmap v0.1.26](https://github.com/wafertools/tsmap/blob/main/CHANGELOG.md#0126--2026-08-15)*

## 2026-08-09 — A full accessibility pass; test numbers stay stable when you reorder columns

**tsmap** had a full UI and accessibility review: keyboard focus is now visible everywhere,
modal dialogs trap Tab and restore focus correctly, every theme now meets WCAG AA contrast
for primary buttons, and several dialogs gained proper Escape/Enter/backdrop-click handling
that some had been missing.

For CSV/JSON data, test numbers are now a stable identity derived from the test's own name or
source column, rather than the order it happened to appear in the file — so reordering or
adding/removing columns no longer silently reshuffles which test is which. Loading a
previously-saved test selection now recovers a renumbered test by matching its name instead
of losing track of it.

*See: [tsmap v0.1.25](https://github.com/wafertools/tsmap/blob/main/CHANGELOG.md#0125--2026-08-09)*

## 2026-08-04 to 2026-08-07 — wafermap tells you when it's had to skip something; a downloadable offline examples package

**wafermap** now surfaces its own advisories directly in the UI — a warning indicator appears
whenever, for example, it had to guess at wafer geometry from the data, or capped analysis at
250 tests and skipped the rest. Previously some of these were invisible unless you went
looking; now there's always a visible signal when something might be off, in both the
standalone map view and **tsmap**, which picked this up automatically. Findings that used to
repeat the same fact several times (a hard-bin row, its identical soft-bin twin, and a yield
row all restating one edge failure) now collapse into one.

For anyone evaluating or integrating **wafermap** itself, every example now ships as a
downloadable, offline-capable archive — unzip and open, no build step, no network required,
which matters on a locked-down fab network.

*See: [tsmap v0.1.24](https://github.com/wafertools/tsmap/blob/main/CHANGELOG.md#0124--2026-08-07),
[wafermap v0.22.0](https://github.com/wafertools/wafermap/blob/main/CHANGELOG.md#0220--2026-08-04)*

## 2026-08-02 — More accurate edge-die yield

**tsmap**'s displayed yield numbers are now more accurate for wafers where the physical
geometry has to be inferred from the data rather than read from an explicit map file: dies at
the wafer edge that were actually tested were sometimes wrongly treated as falling outside
the wafer and dropped from yield and statistics entirely. Every affected die was a genuine
tested site, so recovering them means displayed yield can shift slightly, and is now correct.
