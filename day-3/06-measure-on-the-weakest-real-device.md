# Measure on the weakest real device with a release build

Your laptop and the emulator are 5–20× faster than a cheap TV. Development builds are slower in different ways (extra checks, no minification, dev-only warnings). Numbers from either are not the numbers your users get.

- **Pick the weakest device you support** (oldest LG, cheapest Android TV stick) and measure there. If it is fast on that one, it is fast everywhere.
- **Use a release / production build**: `--mode release` / Release scheme on native, `mode: 'production'` on web, installed on the TV.
- **Measure the same things every time**:

| Metric | How |
| --- | --- |
| Cold start to first focused card | Timestamp at app start and at first focus; log both. Stopwatch is fine. |
| Key press to focus moved | Log `performance.now()` on key event and on the next focus change. |
| Holding Right on a long rail | Watch for focus lag and dropped frames. |
| Memory after browsing 5 sections | Device profiler or remote inspector. |
| Back to Home | Should be instant if Home is kept mounted. |

- **Profile, then fix.** Android Studio profiler / Perfetto for Android TV, Instruments for Apple TV, the remote inspector's Performance tab for LG and Samsung. React DevTools Profiler shows which components re-render on a key press (use it in a profiling build; dev builds exaggerate render cost).
- **Write the numbers down** and re-measure after each change, so you know whether it helped.

## Explain it out loud

**Why does top-level focus state hurt performance?**

Focus changes on every single key press, so it is the most frequently changing state in a TV app. If the focused id lives in top-level context or a screen's state, every component that reads it re-renders on every press: on a home screen that is every card in every rail, often a couple of hundred components. On a slow TV that takes tens to hundreds of milliseconds, so focus visibly lags the remote, and when the user holds an arrow key the presses queue up and focus keeps moving after they let go.

The fix is to keep the "am I focused?" flag inside each item, so one press re-renders only the two items involved. Anything that genuinely needs the focused id (like a hero background) subscribes to a small store with a selector, so only it re-renders, and expensive reactions are debounced. The React Compiler does not save you here: a changed context value is a changed input, and every consumer still re-renders.

**Why can focus be lost inside a virtualised list?**

A virtualised list only mounts the items near the visible area and unmounts (or recycles) the rest to save memory. Focus belongs to a mounted element. If the focused item, or the item focus is about to move to, is outside the list's render window, it doesn't exist at that moment, so the focus engine has nothing to hold on to and focus falls to some other element or to nothing. Recycling lists make it worse: the native view that had focus is reused for a different item, so focus seems to jump to the wrong card. Re-keying items on a data refresh has the same effect.

To prevent it: keep the focused item and its neighbours inside the render window (enough `windowSize`, scroll the focused item into view), avoid `removeClippedSubviews` on TV, use stable item ids as keys, and remember the last focused id per rail so focus can be restored if it is lost.