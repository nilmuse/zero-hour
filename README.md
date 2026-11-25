# zero-hour

Display a countdown to a date and time with a sprite-based `<countdown-timer>` element.

[![npm](https://img.shields.io/npm/v/zero-hour.svg?colorB=brightgreen)](https://www.npmjs.com/package/zero-hour)
[![NPM Downloads](https://img.shields.io/npm/dm/zero-hour.svg?style=flat)](https://www.npmjs.com/package/zero-hour)

[Demo](https://codepen.io/ux-ui/pen/MYyMpPw)

## Installation

```bash
npm install zero-hour
```

## Quick start

Provide a digit sprite at `/sprites/digits.webp`: ten equal-width frames in one horizontal row, ordered `0` through `9`. Optionally provide a colon image at `/sprites/sep.webp`. These images are your assets; the package does not include them.

Add this markup:

```html
<countdown-timer
  digits-url="/sprites/digits.webp"
  separator-url="/sprites/sep.webp"
  units="m:s"
  autostart="false"
></countdown-timer>
```

In a browser entry point, set a target one minute from now and start the timer after the markup is available:

```ts
import { initCountdownTimers } from 'zero-hour';

const timer = document.querySelector('countdown-timer')!;
const target = new Date(Date.now() + 60_000).toISOString();

timer.setAttribute('date', target.slice(0, 10));
timer.setAttribute('time', target.slice(11, 19));
timer.setAttribute('utc', '+00:00');

initCountdownTimers({
  onDone: (element) => {
    element.classList.add('is-done');
    console.log('Countdown finished');
  },
});

timer.start();
```

Importing the package registers the element and supplies its Shadow DOM styles. `autostart="false"` keeps it stopped while the example sets the target.

For a fixed deadline, set `date`, `time`, and `utc` in markup and omit `autostart="false"`.

## API

- Importing `zero-hour` registers `<countdown-timer>` as a side effect. Use this entry in a browser; it requires DOM globals.
- `initCountdownTimers(options?)` finds matching elements, optionally attaches a completion callback, applies styles, and returns the elements.
- `zeroHourCssText` exports the packaged minified CSS text.

### Initialization options

| Option | Type | Default | Description |
| --- | --- | --- | --- |
| `selector` | `string` | `'countdown-timer'` | Selector used to find elements. |
| `onDone` | `(element: HTMLElement) => void` | `undefined` | Called once per element for this helper call. Called immediately if that timer is already complete. |
| `stylesheet` | `CSSStyleSheet \| string \| null` | `undefined` | Styles to apply inside each timer's shadow root. Omitted, `null`, and empty-string values leave existing styles unchanged. |

TypeScript signature:

```ts
declare function initCountdownTimers(options?: {
  selector?: string;
  onDone?: (element: HTMLElement) => void;
  stylesheet?: CSSStyleSheet | string | null;
}): HTMLElement[];
```

### Attributes

| Attribute | Type | Default | Description |
| --- | --- | --- | --- |
| `digits-url` | `string` | Absent | Required to start the timer and display graphical digits. |
| `separator-url` | `string` | `null` | Separator image URL. Omit it to hide separators. |
| `autostart` | `boolean` | `true` | Start on connection. An empty attribute enables it; use `autostart="false"` to disable it. |
| `date` | `YYYY-MM-DD` | Absent | Target date. Without a valid date, the display resolves to zero. |
| `time` | `HH:MM[:SS]` | `'00:00:00'` | Target time. |
| `utc` | `string` | `'UTC+0'` | Target UTC offset, such as `UTC+03:00`, `+03:00`, or `UTC-5`. |
| `units` | `string` | `'d:h:m:s'` | Visible groups, selected from `d`, `h`, `m`, and `s`, separated by `:`. |
| `mode` | `'static' \| 'scroll'` | `'static'` | Use `scroll` for a rolling digit transition. |

The page's `date`, `time`, `utc`, and `units` query parameters can override the matching attributes. Valid date, time, and UTC query values take precedence; invalid values fall back to the attributes.

### Methods

Call these on a `<countdown-timer>` element after importing the package:

- `start()` stops any previous run and starts the countdown. It does nothing without `digits-url`.
- `stop()` clears the scheduled tick.
- `reset()` clears the completion flag and restarts when `autostart` is enabled; otherwise it updates the static display.
- `isRunning()` returns whether the timer has an active start time and a scheduled tick.
- `isDone()` returns whether the target moment has passed. It returns `false` without a digit URL or valid target date.
- `adoptStylesheet(sheet)` applies a `CSSStyleSheet` to the shadow root.
- `adoptStyles(styles)` accepts CSS text, a `CSSStyleSheet`, `null`, or `undefined`. Text uses a constructable stylesheet with a `<style>` fallback; `null` or `undefined` restores the default styles.

### Events

| Event | When | Payload (`event.detail`) |
| --- | --- | --- |
| `done` | Once when the displayed countdown reaches zero; it can fire again after `reset()`. | `null` |

Listen on the timer itself; `done` does not bubble. The `onDone` initialization helper notifies only once per element for that helper call, even if a reset produces another event.

## Styling

Each timer installs the bundled styles inside its shadow root. To apply a replacement stylesheet to existing timers:

```ts
import { initCountdownTimers, zeroHourCssText } from 'zero-hour';

initCountdownTimers({ stylesheet: zeroHourCssText });
```

You can also supply CSS text from your build pipeline. For example, in a Vite project with a stylesheet at `./assets/scss/components/zero-hour.scss`:

```ts
import { initCountdownTimers } from 'zero-hour';
import zeroHourCss from './assets/scss/components/zero-hour.scss?raw';

initCountdownTimers({ stylesheet: zeroHourCss });
```

The CSS sidecar is available for tooling that needs a stylesheet file:

```ts
import 'zero-hour/zero-hour.css';
```

Importing the sidecar into the document does not replace styles inside a timer's shadow root.

## Display limits

- Ticks are scheduled on second boundaries.
- Days use two digits and are capped at `99`.
- `units` selects visible groups; hidden day groups do not convert days into extra hours.
- Digit sprites must have ten horizontal frames ordered `0`–`9`; the frame index is the digit value.
- Separators are hidden when `separator-url` is absent.

## License

MIT
