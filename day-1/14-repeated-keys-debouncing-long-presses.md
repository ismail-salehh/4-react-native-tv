# Repeated keys, debouncing, long presses, one owner of Back

- **Repeated keys.** Holding an arrow sends a stream of key events (often 10–20 per second). The UI must keep up: cheap `onFocus`, virtualized lists, no heavy work per item. On slow TVs consider throttling scroll or using "page jump" on long hold.
- **No double actions.** A press can arrive as key down + key up, or the user presses twice while the next screen loads. Guard actions:
  ```tsx
  const busy = useRef(false);
  const onPress = async () => {
    if (busy.current) return;
    busy.current = true;
    try { await openPlayer(); } finally { busy.current = false; }
  };
  ```
  Also check key action (down vs up) in `useTVEventHandler` and key listeners.
- **Debounce side effects of focus**, e.g. load the preview/background image 300 ms after focus settles, not on every card the user passes.
- **Long presses.** `onLongPress` on `Pressable`, or `longSelect` in `useTVEventHandler`. Don't hide important features behind long press; users don't discover them. Use it for shortcuts only.
- **One owner of Back.** Exactly one place in the app decides what Back does (see below).

### Explain it out loud

**Who should own the Back button?**

One central owner: the navigation layer (a single `BackHandler` listener, or the router), not each component on its own. Back should do the most specific thing first, in a fixed order:

1. Close the topmost overlay/modal, if one is open.
2. Exit a sub-state (close search keyboard, leave the side menu and return focus to content, exit fullscreen player).
3. Go back one screen in the navigation stack.
4. At the root: scroll/focus back to the top first, then on the next press show an exit confirmation (or let the OS exit, as tvOS requires).

If several components each register their own Back listener, they fight: one press closes the modal *and* goes back a screen, or nothing happens. Components can **register** with the owner ("I'm open, call me first"), but the owner decides.

**Why can touch and hover not be relied on?**

Most TV remotes have no pointer and no touchscreen, only a D-pad and a few buttons. Touch doesn't exist on the TV. Hover only exists on some devices (LG Magic Remote, a mouse plugged into an Android TV box), so a feature that depends on hover is unreachable for everyone else. And when hover *does* exist, it must not fight with focus: the D-pad user and the pointer user must see the same highlighted element. Everything must work with focus + OK + Back alone; pointer support is an extra on top.

---

Previous: [The TV mindset](./01-tv-mindset.md) · Next: [Focus on the web build](./03-focus-on-web.md)
