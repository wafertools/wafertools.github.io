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

## 2026-09-12 — A clearer default colour scale for value and stacked maps

**Value maps and stacked maps now use the Viridis colour scale by default, and they will look
different.** The old scale ran blue → cyan → yellow → red. That is a rainbow, and rainbows have a
specific problem for reading data: their brightness does not rise steadily. Cyan and yellow are
both near maximum brightness while blue and red are much darker, so two genuinely different
readings could look equally intense — and because hue changes fast around cyan and yellow, the
map drew ring-shaped boundaries that no process step put there. Engineers told us they could not
judge how far apart two colours were. Viridis brightens steadily from one end to the other, so
the colour you see corresponds to the value you have, and a dense grid of dies shows small
differences honestly.

On a stacked map it also puts the emphasis the right way round. The healthy bulk of the wafer —
where nothing failed — now sits back as dark ground, and the edge ring, scratch or cluster you
are actually hunting for is the bright thing on it. Previously that was inverted: the good area
glowed and the defects were dark specks.

**If you preferred the old look, it is still there.** Pick **Jet** from the Colour scheme menu —
it is the same rainbow family, now honestly labelled as one, alongside a note that rainbow scales
trade read accuracy for familiarity.

**Every scale now runs the same direction: dark for low, bright for high.** Four of them —
Cividis, Plasma, Inferno and Greyscale — had been running inverted, which meant switching colour
scheme silently flipped which end of your wafer looked "hot". They now match their published
definitions, so a scale you recognise from elsewhere reads the way you expect.

**New Reverse gradient option.** Some parameters have their interesting end at the bottom, and
monochrome print has its own convention that more ink means more. One tick box in the same Colour
scheme menu flips whichever scale you have chosen, including one your own application registered.
tsmap remembers the setting across restarts, alongside your bin and value colour choices.

**Also new: the Mako colour scale**, a blue-to-pale-green alternative with a little more
separation at the top end than Viridis.

---

## 2026-09-11 — Bin colours you can trust; STDF and ATDF together; host tsmap on your own intranet

**Bin colours now follow pass and fail, and different bins no longer share a colour.** wafermap
used to pick a bin's colour from its bin number, so two bins could be drawn in the same colour and
bin 1 came out green even when your pass bins said it failed. Colour now follows the pass bins
you actually use — passing bins take green pass shades, failing bins the rest, ranked by how many
dies they hold — and the palettes were re-chosen by measurement so every pair stays distinguishable,
including a colour-blind-safe set. When a lot has more bins than a palette can separate, the map
says which bins share a colour rather than letting you assume they differ. **Bin maps therefore
look different from before**; the change is deliberate. Bin and value colours are now separate
choices, and tsmap remembers both across sessions; a bin definitions file can also set a bin's
colour.

**Two parser fixes change numbers you may have relied on.** STDF wafer geometry (the WCR record:
wafer size, die size, orientation) was read two bytes out of place, so any file that carried it
had its geometry misread; and ATDF files were read in STDF's field order rather than ATDF's own.
Both are now read by their specifications. Separately, **a file holding several lots, test
temperatures or programs now loads as it should**: CSV/JSON/Parquet rows used to be grouped by
wafer ID alone, merging every lot's W01 into one map, and a concatenated STDF with several lot
records labelled every wafer with the last lot. Each wafer now carries its own lot.

**STDF and ATDF load together**, since they are the same records in binary and text. The file
filter got easier to read too: columns lead with name, lot, wafer count and when each file was
**Tested**; columns with nothing to show for the files in view are hidden; Parquet files report
their wafer count; and when a scan mixes kinds of file, one click chooses which kind to show.

**tsmap now ships as a static bundle you can host yourself.** Every release carries
`tsmap-<version>-web.zip` alongside the desktop installers: unpack it into any web server's
document root, browse to it, and that is the whole procedure — no installer, no server-side
component, and **no internet access needed at any point**, installation included. Files are still
parsed in the browser, so nothing reaches your server either; it only ever serves the
application.

