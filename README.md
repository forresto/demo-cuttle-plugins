> ⚠️ pre-release preview, won't work quite yet 😉

# Cuttle Parameter Plugin Examples

Parameter Plugins give a [Cuttle](https://cuttle.xyz/) parameter its own interface: a self-contained HTML page that edits the parameter's value. This repo is a reference for writing them, by hand or with an AI assistant.

Start a chat with this repo as context: [ChatGPT](https://chatgpt.com/?q=Help%20me%20design%20and%20build%20a%20Cuttle%20Parameter%20Plugin.%20Here%20is%20a%20reference%3A%20https%3A%2F%2Fgithub.com%2Fforresto%2Fdemo-cuttle-plugins) · [Claude](https://claude.ai/new?q=Help%20me%20design%20and%20build%20a%20Cuttle%20Parameter%20Plugin.%20Here%20is%20a%20reference%3A%20https%3A%2F%2Fgithub.com%2Fforresto%2Fdemo-cuttle-plugins) · [Gemini](https://share.gemini.google/AHC303T1Rqwk).

## Making a plugin

1. In Cuttle, set a parameter's type to Parameter Plugin and choose **Edit Plugin…**.
2. Paste the plugin's HTML into the HTML editor. It is saved in the project, so there's nothing to host.
3. Use the plugin to set the value, then use the value in your project.

More full examples (Map Picker, Coordinates Picker, Freehand Sketch) are in Cuttle's Examples panel. Drag one into a project, then open its HTML with **Edit Plugin…**.

## Examples

- [example-plugin](./example-plugin/): a minimal plugin with one input, an SVG preview, and a Save button. Outputs a star as score lines.
- [Image to Palette](./image-to-palette/) ([demo](https://forresto.github.io/demo-cuttle-plugins/image-to-palette/index.html)): extract a color palette from an image.

## How a plugin works

A plugin is one HTML page that loads the plugin SDK:

```html
<!doctype html>
<meta charset="utf-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<script src="https://cuttle.xyz/parameter-plugin/v1.js"></script>
<link rel="stylesheet" href="https://cuttle.xyz/parameter-plugin/v1.css" />

<!-- UI here -->

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
- **Title:** Cuttle titles the plugin window with the parameter's name; `<title>` is only used for standalone downloads.
- **Output:** return the complete, single-file HTML for the user to paste into Cuttle.

## Styles (`v1.css`)

The stylesheet is optional. It gives the plugin the Cuttle look, and sits in a `cuttle` cascade layer so any plugin CSS overrides it. Leave out the link to style from scratch.

- Styled directly: `body`, `h3`, `small`, `output`, `input`, `select`, `textarea`, `button`, `summary`, `details`.
- `.layout` (preview beside controls, stacked when narrow) · `.preview` (white card for a canvas, SVG, or map; `aria-busy="true"` dims it) · `.empty` (placeholder text in the preview)
- `.controls` (column of panels) · `.panel` (grey group, usually starting with an `h3`) · `.row` (label and control) · `.pair` (controls side by side in a row cell)
- `.seg` (segmented radio control) · `.actions` (equal-width buttons) · `.primary` (blue main button, usually Save) · `.note` (status text, hidden when empty; add `.error`)
- Variables: `--bg`, `--panel`, `--field`, `--field-h`, `--field-border`, `--ink`, `--ink2`, `--line`, `--blue`, `--blue-h`, `--error`, `--error-ink`, `--mono`.

See [example-plugin](./example-plugin/index.html) for the markup.

## Magic fields

- **`value._text`**: optional one-line preview label in Cuttle.
- **`value._image`**: optional data URL (SVG, PNG, or JPEG) shown as the parameter preview; SVGs are importable as vector artwork. Store the SVG only in `_image`, not duplicated under another key.

### SVG `_image` rules

- Set `xmlns`, `width`/`height` in `mm`, and matching `viewBox` numbers so coordinates and stroke widths are in mm.
- Encode with `cuttle.svgDataUrl(svg)`.
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

Read saved fields in expressions (e.g. `parameterValue.points`) or render `_image` in a code component:

```js
if (!parameterValue || !parameterValue._image) return;
const svg = getSVGFromURL(parameterValue._image);
if (svg) return svg.render();
```
