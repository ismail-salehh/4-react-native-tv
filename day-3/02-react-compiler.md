# React Compiler

The React Compiler is a build-time tool (a Babel plugin) that adds memoisation for you. It is stable and optional.

### What it memoises automatically

- **Components**: a child re-renders only when the props it uses change, as if wrapped in `React.memo`.
- **JSX**: unchanged parts of the output are reused when unrelated state changes.
- **Callbacks**: functions created in a component, including inline arrows like `onPress={() => onSelect(item.id)}`, keep the same identity between renders, as with `useCallback`.
- **Values**: calculations inside components and hooks are cached, as with `useMemo`.

So most hand-written `useMemo`, `useCallback`, and `React.memo` are no longer needed in new code.

### Gotchas

- **It does not fix top-level focus state.** When a context value changes, every component that reads it still re-renders. The compiler skips work when inputs are *unchanged*; a new `focusedId` is a changed input. Keep focus local (above).
- **It relies on the Rules of React.** Mutating props or state, reading `ref.current` during render, or calling hooks conditionally can make it skip that component. Run the ESLint rules for the compiler (included in `eslint-plugin-react-hooks` in recent versions, `eslint-config-expo` on SDK 55+) to find these.
- **Skipped components fail silently.** By default the compiler leaves code it can't handle as it was. The app still works but that component isn't optimised. Check in React DevTools: compiled components show a "Memo ✨" badge.
- **Only your code is compiled.** `node_modules` and class components are not optimised.
- **Keep `useMemo` / `useCallback` where you need a guarantee**, e.g. a value used as an effect dependency, so the effect does not fire again. Don't remove existing manual memoisation in bulk without testing; it can change the compiled output.
- **Opt out** one component or file with the `'use no memo'` directive while you fix a problem.
- **Older React needs a runtime.** React 19 needs nothing extra. For React 17/18 set `target: '18'` (or `'17'`) and install `react-compiler-runtime` as a normal dependency.
- **Plugin order.** The compiler must run first in the Babel plugin list.
- **Both builds need it.** Metro (Android TV, Apple TV) and webpack (LG, Samsung) each use their own Babel config. Enable it in both, or the web build stays uncompiled.

```js
// babel.config.js (bare React Native / webpack babel-loader)
module.exports = {
  plugins: [
    'babel-plugin-react-compiler', // must be first
    // ...other plugins
  ],
};
```

On Expo SDK 54 and later Babel is set up for you; turn it on with `"experiments": { "reactCompiler": true }` in `app.json`.

> **Verify:** check the current React Compiler docs (react.dev/learn/react-compiler) and your Expo SDK guide for setup, the lint rule package, and the `target` option for your React version. Then profile a rail screen before and after on a real TV: the compiler should cut re-renders, but it does not make a slow render fast.