# Focus and navigation smoke tests

A smoke test runs the real app build, presses keys, and checks that focus ended up where it should. It catches the bugs component tests can't: lost focus, dead ends, broken Back, a screen that won't open.

### Make focus visible to tests

Tests need to ask "what is focused now?". Give every focusable element a stable id, and expose focus:

- **Native**: `testID` on each focusable (`card-${item.id}`, `menu-home`). Test tools can read the focused element.
- **Web**: a `data-testid` and a `data-focused="true"` attribute when focused (Norigin gives you `focused` from `useFocusable`), or move DOM focus with it so `document.activeElement` is correct.

### Key sequences

Write the user's path as a list of key presses and the expected focused element after it. Keep each sequence short (one journey) and run them on every release build.

**Android TV**: Maestro flows can press remote keys:

```yaml
# home-to-details.yaml
appId: com.example.tvapp
---
- launchApp
- assertVisible:
    id: "card-hero-1"
- pressKey: Remote Dpad Down
- pressKey: Remote Dpad Right
- pressKey: Remote Dpad Center
- assertVisible: "Play"
- pressKey: Back
- assertVisible:
    id: "card-rail1-2"     # back on the same card
```

Without a test tool, `adb` alone is enough for a quick script:

```sh
for k in DPAD_DOWN DPAD_RIGHT DPAD_CENTER BACK; do
  adb shell input keyevent "KEYCODE_$k"; sleep 1
done
```

**Apple TV**: XCUITest drives the Siri Remote with `XCUIRemote.shared.press(.right)`, `.select`, `.menu`, and can check which element has focus (`hasFocus`).

**Web (LG, Samsung)**: Playwright or Puppeteer against the production build in Chrome, with the TV key codes:

```ts
await page.keyboard.press('ArrowDown');
await page.keyboard.press('ArrowRight');
await expect(page.locator('[data-focused="true"]')).toHaveAttribute('data-testid', 'card-rail1-2');

// Back as the TV sends it
await page.evaluate(() =>
  window.dispatchEvent(new KeyboardEvent('keydown', { keyCode: 461 } as any)));
```

This runs on a modern Chrome, so it tests your navigation, not the TV engine. Repeat the important journeys by hand on the TV.

Sequences worth having:

- Launch → first focus is on the expected element (not nothing).
- Every menu item → its screen opens with focus inside the content.
- Home → card → Details → Back: focus on the same card.
- Open a modal → Back: modal closed, focus on the button that opened it.
- Hold Right 30 presses on the longest rail → focus still visible and on a card.
- Down through every rail and Up again → focus returns to the card left in each rail.
- Back from every screen until the root → exit prompt (or exit), never a blank screen.

> **Verify:** check the current Maestro docs for TV key names and Android TV support, and the XCUITest docs for `XCUIRemote` on your Xcode version.

### Manual focus walk

Automated sequences check the paths you wrote. A **focus walk** finds the ones you didn't. Before calling a screen done, on a real device, with only the remote:

1. **From the initial focus, press every direction** from every element at the edges: first and last card of each rail, top and bottom rail, menu items, buttons. Nothing should be a dead end when there is content in that direction, and nothing should jump far away.
2. **Check the focus is always visible**: every element shows a clear focus style, never clipped, never hidden behind another card.
3. **Open and close everything**: modals, menus, the keyboard. Focus starts inside and returns to where it was.
4. **Press Back at every level** and check it does the most specific thing first.
5. **Do it while things change**: during loading, right after data refreshes, after an error, after coming back from the background.
6. **Do it in RTL** if you ship a right-to-left language: Left and Right must still match what the user sees.
7. **Note every problem with the key sequence** that triggers it, so it can become a smoke test.
