# Focusable vs non-focusable

Only some elements can take focus. On native TV, `Pressable`, `TouchableOpacity`, `TouchableHighlight` and `Button` are focusable. A plain `View` or `Text` is not. If a card isn't a focusable component, the user can't reach it.

```tsx
<Pressable
  onPress={() => openDetails(item.id)}
  onFocus={() => setFocused(true)}
  onBlur={() => setFocused(false)}
  style={[styles.card, focused && styles.cardFocused]}
>
  ...
</Pressable>
```

- **Always draw a focus state.** On Android TV there is no default highlight for custom components. `Pressable`'s `style` function also receives `{ focused }`.
- **Keep `onFocus` cheap.** It runs on every arrow press.
- **Disable, don't hide.** A `disabled` button is skipped by focus. A visible button that can't be focused confuses users.
- `focusable={false}` makes an element unreachable on purpose (decoration, read-only content).

### Initial focus with `hasTVPreferredFocus`

Decides where focus lands when a screen opens. Without it, the platform picks something, often the top-left element.

```tsx
<Pressable hasTVPreferredFocus onPress={play}>
  <Text>Play</Text>
</Pressable>
```

Only one element per screen should have it. Put it on the action the user most likely wants (see press cost in [The TV mindset](./01-tv-mindset.md#press-cost)).
