# Works in Chrome but breaks on the TV (old engine, web)

**Symptom:** the web build is fine in desktop Chrome, but on an LG or Samsung TV it shows a black screen, a missing section, wrong spacing, or keys that don't work.

**Reproduce:** the production build on the **oldest** TV you support. Open the remote inspector (`ares-inspect` on LG, the Tizen web inspector on Samsung) **before** launching, so you catch startup errors.

| Symptom | Usual cause | Check | Fix |
| --- | --- | --- | --- |
| Black screen on start | Syntax the engine can't parse (optional chaining `?.` and `??` need Chromium 80), often from `node_modules` | Console: `SyntaxError: Unexpected token` with a file name | Set `browserslist` to the oldest engine; transpile the package in `node_modules` ([Polyfills](../docs-3/02-compatibility-and-errors.md#polyfills-for-old-engines)) |
| Crash on one screen | Missing API (`Array.prototype.flat`, `Object.fromEntries`, `structuredClone`, `ResizeObserver`) | Console: `X is not a function` / `X is not defined` | `core-js` with `useBuiltIns: 'usage'`, or a dedicated polyfill / feature detection |
| Cards touching, layout off | CSS not supported (flex `gap` needs Chromium 84, `aspect-ratio` 88, `inset` 87) | Inspect the element: the property is crossed out or missing | Use margins, explicit width/height, `top/left/right/bottom` |
| Keys don't work | Engine sends `keyCode` only, or different codes | Log `e.key` / `e.keyCode` on the TV | Map by `keyCode` per platform |
| Animation never ends, element stays invisible | Unsupported CSS or Web Animations feature | Check the element's final style in the inspector | Simple `transform` / `opacity` transitions, with a fallback that shows the content ([animations](../docs-2/01-layout-and-styling.md#check-nothing-stays-invisible-on-old-engines-web)) |
| Works on the newest TV, slow on old | Engine is far slower than desktop Chrome | Performance tab on the TV | See [Performance](../docs-3/01-performance.md); test with Chrome CPU throttling during development |

**Compare** to narrow it down: read the TV's Chromium version from its user agent, then run the same build in an old Chrome of that version (or check the feature on caniuse.com for that version). If it breaks there too, it's the engine, not the TV.

> **Verify (web):** check the current LG and Samsung docs for the Chromium version of each webOS / Tizen version you support, and how to open the remote inspector on your TV models. Version numbers for CSS and JS features are from caniuse.com / MDN; check them for the exact features you use.