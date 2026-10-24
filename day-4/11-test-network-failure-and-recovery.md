# Test network failure and recovery

TVs sit on home Wi-Fi that drops, routers that restart, and slow connections shared with other devices. The app must never end up on an endless spinner or a screen with no focus.

### What to test

| Case | How to cause it | Expected |
| --- | --- | --- |
| **Start offline** | Network off, cold start | Offline state with a focused **Retry**; no crash, no blank screen |
| **Drop while browsing** | Turn Wi-Fi off on the router or device while on Home | Content already shown stays; new requests show an error in place; focus stays where it was |
| **Come back online** | Turn the network back on | Retry works; optionally refresh automatically, **without moving focus** |
| **Slow network** | Throttle to slow 3G | Skeletons appear, focus is usable on what has loaded, requests time out instead of hanging |
| **Server errors** | Proxy or mock returns 500 / 503 for one endpoint | Only that rail or screen shows an error; the rest works |
| **Bad data** | Mock returns an empty list or a missing field | Empty state or hidden rail; the [error boundary](../docs-3/02-compatibility-and-errors.md#error-boundaries-per-rail-and-screen) catches render errors |
| **Session expired** | Return 401 | Sent to login (device code / QR), not stuck on an error |
| **Drop during playback** | Network off while video plays | Player shows an error with Retry / Back; Back still works |
| **Resume after a long time** | Suspend the app overnight (or fake it) | Data refreshes, token refreshes, focus still on a valid element |

### How to cause it

- **Android TV**: `adb shell svc wifi disable` / `enable` (or airplane mode on the emulator); emulator network speed settings for slow networks.
- **Apple TV**: the Network Link Conditioner (in Xcode's Additional Tools) on the Mac running the simulator, or on the device via Developer settings.
- **Web**: Chrome DevTools Network tab: **Offline** and throttling presets; on the TV, unplug the router or block the API host.
- **Any platform**: a debugging proxy (Charles, Proxyman, mitmproxy) to return 500, 401, slow responses or bad data for one endpoint, or a mock server in a test build.

### What recovery needs in the code

- **Timeouts on every request.** A request on a dead connection can hang for a long time. Abort after a fixed time (`AbortController` + `setTimeout`, with a polyfill or fallback where the engine lacks it) and show the error state.
- **Errors per rail and per screen**, using the shared [loading / error / offline states](../docs-2/03-app-structure.md#one-set-of-states), each with a focusable Retry.
- **Focus on the Retry button** when an error replaces the content the user was focused on; otherwise focus is lost.
- **Keep shown content** when a refresh fails. Show a small message instead of replacing a full screen of content with an error.
- **Retry with backoff** for automatic retries (e.g. 1 s, 2 s, 4 s, then stop and show Retry), so a down server isn't hammered by every TV at once.
- **Listen for reconnection** (`@react-native-community/netinfo` on native, `online` / `offline` events on web) to refetch, but don't move focus or reset scroll when the data arrives.