This is the answer for a site that cannot reach the public internet, or whose policy says lot
data must not touch an external origin. It is also the only *dependable* offline option in a
browser: the hosted app keeps working without a network once loaded, but only because the
browser cached it — a hard refresh, an evicted cache or a different profile all send it looking
for our servers again. A copy on your own machine has no such dependency. Two things catch
people out and are documented: `.wasm` must be served as `application/wasm` (nginx, Apache and
Python get this right; **IIS needs a MIME entry added**), and it must be served over http/https
rather than opened as a file.

**Reload the same test definitions in one click.** If you follow a product or test-program
series, you were re-navigating the file picker for the same definitions file on every dataset.
**Load definitions** now has a **▾** listing the files you recently loaded *or saved*, wired into
the test selector and the `Setup ▾` definitions dialogs. On the desktop the file is re-read from
disk, so you always get its current contents and are told when it has changed. In a browser
there is no path to re-read — a browser picker hands a page contents, never a location — so the
entry is the copy taken when you last used it. That is fine for a selection or a set of renames
and needs care with **spec limits**, so the menu says so, each row dates its copy, and applying
one logs a warning naming the file when it sets limits. Re-pick through *Choose a file…* if the
file may have moved on.

**The file and setup menus are tidier.** Recent files moved into **Open files ▾** beside
*Choose files…* and *Scan a folder…* — recents are a way of answering *which file*, which is the
question that caret exists for — so the separate **Recent** button is gone. The **Lot ▾** menu is
now **Setup ▾**, grouped into *Tests & bins*, *Wafers* and *Analysis*, because no single word
describes six unrelated dialogs. And its two overlapping test entries are one: *Filter tests…*
and *Test definitions…* read and wrote the same file to the same effect, differing only in
whether the data was re-parsed — a consequence of what you changed, not a choice worth putting
to you. **Setup ▾ → Tests…** now covers selecting, renaming, limits, units, type and save/load
together.

**You can see and clear what tsmap remembers.** **Help → Reset saved settings…** lists what is
actually stored on your machine — theme, recent files, saved column mappings, wafer splits,
diameter overrides — explains each one in a line, and forgets only what you tick. Until now a
wrong saved column mapping was reapplied on every load of that layout with no way to undo it
short of browser developer tools. Nothing here ever leaves your machine, and loaded wafer data is
never stored.

Two fixes worth naming. **The last-used directory was never remembered on Windows** — it had
been resolving its state file from a variable Windows does not set for a GUI application, so the
picker silently always opened at the default; it now asks the OS for the right location on all
three platforms. Your settings carry over this update even though the desktop app's internal
identifier changed with the move to the wafertools organisation. And **tsmap's About dialog now
names the engine underneath** (`wafermap 0.28.0`),
reported by the bundle itself rather than a manifest — when a map looks wrong, which version drew
it is the first thing worth knowing.

**wafermap** makes that version publicly readable (`WMAP_VERSION`) so any application embedding
it can do the same, and fixes two things adopters hit: `analyzeWaferMap` raised an
"option corrected" advisory for an option that was never set — the most ordinary thing a host
writes, forwarding an optional value that happens to be undefined — and `buildWaferMap` with an
empty `testDefs` array silently lost every value plot mode and test column. In a single-wafer
view, clicking a box in the test-value distribution now opens the map on that test, as it already
did in a gallery. Its documentation
site also gained a light/dark theme, a table of contents that follows your scroll on the long
API reference, and hover definitions for domain terms like STDF, PTR and Cpk.

