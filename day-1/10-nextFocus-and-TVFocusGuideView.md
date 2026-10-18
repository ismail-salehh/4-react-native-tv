# `nextFocus*` and `TVFocusGuideView`

The native focus engine moves focus to the **nearest focusable element in the pressed direction**. Usually that's right. When it isn't, you have two tools.

### `nextFocusUp` / `nextFocusDown` / `nextFocusLeft` / `nextFocusRight`

Point one element at another explicitly, using a node handle:

```tsx
const playRef = useRef(null);

<Pressable
  nextFocusDown={findNodeHandle(playRef.current) ?? undefined}
  ...
/>
<Pressable ref={playRef} ... />
```

Good for single fixes. Hard to maintain when you use many of them.

### `TVFocusGuideView`

A container that redirects focus into itself and controls what happens at its edges:

```tsx
import { TVFocusGuideView } from 'react-native';

<TVFocusGuideView autoFocus trapFocusLeft trapFocusRight style={styles.menu}>
  {menuItems}
</TVFocusGuideView>
```

- `autoFocus`: when focus enters the guide, send it to the **last focused child** (or the first). This is the easiest way to get "remember position" in menus and rails.
- `destinations={[ref1, ref2]}`: when focus enters the guide, send it to these elements.
- `trapFocusUp/Down/Left/Right`: focus can't leave in that direction.

Use guides to bridge gaps: if two areas aren't aligned (e.g. a short menu next to a tall grid), an empty guide in the gap catches the press and forwards it.