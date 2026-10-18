# How focus works on web

### Spatial navigation library (Norigin in tv-app)

`tv-app` uses [`@noriginmedia/norigin-spatial-navigation`](https://github.com/NoriginMedia/Norigin-Spatial-Navigation). It listens to key events, keeps its own tree of focusable components, and on each arrow press computes the next element by position.

Setup, once at app start:

```tsx
import { init } from '@noriginmedia/norigin-spatial-navigation';

init({
  // debug: true,           // logs focus decisions
  // visualDebug: true,     // draws focusable areas on screen
});
```

A focusable component:

```tsx
import { useFocusable } from '@noriginmedia/norigin-spatial-navigation';

function Card({ item, onSelect }) {
  const { ref, focused } = useFocusable({
    focusKey: `card-${item.id}`,
    onEnterPress: () => onSelect(item),
    onFocus: () => scrollIntoView(item.id),
  });

  return (
    <div ref={ref} className={focused ? 'card card--focused' : 'card'}>
      {item.title}
    </div>
  );
}
```

A container (rail, menu, modal) that groups children:

```tsx
import { useFocusable, FocusContext } from '@noriginmedia/norigin-spatial-navigation';

function Rail({ rail }) {
  const { ref, focusKey } = useFocusable({
    focusKey: `rail-${rail.id}`,
    saveLastFocusedChild: true,   // return to the card the user left
    trackChildren: true,
  });

  return (
    <FocusContext.Provider value={focusKey}>
      <div ref={ref} className="rail">
        {rail.items.map(item => <Card key={item.id} item={item} />)}
      </div>
    </FocusContext.Provider>
  );
}
```

How the native concepts map to Norigin:

| Native (react-native-tvos) | Web (Norigin) |
| --- | --- |
| Focusable `Pressable` | `useFocusable()` + `ref` on the element |
| `onFocus` / `onBlur` | `onFocus` / `onBlur` options, or the `focused` value |
| `onPress` | `onEnterPress` |
| `hasTVPreferredFocus` | `setFocus('focus-key')` or `focusSelf()` in an effect |
| `nextFocus*` | `onArrowPress` returning `false` and calling `setFocus(...)` |
| `TVFocusGuideView autoFocus` | `saveLastFocusedChild: true` on the parent |
| `TVFocusGuideView trapFocus*` | `isFocusBoundary: true` (+ `focusBoundaryDirections`) |
| `BackHandler` | Your own `keydown` listener for 461 (LG) / 10009 (Samsung) |

Restoring focus by item id works the same way as native: store the id and call ``setFocus(`card-${id}`)`` once the screen has rendered.

### RN focus APIs mapped onto react-native-web (web)

`react-native-web` renders RN components as DOM elements. For focus that means:

- `Pressable` / `Touchable*` become elements with `tabIndex`, so they can receive DOM focus, and `onFocus` / `onBlur` fire as DOM events.
- **TV-specific props do nothing on web**: `hasTVPreferredFocus`, `nextFocusUp/Down/Left/Right`, `TVFocusGuideView`, `useTVEventHandler`, `TVEventControl`. They don't exist in react-native-web; they are ignored or missing.
- `BackHandler` is a no-op on web.
- `Platform.isTV` is not set; branch on `Platform.OS === 'web'`.

So shared components can render the same, but the **focus wiring must be split** between native and web (`Card.native.tsx` / `Card.web.tsx`, or a small `Focusable` wrapper that uses `Pressable` on native and `useFocusable` on web).
