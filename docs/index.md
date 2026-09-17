# wafertools

Tools for reviewing semiconductor wafer test data: rendering wafer maps, computing yield
and spatial statistics, and browsing STDF/ATDF/CSV/JSON/Parquet test results.

There are two projects, and which one you want depends on a single question: **do you
want to look at wafer data, or put wafer maps inside something you are building?**

### I want to look at wafer data → **tsmap**

**[tsmap](https://wafertools.github.io/tsmap/)** is a finished, free, MIT-licensed
application. Open STDF, ATDF, CSV, JSON or Parquet (and `.gz`/`.zip`) and get interactive
yield maps, parametric heat maps, findings, a full chart suite, lot galleries and
exportable reports — no code, no account, no upload. It runs on Linux, macOS and Windows,
and also [in your browser](https://wafertools.github.io/tsmap/app/), where files are still
parsed entirely on your own machine.

Most people arriving here want this one.

[Open it in the browser](https://wafertools.github.io/tsmap/app/) ·
[Download](https://github.com/wafertools/tsmap/releases) ·
[Docs](https://wafertools.github.io/tsmap/) ·
[GitHub](https://github.com/wafertools/tsmap)

### I'm building an application → **wafermap**

**[wafermap](https://wafertools.github.io/wafermap/)** (`@wafertools/wafermap` on npm) is
the rendering and analysis library tsmap itself is built on: interactive wafer maps, yield
and spatial statistics, embeddable in your own app. Pure ES modules, no runtime
dependencies, no server.

Reach for it when you need wafer maps *inside* your own application, a data source tsmap
doesn't read, or behaviour it doesn't offer — and note you can start with tsmap and
integrate later. Two calls render a working map; tsmap itself uses fifteen of the library's
hundred-odd exports.

[Docs](https://wafertools.github.io/wafermap/) ·
[Quick Start](https://wafertools.github.io/wafermap/quickstart/) ·
[npm](https://www.npmjs.com/package/@wafertools/wafermap) ·
[GitHub](https://github.com/wafertools/wafermap)

### What changed recently

[**What's New**](whats-new.md) is a curated, plain-language timeline of new capabilities
across both projects.

## Why these exist

Both are written by **Paul Robins**, after nearly four decades in semiconductor test —
most of it building test data analysis tools: wafer map and chart viewers, and the
platforms they were built on.

That work leaned on free software throughout — zlib, Tcl/Tk, SQLite, GCC — and it was a
good deal. These projects are some of it going back the other way, prompted by finding
out that engineers doing this job today still have no decent free option for wafer map
analysis.

Both are MIT licensed: free to use, modify and redistribute, including in commercial and
closed-source products.

## Questions

[GitHub Discussions](https://github.com/wafertools/.github/discussions)
is open to anyone using either project.
