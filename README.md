# Clipper2 WASM Test Suite

Interactive test harness and visualization for the [Clipper2](https://github.com/AngusJohnson/Clipper2) polygon-clipping library, compiled to WebAssembly via the [Clipper2-WASM](https://github.com/ErikSom/Clipper2-WASM) port.

It began as a sandbox while building the geometry engine for EasyTrace5000 (and now EasyShape5000). It now lives on its own as **living documentation**: a working reference for the Clipper2 WASM call syntax, plus custom arc-reconstruction experiments. It is not under active development (although I might update it to follow Clipper2 WASM library updates).

## About Clipper2

Clipper2 (Angus Johnson) is a 2D library for boolean operations on polygons — intersection, union, difference, XOR — and for polygon offsetting (inflate / deflate). It is robust because it computes on **integer coordinates** internally, so this suite scales floating-point input by a fixed factor before handing it to the library and divides back on the way out.

Key concepts the tests exercise:

- **`Path64` / `Paths64`** — a path is a list of 64-bit integer points; `Paths64` is a list of paths. (`PathD` / `PathsD` are the floating-point equivalents.)
- **Fill rules** — `EvenOdd`, `NonZero`, `Positive`, `Negative` determine which regions count as "filled".
- **Winding** — subjects are expected counter-clockwise (positive area); the suite normalizes this with native `AreaPath64`.
- **`PolyTree`** — a hierarchical result that preserves outer/hole/island nesting, flattened here into a structured `{ outer, holes, islands }` object by nesting parity.
- **Z-coordinate metadata** — points carry an optional `z` value that survives boolean operations. The arc-reconstruction tests use this to tag tessellated arcs and rebuild analytic curves afterward.

The [Clipper2-WASM](https://github.com/ErikSom/Clipper2-WASM) port (Erik Som) compiles the C++ to WebAssembly with Emscripten/embind. One syntax gotcha worth remembering: **WASM objects are not garbage-collected — every `Path64`, `Paths64`, `Clipper64`, etc. must be freed with `.delete()`**. This suite centralizes that in a memory tracker.

## What the suite demonstrates

- Boolean operations (union / intersection / difference / XOR) with draggable shapes, including clipping against an imported SVG outline.
- Polygon offsetting via `InflatePaths64` (join and end types, miter limit).
- Minkowski sum and difference.
- Point-in-polygon testing (`PointInPolygon64`).
- Path simplification and hole detection (the "Letter B" test builds a glyph with correctly-wound holes).
- PolyTree extraction into a nested outer/holes/islands structure.
- Arc reconstruction — carrying arc metadata through boolean ops and rebuilding analytic arcs from the polygonal result.
- SVG export of input, raw, and output geometry.

## Live page

https://ricardojcmarques.github.io/Clipper2-WASM-Example/

## Running locally

The page loads a `.wasm` binary, so it must be served over HTTP(S); opening `index.html` from `file://` will fail. Any static server works:

```
# from the repository root
python -m http.server 8080
# open http://localhost:8080/
```

VS Code Live Server also works. Use the on-page buttons to run each test.

## Requirements

A browser with WebAssembly support. No build step, no external/CDN dependencies — everything is static.

## File structure

Entry point
- `index.html` — page, test cards, script loading, app bootstrap
- `clipper2testpage.css` — styles

Bundled WASM library
- `clipper2z.js` — Emscripten glue
- `clipper2z.wasm` — compiled binary

Application modules
- `clipper2-defaults.js` — configuration and reusable test geometry
- `clipper2-core.js` — module init, state, memory cleanup
- `clipper2-geometry.js` — geometry helpers and SVG-to-path conversion
- `clipper2-operations.js` — boolean ops, offsets, Minkowski, PolyTree
- `clipper2-rendering.js` — canvas rendering
- `clipper2-arc-reconstruction.js` — arc metadata and analytic-curve reconstruction
- `clipper2-svg-exporter.js` — SVG string export
- `clipper2-tests.js` — test definitions and runners
- `clipper2-ui.js` — UI wiring and interaction

## License & credits

Licensed under the Boost Software License 1.0 — see [LICENSE](LICENSE).

- Test page code and UI — © 2025-2026 Eltryus — Ricardo Marques
- Clipper2 — © 2010-2026 Angus Johnson — https://github.com/AngusJohnson/Clipper2
- Clipper2-WASM — Erik Som — https://github.com/ErikSom/Clipper2-WASM