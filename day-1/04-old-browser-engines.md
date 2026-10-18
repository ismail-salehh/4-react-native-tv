# Old browser engines on LG and Samsung TVs

LG (webOS) and Samsung (Tizen) apps are web apps running in the TV's built-in browser engine. That engine is **fixed for the TV's lifetime**, so a 2018 TV still runs a 2018 Chromium.

Approximate engine per platform version (check the vendor docs for your exact targets):

| Platform | Model year | Engine |
| --- | --- | --- |
| webOS 4.x | 2018 | Chromium 53 |
| webOS 5.x | 2020 | Chromium 68 |
| webOS 6.x | 2021 | Chromium 79 |
| webOS 22 | 2022 | Chromium 87 |
| webOS 23 | 2023 | Chromium 94 |
| webOS 24 | 2024 | Chromium 108 |
| Tizen 4.0 | 2018 | Chromium 56 |
| Tizen 5.0 | 2019 | Chromium 63 |
| Tizen 5.5 | 2020 | Chromium 69 |
| Tizen 6.0 | 2021 | Chromium 76 |
| Tizen 6.5 | 2022 | Chromium 85 |
| Tizen 7.0 | 2023 | Chromium 94 |
| Tizen 8.0 | 2024 | Chromium 108 |

> **Verify (web):** for each target you support, confirm the engine yourself. Log `navigator.userAgent` on the real TV (or open the remote inspector) and read the `Chrome/xx` part. Write the lowest version into the project's `browserslist`.

What breaks on old engines:

- **Syntax:** optional chaining `?.` and nullish `??` (need Chromium 80), class fields, top-level `await`. One unsupported token means a **blank screen**: the whole bundle fails to parse.
- **APIs:** `Array.prototype.flat`, `Object.fromEntries`, `ResizeObserver`, `IntersectionObserver`, `fetch` details, `Promise.allSettled`.
- **CSS:** `gap` on flexbox (Chromium 84), `aspect-ratio` (88), `inset`, CSS variables in some places.

How to handle it:

- Set `browserslist` to your oldest real engine (e.g. `chrome 53`) so Babel transpiles down to it.
- Make sure Babel also processes the `node_modules` packages that ship modern syntax.
- Add polyfills (`core-js`) for the APIs you use.
- Avoid flex `gap`; use margins.
