# Component tests for focus and key-handling logic

Jest runs without a TV, without a focus engine and (on web) without real layout. So test **your logic** here, not the platform's focus movement.

### Pull the logic into plain functions

Most TV logic is a decision: "given this key and this state, what happens?" Write it as a plain function and the test needs no rendering at all.

```ts
// backOrder.ts: the single owner of Back decides with this
type BackState = { overlayOpen: boolean; keyboardOpen: boolean; menuOpen: boolean; canGoBack: boolean };
type BackAction = 'closeOverlay' | 'closeKeyboard' | 'focusContent' | 'goBack' | 'exitPrompt';

export function backAction(s: BackState): BackAction {
  if (s.overlayOpen) return 'closeOverlay';
  if (s.keyboardOpen) return 'closeKeyboard';
  if (s.menuOpen) return 'focusContent';
  if (s.canGoBack) return 'goBack';
  return 'exitPrompt';
}
```

```ts
// backOrder.test.ts
const base = { overlayOpen: false, keyboardOpen: false, menuOpen: false, canGoBack: false };

test('overlay closes before anything else', () => {
  expect(backAction({ ...base, overlayOpen: true, canGoBack: true })).toBe('closeOverlay');
});

test('menu open on Home moves focus back to content, does not exit', () => {
  expect(backAction({ ...base, menuOpen: true })).toBe('focusContent');
});

test('root screen asks before exit', () => {
  expect(backAction(base)).toBe('exitPrompt');
});
```

Other good candidates for plain functions:

- **Key mapping**: `keyFromEvent(e)` returns `'back'` for 461 on LG and 10009 on Samsung, `'right'` for `ArrowRight` and `keyCode` 39. Test every code in your [key matrix](../docs/02-remote-and-focus.md#key-matrix-per-platform-and-per-brand).
- **Focus restore**: `restoreIndex(items, lastFocusedId)` returns the saved item's index, the nearest valid index if it was removed, and `0` for an empty id.
- **Grid moves**: in a custom grid, `nextIndex(index, 'down', columns, count)` at edges, on the last short row, in RTL.

### Component tests with React Native Testing Library

When the logic lives in a component, render it and fire the events the focus engine would send. `fireEvent(element, 'focus')` calls the `onFocus` prop.

```tsx
import { render, screen, fireEvent } from '@testing-library/react-native';

test('card shows focus style only while focused', () => {
  render(<Card item={{ id: '1', title: 'Dune' }} onSelect={jest.fn()} />);
  const card = screen.getByTestId('card-1');

  fireEvent(card, 'focus');
  expect(card).toHaveStyle({ borderColor: 'white' });

  fireEvent(card, 'blur');
  expect(card).not.toHaveStyle({ borderColor: 'white' });
});

test('pressing OK twice while loading opens the player once', async () => {
  const openPlayer = jest.fn(() => new Promise(r => setTimeout(r, 100)));
  render(<PlayButton openPlayer={openPlayer} />);

  fireEvent.press(screen.getByText('Play'));
  fireEvent.press(screen.getByText('Play'));

  expect(openPlayer).toHaveBeenCalledTimes(1);
});

test('focusing a card saves its id for restore', () => {
  const saveFocus = jest.fn();
  render(<Rail rail={rail} onItemFocus={saveFocus} />);
  fireEvent(screen.getByTestId('card-42'), 'focus');
  expect(saveFocus).toHaveBeenCalledWith(rail.id, '42');
});
```

**Back on native.** React Native ships a mock `BackHandler` with a `mockPressBack()` helper. Use it to test that the screen that should handle Back does, and that it returns `true`:

```tsx
jest.mock('react-native/Libraries/Utilities/BackHandler', () =>
  require('react-native/Libraries/Utilities/__mocks__/BackHandler'),
);
import { BackHandler } from 'react-native';

test('Back closes the modal and not the screen', () => {
  const goBack = jest.fn();
  render(<DetailsScreen goBack={goBack} />);
  fireEvent.press(screen.getByText('More info')); // opens modal

  BackHandler.mockPressBack();

  expect(screen.queryByTestId('info-modal')).toBeNull();
  expect(goBack).not.toHaveBeenCalled();
});
```

**Keys on web.** With `@testing-library/react` (jsdom), fire key events on `window` with the TV's codes:

```tsx
fireEvent.keyDown(window, { keyCode: 461 }); // LG Back
expect(onBack).toHaveBeenCalledTimes(1);
```

What **not** to test in Jest:

- **Real focus movement.** Which card the native focus engine picks on Right depends on real geometry. jsdom has no layout (every box is 0×0), so a spatial navigation library like Norigin can't compute neighbours there either. Test this with smoke tests on a real build.
- **Platform internals.** Don't mock `useTVEventHandler` deeply to prove that the OS sends events. Test your handler function with a fake event object instead.

> **Verify:** check the paths of React Native's `BackHandler` mock and the jest setup in your React Native / react-native-tvos version; internal module paths can move between versions.