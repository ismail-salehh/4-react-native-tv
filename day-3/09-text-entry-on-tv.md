# Text entry on TV

Typing with a D-pad is slow: every letter is several presses on an on-screen keyboard. The best text entry on TV is the one the user doesn't have to do. This file covers the on-screen keyboard, moving typing to the phone with a QR code, and voice.

## Order of preference

1. **Don't ask for text.** Pick from a list, use recent searches, suggest as the user types.
2. **Type on the phone.** QR code or short code (device code login, phone-as-keyboard).
3. **On-screen keyboard** with live results, so the user stops after a few letters.
4. **Voice** as an extra, where the platform supports it.

## On-screen keyboard

### System keyboard

Each platform opens its own keyboard when a text field is focused and activated:

| Platform | What happens |
| --- | --- |
| Android TV | `TextInput` opens the system keyboard (IME). Its layout depends on the device and which keyboard is installed. |
| Apple TV | `TextInput` opens the full-screen tvOS keyboard; your screen is hidden behind it while typing. |
| LG webOS (web) | The system virtual keyboard appears when an `<input>` gets DOM focus. |
| Samsung Tizen (web) | The system IME appears when an `<input>` gets DOM focus; it has its own Done / Cancel keys. |

Things to get right:

- **Focus vs editing.** Moving focus onto the field should *not* open the keyboard; pressing OK should. Otherwise the user can't move past the field without the keyboard popping up.
- **On web with spatial navigation**, the library tracks focus, but the system keyboard opens only when the real DOM `<input>` is focused. Call `inputRef.current.focus()` on OK, and `blur()` when done.
- **Back closes the keyboard first**, then leaves the screen (see who owns Back in [Remote and focus](../docs/02-remote-and-focus.md)).
- **Arrow keys while typing belong to the keyboard**, not your spatial navigation. Pause your navigation while the input is in edit mode, or arrows will move focus out of the field.
- **After Done**, put focus on the next useful thing (the first result, the Submit button), not back at the top.

> **Verify (web):** check the current LG and Samsung docs for the virtual keyboard: the events for keyboard shown / hidden, the key codes for Done and Cancel on Samsung's IME, and how each handles `type="password"` and `type="email"`. Test on a real TV: the emulator keyboard behaves differently.

### In-app keyboard for search

Many TV apps draw their own keyboard for search: a grid of letters and numbers, plus Space, Delete, and Clear, next to the results.

- It's **the same on every platform**, so focus and layout behave predictably.
- **Results update live** after 2–3 characters (debounced ~300 ms), so the user can move Right into the results without finishing the word.
- Lay it out for the D-pad: **A–Z in a grid** (e.g. 6 or 7 columns), not QWERTY rows, which mean more presses for most letters. Put Delete and Space where they're one press away.
- Support **long press / repeat** on Delete, and on web, also accept physical keyboard input (some users plug one in, and it helps testing).
- Respect the app language: an RTL language needs its own letters and right-to-left order (see [Accessibility and RTL](../docs-2/02-accessibility-and-rtl.md)).

Use the system keyboard for free-form fields (e.g. a profile name) and the in-app one for search.

## QR input: type on the phone

The user scans a QR code with their phone, gets a web page with a normal text field, types there, and the text appears on the TV.

```
TV                              Server                     Phone
 | create session  ------------> |                           |
 | <-- sessionId, short URL      |                           |
 | show QR (URL + sessionId)     |                           |
 |                               | <-- user opens URL, types |
 | <-- text (socket or polling)  |                           |
 | fill the field, move focus    |                           |
```

- Use it for long or awkward input: **search, email, Wi-Fi-like codes, feedback**.
- For **login, use the device code flow** (see [Login on TV](../docs-2/03-app-structure.md#login-on-tv-qr-code-or-code-entered-on-a-phone)), so the password is typed only on the phone and never reaches the TV.
- Show the **short URL and code** next to the QR for users who can't scan.
- **Expire the session** after a few minutes, and **stop listening** when the user leaves the screen or the app is suspended.
- Keep the on-screen keyboard as the alternative on the same screen; not everyone has a phone at hand.

## Voice as an advanced option

Voice is the fastest way to search when it works, but support differs a lot:

| Platform | Voice |
| --- | --- |
| Android TV | Voice search through the system assistant, which can send the query to your app's search if you integrate with it; dictation on some keyboards. |
| Apple TV | Siri dictation on the tvOS keyboard (the microphone button); Siri search across apps if you integrate with it. |
| LG webOS | Voice through the Magic Remote on supported models and regions. |
| Samsung Tizen | Voice assistant on supported remotes and regions. |

- Treat voice as **an extra path, never the only one**. Many remotes have no microphone, and support varies by model, region, and language.
- When the platform puts dictated text into the field, your search should handle it like typed text (live results, no extra step).
- Deep integration (system-wide voice search opening your app on a result) is platform work: do it after the basics work, behind an adapter.

> **Verify:** check the current Android TV, tvOS, LG, and Samsung docs for voice and search integration on the versions you support. Voice features depend on region and language, so test with the languages your app ships.
