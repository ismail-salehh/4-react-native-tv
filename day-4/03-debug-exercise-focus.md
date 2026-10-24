# Focus disappears or jumps

**Symptom:** after some action, nothing is highlighted and arrows do nothing (lost focus), or focus lands on an unexpected element (jump).

**Reproduce:** the usual triggers are a data refresh, a loading state, closing a modal, coming back with Back, and holding an arrow on a long rail. Try each while focused deep in a rail.

**Check:** log every focus change with an id, and print the last few on every key press:

```tsx
// native: in the card
onFocus={() => track(`focus ${rail.id}/${item.id}`)}
onBlur={() => track(`blur ${rail.id}/${item.id}`)}
```

```js
// web, in the remote inspector
document.addEventListener('focusin', e => console.log('focus', e.target), true);
// with Norigin, also check its own idea of focus
// SpatialNavigation.getCurrentFocusKey()
```

Read the log at the moment it breaks:

| Log shows | Cause | Fix |
| --- | --- | --- |
| `blur X` with no `focus` after | Focused element was unmounted: a loading spinner replaced the content, a conditional render, a virtualised item scrolled out | Keep the content mounted and show loading over it; keep the focused item inside the list window |
| `blur X`, `focus X` again, styles reset | The item remounted: list keyed by index, or a parent's `key` changed on refresh | Stable `keyExtractor={item => item.id}`; don't change parent keys on refresh |
| `focus` on a far-away item | Nearest-element search picked a different neighbour (uneven rows, offset layout, a hidden focusable) | Align layouts, `TVFocusGuideView` / `nextFocus*`, or Norigin focus boundaries |
| `focus` on the first item of the screen after closing a modal or Back | Nothing restored focus | Remember the opener's id; restore it ([Restore focus by item id](../docs/02-remote-and-focus.md#restore-focus-by-item-id-when-returning-to-a-screen)) |
| `focus` on an invisible element | A focusable element is behind an overlay, off-screen, or at `opacity: 0` | Make hidden things non-focusable (`focusable={false}`, unmount, or Norigin `focusable: false`) |
| Focus lands on the wrong card only when holding the key | Recycling list reused the focused native view | Avoid `removeClippedSubviews`; test `FlashList` recycling on the device |

Write a fallback for when focus is lost anyway: after a refresh, if the remembered id is gone, focus the nearest item, then the first item of the rail, then a known safe element such as the menu.
