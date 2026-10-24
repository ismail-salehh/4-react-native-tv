# Wrong Back behaviour

**Symptom:** Back exits the app from a sub-screen, does nothing, skips a level, closes the screen *and* the modal, or works once and then stops.

**Reproduce:** open each layer and press Back once at a time: modal open → Back; search keyboard open → Back; Details → Back; Home with the menu open → Back; Home root → Back. Write down what each press did.

| Cause | Check | Fix |
| --- | --- | --- |
| **Several owners of Back** | Search for `BackHandler.addEventListener`, `keydown` listeners checking 461 / 10009, and `enableTVMenuKey` across the code | One owner (navigation layer) that asks the topmost layer first ([one owner of Back](../docs/02-remote-and-focus.md#repeated-keys-debouncing-long-presses-one-owner-of-back)) |
| **Listener order** | On native, the **last registered** `BackHandler` listener runs first, and returning `true` stops the others | Register overlays after screens; return `true` only when you handled it |
| **Returned `false` by accident** | An `async` handler returns a Promise, which is not `true` | Keep the handler synchronous; return `true`, then start async work |
| **Kept-mounted screen still listening** | Home is kept mounted under Details and still handles Back | Subscribe only while the screen is focused (`useFocusEffect`) |
| **Fires twice** | Handler runs on both key down and key up, or on key repeat | Act on one action only; ignore repeats for Back |
| **Root exit wrong** | Back at root exits on one platform and does nothing on another | Decide the root behaviour per platform (exit, or "press again to exit" dialog) in the Back owner, test it on each |
| **Apple TV Menu** | `enableTVMenuKey()` left on at root, so Menu can never exit | Disable it on the root screen, as tvOS guidelines expect |

Turn the reproduce steps into a test: the Back order is a pure function you can [unit test](./03-testing.md#component-tests-for-focus-and-key-handling-logic).
