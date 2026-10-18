# Dead ends, traps, unexpected jumps, lost focus

| Problem | Symptom | Typical cause | Fix |
| --- | --- | --- | --- |
| **Dead end** | Pressing a direction does nothing, but there is content that way | No focusable element is geometrically "in that direction" | `TVFocusGuideView` with `destinations`, or `nextFocus*` |
| **Trap** | User can't get out of an area | Over-eager `trapFocus*`, or a list that swallows keys | Only trap in modals; test every edge |
| **Unexpected jump** | Focus skips to a far-away element | Nearest-element search picks a different element than the eye expects (rows with different heights, offset layouts) | Align layouts, add guides, set explicit `nextFocus*` |
| **Lost focus** | No element is highlighted; arrows do nothing | The focused element was unmounted (list refresh, loading state, conditional render) | Keep focused elements mounted; restore focus after data changes |

Recovering when focus is lost:

- **Don't unmount the focused element.** Show loading over the old content instead of replacing it.
- **Use stable `key`s** in lists so a re-render doesn't remount items.
- **Have a fallback.** After a data refresh, if the previously focused id is gone, focus the nearest item or the first item of the rail, and failing that a known safe element (e.g. the menu).
- **Test it:** focus an item, trigger a refresh, then press an arrow. If nothing moves, focus was lost.
