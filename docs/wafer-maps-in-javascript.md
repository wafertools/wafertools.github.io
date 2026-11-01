---
title: How to draw a wafer map in JavaScript
description: Render an interactive wafer map from die test data in a browser or a JavaScript application, with working code and no runtime dependencies.
---

# How to draw a wafer map in JavaScript

Use **wafermap**, a free, open-source JavaScript library (`@wafertools/wafermap` on npm). It
turns die positions and bins into an interactive wafer map on a canvas, with a toolbar,
tooltips and zoom. It has no runtime dependencies and needs no server.

If you only want to *look at* a wafer file rather than build something, you do not need to
write code: [open it in tsmap](draw-wafer-map-from-stdf.md), which is built on this library.

## A working map in about twenty lines

Save this as an HTML file and open it in a browser. There is no build step.

```html
<!doctype html>
<html lang="en">
<head><meta charset="utf-8"><title>My first wafer map</title></head>
<body>
  <div id="map" style="width:600px; height:600px;"></div>

  <script type="module">
    import { buildWaferMap }  from 'https://esm.sh/@wafertools/wafermap';
    import { renderWaferMap } from 'https://esm.sh/@wafertools/wafermap/render';

    // x and y are integer die grid positions from the prober, not millimetres.
    // hbin is the hard bin: the pass or fail category from the test equipment.
    const results = [];
    for (let x = -14; x <= 14; x++) {
      for (let y = -14; y <= 14; y++) {
        const r = Math.sqrt(x * x + y * y);
        if (r > 14.3) continue;
        const h = ((Math.imul(x + 100, 2654435761) ^ Math.imul(y + 100, 2246822519)) >>> 0);
        results.push({ x, y, hbin: r > 11 && (h % 100) < 55 ? 2 : 1 });
      }
    }

    // buildWaferMap makes the wafer model. It is pure: no DOM access.
    const result = buildWaferMap({ results, passBins: [1] });

    // renderWaferMap mounts an interactive canvas into the container.
    renderWaferMap(document.getElementById('map'), result);
  </script>
</body>
</html>
```

The map colours each die by bin, with the passing bin in green and the failing bin in red.
The toolbar switches plot mode and colour scheme, rotates or flips the wafer, zooms, and
downloads a PNG.

In your own project, install it from npm:

```bash
npm install @wafertools/wafermap
```

## Using your own data

Each die is an object with `x`, `y` and `hbin`, plus optional soft bin, test values and
wafer details. A few points save a lot of time:

- **Pass an array of numbers, not strings.** A CSV parser returns every field as text.
  Convert coordinates, bins and test values with `Number()` before calling
  `buildWaferMap`, or the dies will not count.
- **Set `passBins` for your test program.** Which bins pass decides the yield figure, so do
  not rely on the default of bin 1 if your program differs.
- **Give the wafer a diameter for partial data.** With only die positions the library infers
  the wafer outline from their extent, which is wrong for a half wafer or one quadrant.
- **Give the container a fixed size.** The canvas fills whatever size the container has.

The [quick start](https://wafertools.github.io/wafermap/quickstart/) and the
[troubleshooting page](https://wafertools.github.io/wafermap/troubleshooting/) cover these
in more detail.

## What else it does

- **Galleries** of many wafers, with `renderWaferGallery`, for a whole lot.
- **Statistics and findings**, with `analyzeWaferMap`: yield, spatial patterns such as edge
  rings and clusters, and test-value outliers, shown in a Summary panel.
- **Insights charts** (yield by wafer, bin pareto, distributions, correlation) built in.
- **Frameworks**: guides for [React](https://wafertools.github.io/wafermap/react/),
  [Vue](https://wafertools.github.io/wafermap/vue/) and
  [SvelteKit](https://wafertools.github.io/wafermap/sveltekit/), a
  [web worker](https://wafertools.github.io/wafermap/api/worker/) for large lots, and
  TypeScript types included.

A general charting library can draw dies as a scatter plot. wafermap adds the wafer-specific
parts: the wafer outline and edge exclusion, bins and yield, reticle overlays, retests and
the analysis above, so you do not have to build those yourself.

## Reading STDF files

wafermap draws data you give it; it does not parse tester files. To feed it STDF or ATDF,
use the parser tsmap uses, [`@wafertools/testdata-parser`](https://wafertools.github.io/tsmap/parser-api/),
or map the records to die rows yourself.

## Where to go next

- [Quick start](https://wafertools.github.io/wafermap/quickstart/) and
  [Developer Guide](https://wafertools.github.io/wafermap/guide/)
- [Runnable examples](https://wafertools.github.io/wafermap/examples/), also downloadable
  to run offline
- [API reference](https://wafertools.github.io/wafermap/api/)
- [npm](https://www.npmjs.com/package/@wafertools/wafermap) and
  [GitHub](https://github.com/wafertools/wafermap)

Questions are welcome on
[GitHub Discussions](https://github.com/wafertools/.github/discussions).
