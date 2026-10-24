1. Remote input
- Is the right key event handler being used (useTVEventHandler / TVEventControl)?
- Are D-pad keys (up/down/left/right/select) mapped and logged?
- Do key events fire twice (keydown and keyup both handled)?
- Does long-press or fast repeat flood the handler?
- Is input blocked while a modal, overlay or loading state is shown?
- Is the handler removed on unmount so it doesn't fire on the wrong screen?
- Do platform-specific keys (Play/Pause, Menu, color buttons) behave the same across devices?
2. Focus
- Is anything focused when the screen first loads (hasTVPreferredFocus)?
- Is every interactive element focusable (Pressable/Touchable, not a plain View)?
- Is the focused item clearly visible (scale, border, color)?
- Does focus get lost or vanish after a list re-renders or data reloads?
- Does focus escape to hidden or off-screen elements?
- Are nextFocusUp/Down/Left/Right or TVFocusGuideView used where the default direction guesses wrong?
- Does focus come back to the last item when returning to a screen?
- Do lists scroll to keep the focused item in view?
3. Back button
- Is the hardware back/Menu press handled (BackHandler / TVEventControl menu key)?
- Does back close the top layer first (modal, drawer, player) before leaving the screen?
- Does back on the root screen exit the app or show a confirmation, as the platform expects?
- Are listeners removed on unmount so back doesn't fire twice or on the wrong screen?
- Does navigation history stay in sync with what's on screen?
- On tvOS, is the menu key enabled or disabled at the right times?
4. Layout and clipping
- Is the layout inside the TV safe area (overscan margins on the edges)?
- Does the focus scale or border get cut off by a parent with overflow hidden?
- Do horizontal rows have enough padding for the first and last focused items?
- Are sizes based on screen dimensions instead of fixed pixels meant for phones?
- Does it look right at 720p, 1080p and 4K?
- Is text readable from across the room (font size, contrast)?
- Do zIndex and elevation put the focused item on top of its neighbors?
5. Memory
- Are big lists virtualized (FlatList with windowSize, initialNumToRender, removeClippedSubviews)?
- Are images resized for the screen instead of loading full-resolution posters?
- Is an image cache with limits in use?
- Are timers, intervals, listeners and subscriptions cleaned up on unmount?
- Are video players released when leaving the player screen?
- Does memory keep climbing after going back and forth between screens many times?
- Is the app tested on a low-end device with little RAM?
6. Works on Chrome but breaks on a real TV
- Does the TV's web engine version support the JS features and APIs you use (check polyfills and transpile target)?
- Are you using CSS features the older TV engine doesn't support (gap, aspect-ratio, newer flexbox behavior)?
- Are web-only APIs used that don't exist on the TV (some storage, fetch options, Intl)?
- Do you rely on mouse or hover events that a remote never sends?
- Is performance OK on the TV's weaker CPU and GPU (animations, shadows, blur)?
- Do media codecs and formats play on the TV's decoder?
- Do key codes differ between the TV remote and the keyboard you test with?
- Is the app tested on the real device or emulator with remote debugging, not just Chrome DevTools?
- Do network rules differ on the TV (HTTPS only, CORS, mixed content)?
