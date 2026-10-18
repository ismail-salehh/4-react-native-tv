# Modals and overlays

- **Trap focus inside.** While a modal or overlay is open, focus must not escape to the screen behind it. RN's `Modal` opens a separate native window, which mostly does this for you. A custom overlay (an absolutely positioned `View`) does **not**. Wrap it in `TVFocusGuideView` with all four `trapFocus*` props.
- **Set initial focus** inside the modal (`hasTVPreferredFocus` on the safe/primary action).
- **Back closes the modal.** It's the topmost Back owner while open (see below).
- **Restore focus on close.** Remember what opened the modal (its id) and send focus back to it. Otherwise focus lands at the top of the screen or nowhere.
