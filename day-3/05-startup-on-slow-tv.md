# Startup on slow TVs

The user presses your app's icon and waits. On a slow TV with an old engine, parsing and running JavaScript is the main cost.

**Bundle size**

- **Web (LG, Samsung): every KB counts.** Old TV engines parse and compile JS slowly. Build with `mode: 'production'`, check the bundle with `webpack-bundle-analyzer`, and remove large libraries you only use a little (date libraries, lodash as a whole, icon sets).
- **Native**: Hermes (the default engine) ships precompiled bytecode, so parse cost is lower, but a big bundle still costs time and memory. Keep `inlineRequires` on in the Metro config so modules run only when first used.

**Lazy loading**

Load the first screen's code first. Load the rest when it's needed:

```tsx
const Player = lazy(() => import('./screens/Player'));
const Search = lazy(() => import('./screens/Search'));
const Settings = lazy(() => import('./screens/Settings'));
```

On web, webpack splits each `import()` into its own file. Good candidates: the player, search, settings, login. On native, `lazy` defers *running* the code; with `inlineRequires` the effect is similar.

**Deferred work**

Things that don't need to happen before the first screen is usable should wait until after it:

- Analytics and crash reporting setup (but keep crash reporting early enough to catch startup crashes).
- Prefetching data for rails below the fold and for other sections.
- Warming caches, checking for updates, loading fonts for screens not yet shown.

```ts
// after the first screen has rendered and has focus
requestIdleCallback(() => {
  initAnalytics();
  prefetchOtherSections();
});
```

`requestIdleCallback` is available in React Native and in most TV engines; fall back to `setTimeout(fn, 0)` where it isn't (see feature detection in [Compatibility and errors](./02-compatibility-and-errors.md)).

**Show something real early**: the app shell (menu + skeleton rails) with focus on a safe element, then fill in the content. A blank screen for 6 seconds makes users think the app is broken.
