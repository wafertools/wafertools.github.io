---
title: How to draw a wafer map from an STDF file
description: Draw a wafer map from STDF test data in your browser with no install, and what to do when you need a script or your own application instead.
---

# How to draw a wafer map from an STDF file

The quickest way is to open the file in **tsmap**, a free, open-source wafer map viewer.
It reads STDF directly, so there is nothing to convert, and it runs in your browser with the
file read on your own machine and never uploaded. The same steps work for ATDF (the text
form of STDF), and tsmap also opens CSV, JSON and Parquet files, which take one extra step
to say which column is which.

## In four steps

1. Open [tsmap in your browser](https://wafertools.github.io/tsmap/app/). There is nothing
   to install. A [desktop version](https://github.com/wafertools/tsmap/releases) is also
   available for Linux, macOS and Windows.
2. Drag your `.stdf` file onto the window, or choose **Open files**. A `.stdf.gz`,
   `.stdf.bz2` or `.zip` of STDF files opens the same way.
3. If the file holds many tests, tsmap shows a test selector. Choose the tests you want to
   keep, or accept the defaults.
4. Read the map. Each die is one square, coloured by its hard bin. A lot with several wafers
   opens as a gallery of maps.

No STDF file to hand? Try
[a sample lot with an edge-ring defect](https://wafertools.github.io/tsmap/app/?dataUrl=https%3A%2F%2Fwafertools.github.io%2Ftsmap%2Fsamples%2Ftsmap-edge-ring-lot.stdf&dataFormat=stdf).
The sample files are synthetic and contain no real device data.

## What you get

- A **bin map** for each wafer, coloured by hard bin or soft bin, with the yield for each
  wafer and for the lot.
- A **parametric map** coloured by any test's value, with the test limits marked.
- **Findings**: edge rings, clusters and other patterns the analysis notices, listed so you
  can check them on the map.
- **Insights** charts, data tables and reports, and a PNG download from the map toolbar.

The [Learn tsmap](https://wafertools.github.io/tsmap/learn/) lessons walk through all of
this on a sample lot, and the [glossary](https://wafertools.github.io/tsmap/learn/glossary/)
explains the test-data terms.

## What a wafer map from STDF is made of

A wafer map needs one position and one result per die. In an STDF file those come from the
part result records (PRR): the die's X and Y coordinates, its hard bin and its soft bin.
The wafer information records (WIR and WRR) group the dies into wafers, and the master
information record (MIR) carries the lot. Parametric values come from the parametric test
records (PTR), which tsmap reads alongside the bins.

## Before you trust the picture

- **Which bins count as a pass.** The default is bin 1, but your test program may pass in
  another bin, or several. The yield figure follows the pass bins you set.
- **Retests.** When a die is probed more than once, one result stands for it. The last is
  used by default, and you can choose the first, best or worst.
- **Dies with no position.** Some STDF files carry no die coordinates. tsmap keeps those dies as
  dies without a position, rather than dropping them or guessing a position, and the
  [user guide](https://wafertools.github.io/tsmap/user-guide/#dies-with-no-reported-position)
  explains how they are shown.
- **Supported records.** tsmap reads STDF version 4. Multiple-result parametric records
  (MPR) are not read yet, so tests stored that way do not appear, and the log reports how
  many such records the file had.

## Other ways to do it

- **A script.** Parse the PRR records with a Python STDF library and plot X and Y coloured
  by bin. [stdf2map](https://github.com/CozumelDiver/stdf2map) is a Python command-line tool
  that draws bin wafer maps from STDF files. [stdf-reader](https://pypi.org/project/stdf-reader/)
  is a Python tool that converts STDF to CSV and includes a wafer map view. Choose these
  when the map has to come out of a scripted pipeline.
- **Inside your own application.** Use the [wafermap](https://wafertools.github.io/wafermap/)
  JavaScript library, which tsmap itself is built on. See
  [wafer maps in JavaScript](wafer-maps-in-javascript.md).
- **Other formats.** For a CSV, JSON or Parquet file, tsmap asks you to map the columns
  (X, Y, bin, test values) before it reads the data, and detects common column names
  automatically. See
  [Loading your data](https://wafertools.github.io/tsmap/user-guide/#loading-your-data) in
  the user guide.

## Questions

[GitHub Discussions](https://github.com/wafertools/.github/discussions) is open to anyone
using either project, and a sample file helps if something does not open as you expect.
