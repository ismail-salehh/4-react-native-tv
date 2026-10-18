# LG Magic Remote acts as a pointer, so hover can happen (LG web)

The LG Magic Remote has a gyroscope pointer. When the user waves it, a cursor appears and the TV sends **mouse events** (`mouseover`, `mousemove`, `click`). When they press an arrow key, the cursor hides and you're back to key events.

What this means for the app:

- **Pointer and focus must agree.** When the cursor hovers a card, make that card the focused one, so the next arrow press continues from there:
  ```tsx
  const { ref, focused, focusSelf } = useFocusable({ ... });

  <div ref={ref} onMouseEnter={focusSelf} onClick={() => onSelect(item)}>
  ```
- **Clicks must work** on everything focusable, not just the OK key.
- **Hover styles = focus styles.** Don't style `:hover` differently from focus, or the user sees two highlighted items.
- **Detect the cursor state** if you need to (e.g. show on-screen scroll arrows only in pointer mode):
  ```js
  document.addEventListener('cursorStateChange', e => {
    const pointerVisible = e.detail.visibility;
  });
  ```
- **Wheel scrolling**: the Magic Remote wheel sends `wheel` events. Make sure scrolling lists with it doesn't leave focus on an off-screen item.

Samsung remotes have no pointer by default, so hover is mostly an LG concern. Still, nothing must *require* hover.
