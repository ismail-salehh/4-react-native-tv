# Living room, ten-foot UI, and press cost

**Ten-foot UI** means designing for a screen viewed from about three meters (ten feet) away:

- **Large text.** Body text around 24–28 px at 1080p; nothing under ~20 px.
- **High contrast and a clear focus state.** The user must find the focused item in under a second. Use scale, border, glow or a color change, and ideally more than one of them.
- **Safe area / overscan.** Keep important content about 5% in from every edge (roughly 48 px horizontal and 27 px vertical at 1080p). Some TVs still crop the edges.
- **Few items on screen.** One main action per screen, rails of 5–7 visible cards, not dense lists.
- **Lean-back use.** The user is relaxed, often distracted, and sometimes holding a drink. Don't ask them to type or think hard.

### Press cost

On mobile a tap anywhere costs the same. On TV, **every D-pad press is a cost** the user pays. Treat it like a performance metric.

How to measure a flow:

1. Write down the flow, e.g. *"Home → open the 4th movie in the 3rd rail → play"*.
2. Count every key press from app launch to the goal:

   ```
   Down, Down (reach rail 3)        = 2
   Right, Right, Right (4th card)   = 3
   OK (open details)                = 1
   OK (Play is focused by default)  = 1
                              total = 7
   ```
3. Cut it down:
   - Put the most likely action under initial focus (`hasTVPreferredFocus` on **Play**, not on **Back** or **Add to list**).
   - Order rails by what people actually use ("Continue watching" first).
   - Make focus wrap or jump to sensible places instead of walking item by item.
   - Remember where the user was (see [Remote and focus](./02-remote-and-focus.md#rails-and-grids-remember-the-last-focused-item)).
   - Avoid text input. Offer suggestions, recent searches, or a QR code to finish on the phone.

A good rule of thumb: the main task of any screen should be reachable in **three presses or fewer** from where focus lands.

### Why mobile patterns fail

| Mobile pattern | Why it fails on TV | Use instead |
| --- | --- | --- |
| Swipe-to-open drawer | There is no swipe. A hidden drawer can only be reached if focus can travel to it, and the user doesn't know it's there. | A **visible** side menu that expands when focused. (A visible side menu is fine.) |
| Pull-to-refresh | No pull gesture exists. | Refresh automatically on focus/return, or offer a visible "Refresh" button. |
| Small touch targets, icon-only buttons | Hard to see from 3 m; the focus state is too small to notice. | Large targets with a label and an obvious focus style. |
| Gestures (pinch, long-swipe, double-tap) | The remote has none (the Siri Remote touch surface is the exception, and you can't rely on it). | Explicit buttons and menus. |
| Hover tooltips, hover menus | There is no hover on most remotes. | Show the information on **focus**. |
| Infinite scroll with tiny items | Dozens of presses to get anywhere. | Rails, paging, "jump to letter", search. |
| Bottom tab bar | Far away from where focus usually is, and easy to get trapped in. | Top nav or side menu. |
