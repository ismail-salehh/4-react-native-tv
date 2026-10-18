# Hardware limits

TV hardware is weak and stays in homes for 7–10 years:

- **Slow CPUs.** Low-end ARM SoCs, often slower than a budget phone. JS work blocks focus movement, so a heavy render on focus feels like a broken remote.
- **Low RAM.** 1–1.5 GB shared with the OS and the video decoder. Large images and big lists get the app killed or reloaded.
- **Old OS versions.** Android TV boxes on Android 7–9, old tvOS versions, and TVs that never get firmware updates.

What to do about it:

- Virtualize every list (`FlatList` with `windowSize`, `initialNumToRender`, `removeClippedSubviews`, `getItemLayout`).
- Request images at the size you draw them. Don't load a 4K poster into a 300 px card.
- Keep focus handlers cheap: no network calls or large state updates on every `onFocus`. Debounce the "preview the focused item" logic.
- Animate with the native driver (`useNativeDriver: true`) or CSS transforms on web; animate `transform` and `opacity`, not layout.
- Test release builds. Dev builds are much slower and hide or exaggerate problems.
