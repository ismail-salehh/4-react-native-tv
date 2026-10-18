# `Platform.isTV`, `useTVEventHandler`, `BackHandler`

### `Platform.isTV`

`true` on Android TV and Apple TV. Use it for TV-only layout or behavior in shared code:

```tsx
import { Platform } from 'react-native';

const CARD_WIDTH = Platform.isTV ? 320 : 160;
```

`Platform.isTV` is not set on the web build. Detect web TVs separately (e.g. a build-time flag, or `Platform.OS === 'web'` plus the user agent).

### `useTVEventHandler`

Receives remote events app-wide (react-native-tvos). Use it for **global** keys like Play/Pause, not for moving focus. The native focus engine already handles arrows.

```tsx
import { useTVEventHandler } from 'react-native';

useTVEventHandler(evt => {
  if (evt.eventType === 'playPause') {
    player.toggle();
  }
});
```

On Android, `evt.eventKeyAction` tells you key down (`0`) from key up (`1`). Check it, or every press fires your handler twice.

### `BackHandler`

The single entry point for the Back key on Android TV, and on tvOS once the Menu key is enabled:

```tsx
import { BackHandler, TVEventControl } from 'react-native';

useEffect(() => {
  TVEventControl.enableTVMenuKey(); // tvOS: let Menu reach BackHandler
  const sub = BackHandler.addEventListener('hardwareBackPress', () => {
    if (canGoBack()) {
      goBack();
      return true;   // handled: do not exit
    }
    return false;    // not handled: OS default (exit app / go to home)
  });
  return () => sub.remove();
}, []);
```

On tvOS, apps must let Menu at the root screen go back to the Apple TV home screen (App Store review checks this). Call `TVEventControl.disableTVMenuKey()` when you are on the root screen.