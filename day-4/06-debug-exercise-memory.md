# Slow screen or memory growth

**Symptom:** focus lags the remote, holding Right keeps moving after you let go, rails stutter while scrolling, or the app gets slower the longer it runs and is eventually killed or reloaded by the TV.

**Reproduce:** on the weakest device with a **release build** (see [Measure on the weakest real device](../docs-3/01-performance.md#measure-on-the-weakest-real-device-with-a-release-build)): hold Right on a long rail for 10 seconds; for memory, browse 5 sections and come back to Home, ten times, and read memory after each round.

Slow on key press:

| Check | Tool | Usual cause → fix |
| --- | --- | --- |
| How many components render per key press | React DevTools Profiler (profiling build), or a render counter log | Focus state too high up → keep it in the item ([Keep focus state local](../docs-3/01-performance.md#keep-focus-state-local-to-the-focused-item)) |
| What runs in `onFocus` | Log or profile `onFocus` | Image loads, fetches, analytics per focus → debounce 200–400 ms |
| JS thread vs UI / main thread busy | Perf monitor on native, Performance tab in the remote inspector | JS busy: too many renders. Main thread busy: big images decoding, heavy animations, layout of huge lists |
| Number of mounted cards | React DevTools component tree | Not virtualised, or window too big → virtualise the vertical list |

Memory growing:

| Check | Tool | Usual cause → fix |
| --- | --- | --- |
| Image sizes | Network tab (served size) vs rendered size | 4K or original artwork in small cards → [right-size images](../docs-3/01-performance.md#image-memory-on-low-ram-tvs) |
| Memory after each browse round | Android Studio memory profiler, Xcode Instruments, remote inspector Memory tab (heap snapshots) | Goes up every round and never down → leak |
| Listeners and timers | Search for `addEventListener`, `setInterval`, subscriptions without cleanup | Missing `return () => remove()` in `useEffect` |
| Kept-mounted screens | Which screens stay in the tree | Every visited screen kept alive with its images → unmount or release images ([Keep mounted vs unmount](../docs-2/03-app-structure.md#keep-mounted-vs-unmount)) |
| Caches | Image cache limits, your own data caches | Unlimited cache → set a limit lower than the phone default |

On web, compare two heap snapshots (one after the first round, one after the tenth): objects whose count grows every round, like detached DOM nodes or listeners, are the leak.