*See: [tsmap v0.1.34](https://github.com/wafertools/tsmap/blob/main/CHANGELOG.md#0134--2026-09-11),
[wafermap v0.28.0](https://github.com/wafertools/wafermap/blob/main/CHANGELOG.md#0280--2026-09-11)
— a breaking release for applications built on wafermap: see its changelog for the colour API
changes.*

## 2026-09-09 — Spec limits stay with the file they came from; scan a folder; test-value findings run on their own

**If you load several files at once, this release fixes a correctness bug worth knowing about.**
tsmap merged every file's test definitions into a single list — last one wins — and then plotted
*every* wafer against it. A test number identifies a test within **one test program**, so across
files it is not an identity at all: test 1001 is a threshold voltage in millivolts in one of
tsmap's own sample lots and a leakage current in nanoamps in another. Loading them together
silently discarded all but one definition, then labelled, scaled and capability-scored every lot
against the survivor — box plots of nanoamp readings titled `vth_n_mV` with that test's spec line
drawn across them, a process-capability panel reporting **Ppk −1489**, and drilled-in maps showing
every die out of spec. Nothing warned, because by the time the data reached the map every wafer
carried the same already-wrong list.

Definitions now stay with the file that declared them — **wafermap** gained per-wafer test
definitions to make that possible — and where two files genuinely disagree about a test number you
are told **at scan time, before you choose**, rather than after the load. The imported *data* was
never wrong: tests are selected by number. What was wrong was the name, the units and the limits it
was shown against. If you have ever loaded more than one lot into tsmap at once, this is the release
to move to.

**Test-value findings now run by themselves when they are cheap.** The regional analysis of
parametric values lived behind a menu item, off by default — so a reader looking at the Findings
panel got no hint that a whole category of finding had been skipped, and nothing would ever send
them into the menu to discover it. tsmap now estimates the cost: under about a second it simply
runs the analysis and notes it in the log; over that, the Findings panel itself carries a row
saying what is missing, how much data it would cover and roughly how long it would take, with an
**Analyse** button. The offer and its price arrive together, where you are already looking.

**Point tsmap at a folder instead of picking files.** **Open files** and **Add files** each gained
a ▾ offering *Choose files…* or *Scan a folder…*, and dropping a folder onto the window now scans
it (previously that failed outright). A folder picker asks *where to look*, which the file-triage
table cannot answer — so the two steps stop asking you the same question twice. Subfolders are
opt-in and only offered when there are any.

**wafermap** adds the chart a split experiment is actually run to produce — a **per-test pass-rate
chart broken out by group** — plus a **wafer-to-wafer trend chart**, population tiles on the
Insights overview, split comparison in the lot summary report, and a dedicated Insights example
(the library's largest feature surface previously had no example of its own). Geometry checking
gained its missing half: a supplied wafer diameter that is too *large* for the dies used to pass in
complete silence, and is now reported. The old `inferred-pitch` advisory is gone — it fired on
every build that supplied a diameter without a die pitch, which in tsmap left a permanent red
banner on screen with nothing available to dismiss it.

Smaller things: an **About tsmap** dialog (the app previously stated its version only in the log
and its licence nowhere), **eight more colour themes** taking the picker from 8 to 16,
**interface zoom** with Ctrl/Cmd +/−/0, and resizable columns in the file-filter table. Since the
last entry, both projects also fixed the in-app user guide rendering as plain light — or as pale,
near-illegible text on white — under every dark theme.

*See: [tsmap v0.1.33](https://github.com/wafertools/tsmap/blob/main/CHANGELOG.md#0133--2026-09-09),
[wafermap v0.27.0](https://github.com/wafertools/wafermap/blob/main/CHANGELOG.md#0270--2026-09-09)
· earlier: [tsmap v0.1.32](https://github.com/wafertools/tsmap/blob/main/CHANGELOG.md#0132--2026-08-28),
[wafermap v0.26.1](https://github.com/wafertools/wafermap/blob/main/CHANGELOG.md#0261--2026-08-28)*

## 2026-08-28 — Bin definitions, wafer diameter & edge-exclusion overrides, reports open in-app

**tsmap** can now read, save and load **bin definitions** — human-readable names for hard/soft
bin numbers, plus which ones count as pass. STDF/ATDF already carry this in their own HBR/SBR
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

*Since this entry: the **Lot ▾** menu named above became **Setup ▾** in tsmap 0.1.34, and its
two test entries merged into one **Tests…** — so the paths here read `Setup ▾ → Bin definitions…`
and `Setup ▾ → Diameter & edge exclusion…` on current versions.*

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
