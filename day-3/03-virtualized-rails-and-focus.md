# Virtualised rails and focus

Long rails and grids use a virtualised list (`FlatList`, `FlashList`, or a web equivalent). Only the items near the visible area are mounted; the rest are unmounted or recycled.

**An item that is unmounted cannot keep focus.** If the focused card is removed, there is nothing for the focus engine to hold on to, and focus goes somewhere random or nowhere.

How it happens:

- The user scrolls the rail by focus, but the list's window is too small, so the item they just left (or the next one) is not mounted when focus tries to move to it.
- A rail far down the screen is unmounted while the user is focused in it, e.g. the vertical list virtualises its rails.
- Data refreshes and the list re-keys its items, unmounting and remounting the focused card.
- **Recycling** (`FlashList`, `removeClippedSubviews` on Android): the native view that held focus is reused for a different item, so focus appears to "jump" to the wrong card, or is dropped.

What to do:

- **Keep the focused item and its neighbours mounted.** Give the list enough `windowSize` / `initialNumToRender` that the next few items in each direction are always rendered. Scroll so the focused item stays inside the window (`scrollToIndex` in `onFocus`, or a fixed focus position like "second card from the left").
- **Don't use `removeClippedSubviews`** on TV lists unless you've tested focus with it on the real device.
- **Use stable keys** (`keyExtractor={item => item.id}`), never the index, so a refresh doesn't remount the focused card.
- **Restore by id.** Remember the last focused item id per rail (in a ref or store). If focus is lost, or the user comes back, scroll to that id and focus it (see [Remote and focus](../docs/02-remote-and-focus.md#restore-focus-by-item-id-when-returning-to-a-screen)).
- **Short rails don't need virtualising.** A rail of 20 cards can be a plain `ScrollView`. Virtualise the vertical list of rails, and long grids.

> **Verify (web):** check what your spatial navigation library does when the focused component unmounts (Norigin: whether focus moves to the parent or is lost), and test holding Right on a long rail on a real LG and Samsung TV.