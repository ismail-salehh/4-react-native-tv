# Rails and grids

Rails (horizontal lists stacked vertically) and grids are the core of most TV apps.

### Remember the last focused item

When the user moves down from rail 2 to rail 3 and back up, focus should return to **the card they left in rail 2**, not the first card and not whichever card is geometrically closest.

- `TVFocusGuideView autoFocus` around each rail does this on native.
- Or store it yourself: `lastFocused[railId] = itemId` in `onFocus`, and give that item `hasTVPreferredFocus` / route focus to it on entry.

### Restore focus by item id when returning to a screen

Home → Details → Back should land on the same card. Save the **item id**, not the index or the ref (data may have reordered; refs died when the screen unmounted):

```tsx
// on press
navigationState.setReturnFocus({ railId: rail.id, itemId: item.id });

// when rendering the rail
const restoreIndex = rail.items.findIndex(i => i.id === returnFocus?.itemId);

<FlatList
  horizontal
  data={rail.items}
  initialScrollIndex={restoreIndex > 0 ? restoreIndex : 0}
  getItemLayout={getItemLayout}
  renderItem={({ item }) => (
    <Card item={item} hasTVPreferredFocus={item.id === returnFocus?.itemId} />
  )}
/>
```

Watch out for **virtualization**: if the item isn't rendered (it's off-screen), it can't take focus. Scroll to it first (`initialScrollIndex` + `getItemLayout`), then focus it.