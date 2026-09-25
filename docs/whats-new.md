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

## 2026-09-25 — Spec limits on the charts, files in different units combine, and a slimmer API

**For developers building on wafermap — 0.31.0 is a breaking release:**

- **The exports deprecated in 0.30 are removed** — the low-level drawing pipeline, the chart-data
  and region builders, and a set of internal helpers — along with `buildWaferMap`'s unused second
  argument and the `showPartialDies`/`includePartial` options. Each has a replacement, listed on
  the new [Upgrading](https://wafertools.github.io/wafermap/upgrading/) page; most integrations
  use none of them.
- **Values outside the STDF V4 ranges are now treated as missing** by `buildWaferMap`, as 0.30.4
  announced, with the `input-values-outside-stdf` warning saying what was left out;
  `waferConfig.orientation` accepts 0, 90, 180 or 270.
- **`downloadFilename` is a prefix for every saved file**, CSVs and charts included, rather than
  the name of the map's PNG alone.
- **Each release now has a
  [GitHub Release](https://github.com/wafertools/wafermap/releases)** with its changelog notes.
- The data layer is about 49 KB gzipped, down from about 61 KB.

**If you have hand-written definitions files: `lsl`/`usl` columns are now read as spec limits.**
They used to be read as test limits. The log says so whenever a file uses them. To keep a file's
values as the limits dies are judged by, rename those columns to `lo_limit`/`hi_limit`. Files
saved by tsmap are unaffected.

**Spec limits sit beside test limits on the charts.** The Insights box plot, histogram,
wafer-to-wafer trend and scatter now draw a test's spec limits (LSL/USL) as well as its test
limits (Lo limit / Hi limit), each labelled and dashed differently so the two are never confused.
When a test has both, a **Limits** choice switches all four charts together between both (the
default), either kind, or none. Gridlines are lighter so the limits stand out, and the scatter
marks a limit that falls outside its plotted range at the edge, as the other charts already did.
The wafer map itself still judges pass/fail by the test limits.

**Spec limits can come from your own files.** A test-definitions file sets spec limits with
`lsl`/`usl` (or `lo_spec`/`hi_spec`) columns, beside test limits in `lo_limit`/`hi_limit`, and a
long-format CSV, JSON or Parquet file can map columns to either kind. Process capability is
measured against them, and **Save definitions** writes both. Every column name now means exactly
one kind of limit, and a name that doesn't say which (`min`, `max`, `lower`, `upper`) is ignored
with a warning rather than guessed.

**A test recorded in mV by one file and V by another now stays in the lot.** tsmap used to
treat the two as different measurements and leave the test out of every view that compares
wafers. It now converts the later file's values and limits to the unit the first file uses, and
says so in the log. Only a difference of SI prefix on the same unit is converted; anything else
is still kept apart. A test-definitions file whose limits are written in another prefix is
converted the same way.

**Every format now reads values to the STDF ranges.** CSV, JSON and Parquet join STDF and ATDF:
a bin, position or site number the STDF format cannot hold, or a reading that is not a finite
number, is treated as missing and reported, rather than plotted as if it were ordinary data.

**New warnings are visible without opening the log.** tsmap's **Log** button shows how many
warnings and errors have arrived since you last looked, such as "▲ 3 new warnings".

---

## 2026-09-25 — Spec limits, correct retest matching, and test data read to the STDF spec

**Spec limits and test limits are now shown and used separately.** A test can carry both the
limits it was judged against on the tester (Lo limit / Hi limit) and the process's own
specification limits (LSL/USL) — the two are frequently different, and conflating them has
always been a source of confusion. Process capability (Cpk and friends) now measures against a
test's spec limits when it has them, and against its test limits otherwise, and says which one it
used. Every chart, table and report labels the two apart.

**A result exactly on a limit is judged the way the file says, not assumed.** STDF and ATDF each
record whether a value equal to a test's limit counts as a pass or a fail; that rule is now read
and applied everywhere a die's pass/fail status is shown — the map, the yield figures, the
Summary panel, every chart.

**Retests are matched correctly even when a file gives no die position.** Some testers record
retests by part ID rather than by X/Y, and previously such a lot had no reliable way to say which
record superseded which. Retests are now matched by part ID within a wafer when the file has no
position for them, so retest handling (first/best/worst/last) resolves correctly instead of
treating every attempt as a separate die.

**Test data is read exactly to the STDF/ATDF specification.** Bins, coordinates, test results and
part IDs are all read to the value ranges and validity rules the formats define — including which
recorded results the tester itself marked as usable — so what you see matches the tester's own
judgement of the data. A value outside what the spec allows is reported rather than plotted as if
it were ordinary.

**Derived tests and sweeps can be cleared, and tsmap warns when they won't fit.** The test
selector has a **Remove derived tests** action, and **Setup ▾ → Sweeps…** has **Clear** — both
previously had to be redefined from scratch to remove. After loading a file, tsmap now also warns
when none of your derived tests could be computed, or when a sweep names none of the lot's tests,
with a button to remove just the sweeps that don't apply.

**For developers building on wafermap:**

- **`TestDef.specLow`/`specHigh`** — spec limits, read separately from `limitLow`/`limitHigh`.
  `TestCapability.limitBasis` says which pair a given test's capability was measured against.
- **`TestDef.limitLowInclusive`/`limitHighInclusive`** — whether a value equal to a limit passes,
  read from the file and honoured by every pass/fail surface.
- **`DieResult.supersedes`** — set when the tester marked a record as replacing an earlier one; it
  always wins, whatever `retestPolicy` says.
- **`DieResult.partId`/`Die.partId`** now accept text as well as numbers, as STDF and ATDF define
  them.

---

## 2026-09-23 — Tests computed from other tests, response curves, and charts of just the dies you pick

**You can now define a test as a formula of other tests, and it works everywhere a measured test
does.** A switching window (`t[1202] - t[1212]`), a leakage shift, a ratio, a log of a current,
"did every step of this sweep pass" — each becomes a test you can plot on the map, give spec
limits, and see in every chart, table and report. Anything computed this way is marked **†**
wherever it appears, so it is never mistaken for something the tester measured, and a die missing
one of the inputs shows as no data rather than a misleading zero. In tsmap, add an `expression`
column to your test definitions file.

**Sweeps: read a run of tests as a curve.** Many test programs measure one thing at a series of
conditions — voltages, temperatures, bake times, cycle counts, resistance thresholds — and record
each step as its own test. Declare them as a sweep and Insights draws them as a response curve
(the median with a p10–p90 band), and measures a pair of curves against each other: **where they
cross**, and **how far apart they are** at the levels you choose. The swept value can be given
directly or read straight from the test names (`LRS_STATS_12K` is 12 kΩ), steps that grow by
multiples can go on a log axis, and the axis can carry a unit. In tsmap, load a sweeps file from
**Setup ▾ → Sweeps…**, or pass `--sweeps` on the command line.

**Right-click to chart just the dies you care about.** Select a cluster or a scratch on a map and
right-click — or right-click a wafer in a gallery, or one wafer's bar in a chart — for a
histogram, process capability or any sweep drawn from only that population. The chart says
exactly how many dies it used and from which wafer, so it cannot be mistaken for the whole lot.
The same menu is on the map's new **Chart** button, and on the Menu key.

**An expanded chart now uses its window.** Charts opened in their own window used to stay the
size they were in the grid; each now grows in the way that suits it — plots fill the window, lists
of wafers or bins show every row, and circles and matrices grow to fit.

**tsmap's file dialogs now remember where you were.** Opening data, saving images, exporting,
loading definitions and saving filters each reopen in the folder you last used for that task —
on Linux too, where they used to start at your home folder every time.

**For developers building on wafermap:**

- **`derivedTests`** on `buildWaferMap` — a `TestDef` plus an `expression`. Parsed to a typed
  tree and walked, with no `eval` and no expression engine, so a set is safe to share as JSON.
- **`insights.sweeps`** — sweep definitions with test ranges (`'1200..1230'`), `xValues` or
  `xFromName` (a `{x}` placeholder pattern, deliberately not a regex), `xUnit` and
  `xScale: 'log'`.
- **Drilldown needs no wiring** and has no option: a map takes over right-click only when there
  is something to chart, so a host's own context menu still works on bins-only maps.

---

## 2026-09-21 — Big lots load far faster, and tsmap warns before a crash

**Loading a large gallery is now up to 20× faster, with no single pause long enough to make the
page look frozen.** A gallery's shared legend and Summary panel used to rebuild from scratch
after every wafer arrived, so a progressive load got slower the more wafers it had — 182 seconds
for 50 wafers of 8,000 dies each. The same lot now loads its cards in about 8 seconds, with no
single pause longer than half a second. A lot's Summary panel, which used to lock up the browser
for over 15 seconds on the
largest lots, now fills in piece by piece instead of freezing.

**tsmap warns you before a web-browser load that would crash the tab**, instead of just
crashing. Above roughly 200,000 dies the browser build can run out of memory outright; it now
checks for that specific risk — separately from its existing "this may be slow" warning, which
didn't cover it — and tells you before you commit to the load. The desktop app was never
affected.

**tsmap's loading screen is now one clear, consistent indicator** for every stage of opening a
file — scanning, parsing, analysing, rendering — instead of a small spinner that vanished on
narrower windows and didn't always agree with the rest of the screen about whether something was
still happening. A new "Log phase timings" option in the Help menu can record exactly how long
each stage took, useful if you need to report a slow load.

**A very large wafer or lot could fail outright with an obscure "Maximum call stack size
exceeded" error.** Anything at or above roughly 100,000 dies could hit this in several places —
now fixed everywhere it could occur.

**Correlation charts on very large lots no longer hang.** Above 25,000 dies, the correlation
panel now computes from an evenly-spread sample of the lot rather than all of it, and says so
plainly when it has. The numbers you see are unaffected either way — the sample is large enough
that the values wouldn't change.

**Smaller fixes:**

- A single bad test value no longer turns that whole test's statistics into "NaN" in the Summary
  and Insights panels — it's now correctly left out instead.
- Switching between Insights tabs and back no longer re-runs the whole analysis a second time.

**For developers building on wafermap:**

- **`renderWaferGallery` can now report its own loading progress.** `onItemResolved(resolved,
  total)` fires as each card is built, and `onItemsResolved()` fires once the whole gallery —
  including its shared legend, colours and Summary panel — has actually settled. tsmap's new
  loading indicator (above) is built on this.
- **A sampled correlation matrix says so in its data**, via `CorrelationMatrix.sample`, not just
  in the panel — for hosts reading matrices directly.

---

## 2026-09-17 — A security fix, exports named for their data, choosing a test on a phone

**A security fix: please update.** This release fixes a security issue in how names from a data
file are displayed. It affects earlier versions of both tsmap and wafermap, so update, particularly
if you open files from sources you don't control.

**Saved images and CSVs are named for the data they came from**, for example
`LOT123_W05_hard-bin.png` or `LOT123_25-wafers_yield-by-wafer.png`. Exports used to have fixed
names such as `dies.csv`, so the same export from two wafers saved as `dies.csv` and
`dies (1).csv`, with nothing to say which wafer each held. Anything the data doesn't have is left
out rather than guessed: a wafer with no ID gets no wafer part in the name.

**You can choose a test from the plot-mode menu on a phone or tablet.** With more than six tests,
"Test Value ▶" opens a list, and that list only opened when a mouse hovered over it. A tap opened
it and closed it again straight away. It now opens with a tap, and with Enter or Space from the
keyboard, where the arrow keys move through the tests.

**Ring, quadrant and reticle lines are clear on high-resolution screens.** Map lines and markers
were drawn at a fixed number of screen pixels, so on the high-DPI displays most laptops now have
they came out at half their intended width or less and were hard to see. They now look the same at
every display scale. Nothing changes on a standard display.

**A detached gallery card no longer shows two sets of window buttons.** In the tsmap desktop app a
detached card opens in a window drawn inside the app. Maximized, its header sat just under the
app's title bar, and its minimize button looked like the real one but only shrank the card. That
button is now **Collapse**, with a different icon, and it is hidden while the window is maximized.

**Smaller fixes:**

- **tsmap's Filter files dialog** no longer shows a "selected" count in its title that never
  changed. The count above the table is the live one.
- **A gallery with a fixed number of columns** uses the full width instead of leaving most of each
  row empty.
- **Changing the bin colours in an expanded gallery card** recolours that card.
- **A card put back into the gallery** matches the other cards again instead of keeping the view it
  had while expanded.
- **Outlined buttons in the Summary panel** have a visible border in dark themes.
- **A lot report** analyses its wafers when no summary was prepared beforehand, instead of
  reporting empty wafers.

**For developers building on wafermap:**

- **Five of the exports deprecated in 0.30.0 are staying** after review, because each is the only
  supported way to do something an app needs: `visibleFindings`, `openReportModal`,
  `metadataDisplayValue`, `getReticleCell` and `renderFindingsReportHtml`. They no longer log a
  notice.
- **The rest now have replacements.** Analysis results include process capability (Cp, Cpk, Pp,
  Ppk), pass rates by the tester's recorded verdict, ring and quadrant yield and the spatial
  pattern. There are also new report functions that take built maps, `getBinColors()` on both
  controllers, and `buildWaferMap({ layout: true })` for a die layout with no test data. Every
  remaining deprecation notice names what to use instead.
- **`downloadFilename` becomes a file-name prefix in 0.31.0.** Until then it still names the map
  and gallery PNG, and logs a notice once.
- **Text bins and test values raise a warning.** A CSV parser gives every field as text, and a bin
  of `"1"` is not pass bin `1`: yield read 0% with nothing to say why.

---

## 2026-09-15 — A bin keeps its colour in every lot; your pass bins count everywhere; tsmap works offline

**A bin is now the same colour in every lot.** The 2026-09-11 release coloured bins by how many
dies each held: the biggest failing bin took the first fail colour. That kept colours distinct
within one view, but it meant a colour stood for "the largest failure here" rather than a
particular bin. Bin 7 could be red in one lot and brown in the next, and filtering a gallery could
recolour bins. Wafer-map tools across the industry tie colour to the bin number, so engineers can
learn a program's colours and compare screenshots by eye. wafermap does that again. It still
follows pass and fail: a passing bin never takes a fail colour, a failing bin 1 is never green,
and colours from your bin definitions file still win. Hard bin 5 and soft bin 5 are different
colours, so the two maps can't be mistaken for each other. Screenshots of bin maps will change
colour.

**Pass bins from your file now count everywhere, not just in the yield figure.** tsmap reads each
wafer's pass bins from the file. Until now they reached only the headline yield number, and
everything else assumed bin 1 was the only pass: findings, yield statistics, bin colours, the
Summary panel and report, region yield, the Insights yield charts and the gallery strip. On a
program where bins 1 and 2 both pass, bin 2 showed as a failure beside a yield that counted it as
a pass. Every surface now uses the pass bins in your file. In a lot that mixes test programs, each
wafer is judged by its own pass bins, and a warning names any bin that passes on one wafer but
fails on another.

**Soft-bin tables and yield are right too.** They had been judged against hard pass-bin numbers,
so the gallery's soft-bin yield could read close to 0%. Soft bins are now judged by their own
results.

**Bin legends list passing bins first, then failing bins from most dies to fewest**, the same
order as the Summary panel and the Insights pareto. The map legend and gallery legend used to sort
by bin number, so one lot was listed two different ways on one screen.

**Insights now works for files without die coordinates.** On a file with no X/Y positions, the
"No die position data" panel stayed on top of Insights and hid most of the charts. The same
happened on a file where only some dies have coordinates.

**The tsmap browser app now works offline, and Chrome, Chromium and Edge can install it** as an
app with its own window and launcher entry. It is aimed at platforms tsmap ships no installer for,
such as RHEL. Firefox on the desktop can't install web apps, but the offline caching still works
there. The whole app is cached on first visit, including the file parser, so an offline tsmap can
still open files. The user guide's text works offline; its screenshots load the first time you
view them online. Updates are never applied behind your back: tsmap asks first, because reloading
discards anything you have loaded. None of this changes the desktop app.

**Wafer configuration details are in plain language.** The info row used to show raw STDF codes
such as "Wf Flat: D · Pos X: R". It now reads "Wafer Flat: Bottom · X Increases: Right", with
units on every size.

**Opening a single-wafer file no longer stops to ask for a wafer label** when the file's lot ID
already identifies the wafer.

**For developers building on wafermap: 78 exports are deprecated and will be removed in 0.31.0.**
They are the low-level drawing pipeline (`buildView`, `toCanvas` and related functions), the
chart-data builders the Insights tab now draws for you, and helpers that were exported by
accident. Each still works in 0.30.0 and logs one console notice the first time you call it, saying
what to use instead. If you depend on one with no replacement,
[open an issue](https://github.com/wafertools/wafermap/issues) before 0.31.0. The `passBins`
options on the analysis and render functions are removed: give pass bins to `buildWaferMap` and
everything downstream uses them. The
[changelog](https://github.com/wafertools/wafermap/blob/main/CHANGELOG.md) lists every name.

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
