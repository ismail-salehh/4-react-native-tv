# Clipping and layout failures

**Symptom:** focused card cut off at the edges, focus border hidden behind the next card, text cut or overlapping, content in the overscan area, gaps missing between cards, layout only broken at one resolution.

**Reproduce:** focus the first and last card of a rail, the top and bottom rail, the longest title, and check at 720p, 1080p and 4K (see [Testing](./03-testing.md#test-at-720p-1080p-and-4k-and-on-a-low-end-device)).

| Symptom | Usual cause | Fix |
| --- | --- | --- |
| Scaled card cut at top/bottom or left/right | Parent with `overflow: 'hidden'`; Android `ScrollView` / `FlatList` clip by default | Padding on the list's `contentContainerStyle` for the growth; `overflow` only on the image wrapper ([Don't clip the scale](../docs-2/01-layout-and-styling.md#dont-clip-the-scale-with-overflow)) |
| Focus border behind the next card | Focused cell is not on top | Higher `zIndex` on the cell via `CellRendererComponent` |
| First card touches the screen edge or is cut | No safe-area padding, or scale grows past it | Safe area padding on the list ([Safe areas](../docs-2/01-layout-and-styling.md#safe-areas-overscan-tv-safe-margins)) |
| Cards touch each other on TV only | Flex `gap` on an old engine | Margins instead of `gap` ([no flex gap](../docs-2/01-layout-and-styling.md#old-engines-no-flex-gap-web)) |
| 1 px gaps or blurry borders | Fractional pixels from scaling | Round the scaled values |
| Text overflows its box | Long titles, translations, bigger font in another language | `numberOfLines`, test with the longest real title and every language |
| Everything is too small or too big | App renders at a different size than designed (720p UI on a 4K TV, wrong scale factor) | Scale from the actual window width ([Design at 1920×1080](../docs-2/01-layout-and-styling.md#design-at-19201080-and-scale)) |

**Check:**

- **Colour the boxes.** Temporarily give the list, the rail and the card different background colours. Clipping is obvious when you see where each box ends.
- Android: Developer options → **Show layout bounds**.
- Web: the element inspector in Chrome and in the remote inspector on the TV. Hover the parent of the cut card and look for `overflow: hidden`.
- Remember the [layout vs focus check](./01-troubleshooting-workflow.md#layout-bug-or-focus-bug): press OK first to be sure focus is fine.
