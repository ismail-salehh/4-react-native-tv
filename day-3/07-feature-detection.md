# Feature detection, not device checks (web)

LG and Samsung ship many years of TVs, each with a different Chromium version. A device or version check guesses what the engine can do; feature detection asks.

```ts
// Bad: guessing from the device
if (isTizen && tizenVersion < 5) {
  useScrollFallback();
}

// Good: asking the engine
if ('IntersectionObserver' in window) {
  observeCards();
} else {
  loadAllCardsNow();
}
```

Why device checks go wrong:

- The same model gets firmware updates that change the engine.
- A new model or brand you didn't list gets the wrong branch.
- The user agent string is not always what you expect, and parsing it is fragile.
- The check says nothing about *which* feature is missing, so the next person doesn't know when it can be removed.

Common checks:

```ts
const hasIdle = typeof window.requestIdleCallback === 'function';
const hasResizeObserver = 'ResizeObserver' in window;
const hasAbort = typeof AbortController !== 'undefined';
const supportsGap = typeof CSS !== 'undefined' && CSS.supports?.('gap', '1px');
```

Note `CSS.supports` itself is missing on very old engines, so check it first. And `CSS.supports('gap', ...)` can be true for grid but false for flexbox on some versions (see [flex gap on old engines](../docs-2/01-layout-and-styling.md)); for layout, it's simpler to not depend on the feature at all.

Device checks are still right for things that are **really platform APIs**, not engine features: Samsung AVPlay, LG Luna services, key codes, app lifecycle. Those go behind adapters (below), not in feature code.

### Polyfills for old engines

For JavaScript APIs you use that old engines lack, add polyfills instead of writing fallbacks by hand.

- Write the **lowest engine you support** into `browserslist` (read it from the real TV, see [Old TV browser engines](../docs/01-tv-mindset.md)).
- Use `@babel/preset-env` with `useBuiltIns: 'usage'` and `core-js`, so only the polyfills your code needs are added.
- **Transpile `node_modules` too.** Many libraries ship modern syntax (optional chaining, `??`, class fields). By default `babel-loader` often excludes `node_modules`; an untranspiled library throws a syntax error and the app shows a black screen on old TVs only.
- Some APIs can't be added by `core-js` (e.g. `fetch`, `AbortController`, `ResizeObserver`, `IntersectionObserver`): add a dedicated polyfill or a fallback path, only if your lowest engine needs it.
- Polyfills cost bundle size and startup time. Don't add a blanket polyfill bundle; target the engines you actually support.

```js
// babel.config.js (web build)
module.exports = {
  presets: [
    ['@babel/preset-env', { useBuiltIns: 'usage', corejs: 3 }],
  ],
};

// .browserslistrc
// Chrome >= 53   (example: the oldest engine you support)
```

> **Verify (web):** check the current LG and Samsung docs for the web engine version of each platform version you support, and test the production build on the oldest TV you have. A syntax error from one untranspiled package is the most common "black screen on TV only" bug.

## Keep platform checks from spreading

Without a rule, platform checks spread: one `if (isTizen)` in the player, then in the key handler, then in a screen, then in a card. After a year nobody can say what the app does on LG.

Rules:

1. **Feature code never checks the platform.** Screens, rails, cards, and hooks used by features don't import `Platform`, `isTizen`, or `isWebOS`.
2. **Every platform difference has one home: an adapter.** One interface, one file per platform, chosen by file extension at build time (`Player.tizen.web.tsx`, `Player.web.tsx`, `Player.tsx`). See [Platform files and adapters](../docs-2/03-app-structure.md#platform-files-and-adapters).
3. **Small differences go in a per-platform config module** (`config.tizen.web.ts`), not `if`s: a number, a flag, a key code.
4. **Engine differences use feature detection**, inside the one module that needs the feature, not a global "is old TV" flag.
5. **Enforce it with lint.** An ESLint rule (`no-restricted-imports`) that bans importing the platform module outside `platform/` and adapter folders stops new checks from creeping in.

```
src/
  platform/
    keys.ts  keys.tizen.web.ts  keys.webos.web.ts
    lifecycle.ts  lifecycle.web.ts
    config.ts  config.tizen.web.ts
  player/
    Player.tsx  Player.web.tsx  Player.tizen.web.tsx  types.ts
  screens/      // no platform checks here
  components/   // no platform checks here
```