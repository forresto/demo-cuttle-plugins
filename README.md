> ⚠️ pre-release preview, won't work quite yet 😉

# Cuttle Parameter Plugin Examples

Self-contained HTML Parameter Plugins for [Cuttle](https://cuttle.xyz/), usable as both working plugins and recipes for AI assistants.

Start a chat with this repo as context: [ChatGPT](https://chatgpt.com/?q=Help%20me%20design%20and%20build%20a%20Cuttle%20Parameter%20Plugin.%20Here%20is%20a%20reference%3A%20https%3A%2F%2Fgithub.com%2Fforresto%2Fdemo-cuttle-plugins) · [Claude](https://claude.ai/new?q=Help%20me%20design%20and%20build%20a%20Cuttle%20Parameter%20Plugin.%20Here%20is%20a%20reference%3A%20https%3A%2F%2Fgithub.com%2Fforresto%2Fdemo-cuttle-plugins) · [Gemini](https://share.gemini.google/AHC303T1Rqwk).

## Examples

- [example-plugin](./example-plugin/): SDK lifecycle and UI style reference (layout, colors, native controls). Outputs a star as score lines.
- [Coordinates Picker](./coordinates-picker/) ([demo](https://forresto.github.io/demo-cuttle-plugins/coordinates-picker/index.html)): search OpenStreetMap and drag a marker to pick coordinates.
- [Freehand Sketch](./freehand-sketch/) ([demo](https://forresto.github.io/demo-cuttle-plugins/freehand-sketch/index.html)): draw with `perfect-freehand` strokes.
- [Image to Palette](./image-to-palette/) ([demo](https://forresto.github.io/demo-cuttle-plugins/image-to-palette/index.html)): extract a color palette from an image.
- [Map Picker](./map-picker/) ([demo](https://forresto.github.io/demo-cuttle-plugins/map-picker/index.html)): frame an OpenStreetMap area as laser-ready SVG vectors.

## How a plugin works

A plugin is a single self-contained HTML file pasted directly into a Cuttle parameter's HTML editor (no hosting needed).

```html
<!doctype html>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>My Plugin</title>
<script src="https://cuttle.xyz/parameter-plugin/v1.js"></script>
<script type="module">
  render(); // render defaults immediately; don't wait for cuttle.ready

  cuttle.ready((value = {}) => {
    // populate UI from saved value (may be undefined or missing fields), then render()
  });

  saveButton.onclick = () => cuttle.setValueAndClose(value);
</script>
```

- **Plain JSON only:** `value` must contain only strings, numbers, booleans, `null`, arrays, and plain objects. Any `undefined` field or function makes Cuttle ignore the whole save; `Map`, `Date`, and `Blob` won't survive.
- **Render defaults first:** `cuttle.ready` isn't called inside non-Cuttle preview iframes. Opened standalone in a tab, `ready` fires with `undefined` and `setValueAndClose` downloads `_image` (or JSON).
- **Sandboxed iframe (`srcdoc`):** runs with an opaque `null` origin—no `localStorage`, `sessionStorage`, IndexedDB, or cookies, and `fetch` sends `Origin: null` (public CORS APIs and CDNs work).
- **Dependencies:** load third-party libraries from version-pinned CDNs (e.g. `https://esm.sh/perfect-freehand@1.2.2` in `<script type="module">`).
- **Output:** return the complete, single-file HTML for the user to paste into Cuttle.

## Magic fields

- **`value._text`**: optional one-line preview label in Cuttle.
- **`value._image`**: optional data URL (SVG, PNG, or JPEG) shown as the parameter preview; SVGs are importable as vector artwork. Store the SVG only in `_image`, not duplicated under another key.

### SVG `_image` rules

- Set `xmlns`, `width`/`height` in `mm`, and matching `viewBox` numbers so coordinates and stroke widths are in mm.
- Encode with `"data:image/svg+xml," + encodeURIComponent(svg)`.
- Geometry only: `path`, `rect`, `circle`, `ellipse`, `line`, `polyline`, `polygon`, `g`, `use` (with `transform`). `<text>`, `<image>`, clip paths, and gradients are skipped. `opacity`, `display`, `visibility`, and dashes are ignored (hidden elements still import).
- Always set `fill="none"` on stroked shapes (default SVG fill is black = engrave).
- Laser operations are read from color and stroke width (in mm; strokes thinner than `0.508` import as hairlines):

```svg
<svg xmlns="http://www.w3.org/2000/svg" width="200mm" height="150mm" viewBox="0 0 200 150">
  <rect x="0" y="0" width="200" height="150" fill="none" stroke="#ff0000" stroke-width="0.254"/> <!-- cut (red hairline) -->
  <path d="M25 75H175" fill="none" stroke="#0000ff" stroke-width="0.254"/> <!-- score (blue hairline) -->
  <path d="M25 100H175" fill="none" stroke="#000000" stroke-width="1"/>    <!-- engrave stroke (>= 0.508) -->
  <circle cx="100" cy="40" r="12" fill="#000000"/>                         <!-- engrave fill -->
</svg>
```

## Using the value in Cuttle

Read saved fields in expressions (e.g. `parameterValue.latitude`) or render `_image` in a code component:

```js
if (!parameterValue || !parameterValue._image) return;
const svg = getSVGFromURL(parameterValue._image);
if (svg) return svg.render();
```
