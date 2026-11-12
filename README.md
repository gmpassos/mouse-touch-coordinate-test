# Mouse & Touch Coordinate Precision Test

A standalone, single-page browser experiment that logs the coordinates
reported by mouse, pointer and touch events, and classifies each value as
**integer** or **fractional**.

It exists to answer a practical question: *does this browser, on this
device, actually deliver sub-pixel (fractional) input coordinates?* The page
shows the raw values exactly as the browser reports them — nothing is
rounded, scaled or synthesized.

**Live demo:** https://gmpassos.github.io/mouse-touch-coordinate-test/

## Supported event types

| Family  | Events |
|---------|--------|
| Mouse   | `mousemove`, `mousedown`, `mouseup`, `click`, `dblclick`, `contextmenu`, `mouseenter`, `mouseleave`, `mouseover`, `mouseout` |
| Pointer | `pointermove`, `pointerdown`, `pointerup`, `pointercancel`, `pointerenter`, `pointerleave`, `pointerover`, `pointerout` |
| Touch   | `touchstart`, `touchmove`, `touchend`, `touchcancel` |

Events are not deduplicated: a single physical action (e.g. a tap) may log a
touch event, a pointer event and compatibility mouse events, so their
coordinates can be compared side by side.

## Inspected properties

For `MouseEvent` / `PointerEvent`:

- `clientX` / `clientY` — relative to the viewport
- `pageX` / `pageY` — relative to the document
- `screenX` / `screenY` — relative to the screen
- `offsetX` / `offsetY` — relative to the target element's padding edge
- `pointerType`, `pointerId` (pointer events)
- `button`, `buttons`

For `TouchEvent`:

- `touches.length`
- For each entry of `changedTouches`: `identifier`, `clientX/Y`,
  `pageX/Y`, `screenX/Y`

`changedTouches` is used (rather than `touches`) so that lifted or cancelled
touch points are still reported on `touchend` and `touchcancel`.

Every entry also records a timestamp, the event constructor name and
`window.devicePixelRatio`.

History is bounded to the latest 500 events (newest first).

## Integer vs. fractional classification

Each coordinate is printed with its raw value and a label:

- `integer` — `Number.isInteger(value)` is true (e.g. `120`)
- `fractional` — a finite number with a fractional part (e.g. `120.5`)
- `not a number` — the property was missing or non-numeric in this browser

Fractional coordinates are permitted by the browser APIs — the CSSOM View
and Pointer Events specifications define these properties as `double` — but
whether you actually see them depends on the input hardware (mouse, trackpad,
pen, touchscreen), how the browser processes and quantizes input, the page
zoom level, and the operating system's display scaling. Many configurations
report only integers; others (e.g. high-DPI screens, non-100% zoom, touch or
pen input) commonly produce fractional values.

## How to test

Open the page and interact inside the grey area. Watch the classification
labels in the event history.

1. **Mouse movement** — move the mouse slowly across the area; inspect
   `mousemove` and `pointermove` entries.
2. **Clicks** — single click, double click and right click (`contextmenu`);
   compare `mousedown`/`pointerdown`/`click` coordinates.
3. **Touch** — on a touch device, tap, drag and use multiple fingers; check
   `touchstart`/`touchmove`/`touchend` alongside the pointer events with
   `Pointer type: touch`.
4. **Browser zoom** — repeat the above at different zoom levels (e.g. 90%,
   110%, 125%, 150%) using Ctrl/Cmd `+` / `-`. Note `Device pixel ratio`.
5. **Display scaling** — change the OS display scaling (e.g. 100%, 125%,
   150%, 200% on Windows; "Scaled" resolutions on macOS; fractional scaling
   on Linux) and repeat.

## Browser compatibility

Works in all modern browsers: Chrome, Edge, Firefox and Safari (desktop and
mobile). Pointer Events are supported in all current versions. Touch Events
are only dispatched on touch-capable devices (and, in desktop Firefox and
Safari, may not be exposed at all), in which case only mouse and pointer
events are logged.

## Running locally

No dependencies, build tools or server are required. Either:

- open `index.html` directly in a browser, or
- serve the folder with any static server, e.g.
  `python3 -m http.server 8000` and visit `http://localhost:8000/`.

## License

[MIT](LICENSE)
