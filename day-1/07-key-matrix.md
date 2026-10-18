# Key matrix per platform and per brand

Key codes are different on every platform, and some keys never reach your app because the OS keeps them.

| Key | Android TV (`KeyEvent`) | tvOS (`useTVEventHandler` type) | LG webOS (`keyCode`) | Samsung Tizen (`keyCode`) |
| --- | --- | --- | --- | --- |
| Up / Down / Left / Right | 19 / 20 / 21 / 22 | `up` / `down` / `left` / `right` | 38 / 40 / 37 / 39 | 38 / 40 / 37 / 39 |
| OK / Select | 23 (`DPAD_CENTER`), 66 (`ENTER`) | `select` | 13 | 13 |
| Back | 4 (`BACK`) | `menu` (Menu / Back button) | **461** | **10009** |
| Play / Pause toggle | 85 | `playPause` | — (separate keys) | 10252 |
| Play | 126 | — | 415 | 415 |
| Pause | 127 | — | 19 | 19 |
| Stop | 86 | — | 413 | 413 |
| Rewind / Fast forward | 89 / 90 | `rewind` / `fastForward` | 412 / 417 | 412 / 417 |
| Red / Green / Yellow / Blue | 183–186 | — | 403–406 | 403–406 |
| Home | OS-reserved | OS-reserved (TV button) | OS-reserved | OS-reserved |
| Exit | — | — | — | 10182 (OS-reserved) |
| Volume / Mute | OS-reserved | OS-reserved | OS-reserved | OS-reserved |

Things to know:

- **OS-reserved keys** (Home, volume, power, input, Samsung Exit) never reach the app. Don't design features around them.
- **Samsung needs registration.** Apart from arrows, Enter and Back, keys (media, color, numbers) must be registered or the app never sees them:
  ```js
  tizen.tvinputdevice.registerKey('MediaPlayPause');
  tizen.tvinputdevice.registerKey('ColorF0Red');
  ```
- **Remotes differ within a brand.** Many Android TV remotes have no color keys; Fire TV has no number keys; the Siri Remote has no dedicated Back button on older models (Menu acts as Back).
- **Keep one key map in code** (e.g. `keys.ts`) that translates raw codes into app actions (`'UP' | 'DOWN' | 'SELECT' | 'BACK' | 'PLAY_PAUSE' ...`). The rest of the app only uses the actions.