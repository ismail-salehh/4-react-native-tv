# Error boundaries per rail and screen

A JavaScript error during render unmounts the whole React tree unless something catches it. On a TV that means a black screen, and the user's only option is to quit the app.

An **error boundary** catches render errors in its children and shows a fallback instead. Place them at three levels:

| Level | Catches | Fallback |
| --- | --- | --- |
| **Per rail** | A bad item, a broken card, an unexpected data shape in one rail | Hide the rail, or show a short "Couldn't load" row with a focusable Retry. The rest of the screen keeps working. |
| **Per screen** | A crash in one screen | The screen's error state: short message, focused **Retry**, Back works. The menu stays usable. |
| **Root** | Anything else, including navigation and providers | "Something went wrong" with a focused **Restart** button that remounts the app (or reloads on web). |

```tsx
import { ErrorBoundary } from 'react-error-boundary';

function Rails({ rails }) {
  return rails.map(rail => (
    <ErrorBoundary
      key={rail.id}
      fallback={null}                     // or <RailError onRetry={...} />
      onError={(error) => logError(error, { rail: rail.id })}
      resetKeys={[rail.items]}
    >
      <Rail rail={rail} />
    </ErrorBoundary>
  ));
}
```

Things to get right:

- **Focus after the fallback.** If the focused card was inside the rail that crashed, it is now unmounted and focus is lost. The fallback must either contain a focusable element (Retry) that takes focus, or the screen must move focus to the next rail.
- **Boundaries only catch render errors.** Errors in event handlers, timers, and async code (a failed fetch) are not caught; handle those with state, using the shared loading / error / offline states (see [One set of states](../docs-2/03-app-structure.md#one-set-of-states)).
- **Log with context**: device model, platform version, app version, screen, rail id. TV-only crashes are often tied to one model or engine.
- **Retry should reset the boundary** (`resetErrorBoundary` or `resetKeys`), not just refetch.
- Error boundaries are class components under the hood; `react-error-boundary` gives you a function-friendly wrapper.

## Explain it out loud

**How do you avoid platform checks spreading through the code?**

By giving every platform difference exactly one place to live and keeping feature code platform-free. Real platform API differences (the Samsung AVPlay player, key codes, lifecycle events, LG services) go behind an adapter: one shared interface and one file per platform, picked by the bundler from the file extension (`.tizen.web.tsx` before `.web.tsx` before `.tsx`). Screens import the adapter and never ask which platform they're on. Small differences like a number or a flag go in a per-platform config module chosen the same way. Engine differences between old and new TVs are handled with feature detection and polyfills inside the module that needs the feature, not with device or version checks, because those guess wrong after firmware updates and on models you didn't list.

Then enforce it: a lint rule that forbids importing the platform module outside the `platform/` and adapter folders, so a quick `if (isTizen)` in a screen fails CI. Adding a new platform then means adding files, not editing every screen.
