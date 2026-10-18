# Testing the web build

### 1. In desktop Chrome

Run the webpack dev server and use the keyboard as the remote:

| Remote | Keyboard |
| --- | --- |
| D-pad | Arrow keys |
| OK | Enter |
| Back | Backspace (map it to the same action as 461 / 10009 in dev) |

Check:

- Every screen is fully usable with **only** arrows, Enter and Backspace. Don't touch the mouse.
- Focus is always visible and never lost after data loads.
- Back closes overlays before leaving the screen.
- Turn on Norigin `visualDebug` to see focusable areas.
- Use Chrome DevTools CPU throttling (4×–6×) to approximate TV speed.

### 2. On the real TV (web)

Desktop Chrome is much newer and faster than the TV's engine (see [Old browser engines](./01-tv-mindset.md#old-browser-engines-on-lg-and-samsung-tvs)), so always repeat the test on hardware:

- **LG:** install the `.ipk` with webOS CLI (`ares-install`) and debug with `ares-inspect`. Test with both the D-pad and the Magic Remote pointer.
- **Samsung:** install the `.wgt` with Tizen Studio / `sdb` and use the remote web inspector. Confirm registered keys (media, colors) actually arrive.
- Check for a blank screen on start (syntax not supported by the engine), slow focus on long rails, and the real Back key code.

### Explain it out loud

**How does focus differ between the native build and the web build?**

On **native** (Android TV, Apple TV), the operating system owns focus. It has a built-in focus engine that finds the next element in the pressed direction, draws focus on native views, and sends Back through `BackHandler`. React Native exposes it through props: `hasTVPreferredFocus`, `nextFocus*`, `TVFocusGuideView`, `onFocus`/`onBlur`.

On **web** (LG, Samsung), the browser has no directional focus. A JavaScript library (Norigin in tv-app) owns focus: it listens to `keydown`, keeps its own map of focusable components and their positions, decides where focus goes, and tells the component it is `focused`. The RN TV focus props are ignored by react-native-web, so the same intent is expressed with `useFocusable`, `setFocus`, `saveLastFocusedChild`, and `isFocusBoundary`. Back is a raw key code (461 on LG, 10009 on Samsung) that we handle ourselves. And on LG there is a second input, the Magic Remote pointer, that native platforms don't have, so hover and focus have to be kept in sync.

The goals are the same on both (clear initial focus, no dead ends, remember position, trap in modals, one owner of Back), but who enforces them is different: the OS on native, our code on web.

---

Previous: [Remote and focus](./02-remote-and-focus.md)
