# Keep focus state local to the focused item

Every press of an arrow key changes which item is focused. If that change goes through state high up in the tree, every press re-renders the whole screen.

```tsx
// Bad: focus in top-level context
const FocusContext = createContext<{ focusedId: string; setFocusedId: (id: string) => void }>(...);

function Card({ item }) {
  const { focusedId, setFocusedId } = useContext(FocusContext);
  const focused = focusedId === item.id;
  return (
    <Pressable onFocus={() => setFocusedId(item.id)} style={focused && styles.focused}>
      ...
    </Pressable>
  );
}
```

Every card on the screen reads the context, so when `focusedId` changes, **every card re-renders**: 10 rails × 20 cards = 200 renders for one key press. On a slow TV that can take 50–150 ms, and when the user holds Right the presses pile up and focus lags behind.

Keep the focused flag inside the item:

```tsx
// Good: each card knows only about itself
function Card({ item, onSelect }) {
  const [focused, setFocused] = useState(false);
  return (
    <Pressable
      onFocus={() => setFocused(true)}
      onBlur={() => setFocused(false)}
      onPress={() => onSelect(item.id)}
      style={focused && styles.focused}
    >
      ...
    </Pressable>
  );
}
```

Now one press re-renders **two** cards: the one that lost focus and the one that got it. On web, Norigin's `useFocusable` already works this way: it returns `focused` for that component only.

When something else needs to know what is focused (a hero background that shows the focused title, analytics):

- **Store the last focused id in a ref**, not state, when you only need it later (e.g. to restore focus when coming back to the screen).
- **Use a small store with a selector** (`useSyncExternalStore`, Zustand, etc.) so only the component that reads it re-renders, not every card.
- **Debounce** expensive reactions (loading hero artwork, starting a trailer) by 200–400 ms, so holding Right doesn't start 20 image loads.

```tsx
// focusStore.ts: only subscribers re-render
let focusedId: string | null = null;
const listeners = new Set<() => void>();

export const focusStore = {
  set(id: string) { focusedId = id; listeners.forEach(l => l()); },
  get: () => focusedId,
  subscribe(l: () => void) { listeners.add(l); return () => listeners.delete(l); },
};

export const useFocusedId = () =>
  useSyncExternalStore(focusStore.subscribe, focusStore.get);

// Card: focusStore.set(item.id) in onFocus (no re-render of other cards)
// Hero: const id = useFocusedId(); (only Hero re-renders)
```