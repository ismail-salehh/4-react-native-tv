# Remote input not detected

**Symptom:** a key on the remote does nothing. Not a wrong action: no handler runs at all.

**Reproduce:** add a log at the very top of the key handler (`useTVEventHandler`, `BackHandler`, or the `keydown` listener) and press the key. If the log never prints, it's an input bug. If it prints, the key arrives and the bug is further along (focus or logic).

| Cause | Platform | Check | Fix |
| --- | --- | --- | --- |
| **Key not registered** | Samsung | Media and colour keys don't reach the app unless registered | `tizen.tvinputdevice.registerKey('MediaPlayPause')` at startup, per key |
| **Back goes to browser history** | LG | Back closes the app or navigates history instead of sending key 461 | Set `"disableBackHistoryAPI": true` in `appinfo.json` and handle 461 yourself |
| **Wrong key code** | Web | Log `e.key`, `e.code`, `e.keyCode`. Old engines often give only `keyCode` | Map by `keyCode` with a per-platform key table (Back: 461 LG, 10009 Samsung) |
| **Listener in the wrong place** | Web | Listener on an element that doesn't have DOM focus, or another listener calls `stopPropagation` | Listen on `window` / `document`, check what the spatial navigation library swallows |
| **Menu / Back not sent to JS** | Apple TV | The Menu button exits the app instead of calling your handler | `TVEventControl.enableTVMenuKey()` while you want to handle it; disable it on the root screen so Menu can exit |
| **Nothing focusable on screen** | Native | On Android TV, D-pad events go through the focused view; with nothing focused, events may not reach your handler | Make sure every screen has a focusable element with initial focus, including loading and error states |
| **Handler on an unmounted / kept-mounted screen** | All | The listener belongs to a screen that isn't the visible one | Register key handlers only for the active screen (check `isFocused` from navigation) |
| **OS-reserved key** | All | Home, volume, input, some settings keys never reach apps | Don't build features on them; see the [key matrix](../docs/02-remote-and-focus.md#key-matrix-per-platform-and-per-brand) |

Native checks:

```sh
# Android TV: send keys without the remote, see if the app reacts
adb shell input keyevent KEYCODE_DPAD_RIGHT
adb shell input keyevent KEYCODE_BACK
# Watch what the system receives
adb logcat | grep -i keyevent
```

Web check, pasted into the remote inspector console on the TV:

```js
window.addEventListener('keydown', e =>
  console.log('key', e.key, e.code, e.keyCode), true); // capture phase: runs before the app's listeners
```

If this logs and your handler doesn't, something in your app stops the event. If this doesn't log, the platform never sent it: registration, `appinfo.json`, or an OS-reserved key.
