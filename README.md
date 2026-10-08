> ⚠️ pre-release preview, won't work quite yet 😉

# Cuttle Parameter Plugin Examples

Small, self-contained examples of Parameter Plugins for [Cuttle](https://cuttle.xyz/).

The examples are both **working plugins** and **recipes for AI assistants** that are helping someone build a new Cuttle plugin.

## Live examples

| Example                                     | Description                                                                                   | Live                                                                                 |
| ------------------------------------------- | --------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------ |
| [Coordinates Picker](./coordinates-picker/) | Search OpenStreetMap, choose a place, then fine-tune its coordinates with a draggable marker. | [Open](https://forresto.github.io/demo-cuttle-plugins/coordinates-picker/index.html) |
| [Image to Palette](./image-to-palette/)     | Choose an image and extract a palette of colors for use as a Cuttle parameter.                | [Open](https://forresto.github.io/demo-cuttle-plugins/image-to-palette/index.html)   |
| [Map Picker](./map-picker/)                 | Frame an OpenStreetMap area and save it as laser-ready SVG vectors.                           | [Open](https://forresto.github.io/demo-cuttle-plugins/map-picker/index.html)         |

## Example plugin

[example-plugin/](./example-plugin/) is a complete, self-contained example with the Cuttle SDK lifecycle and a visual reference for plugin UI: colors, layout, and native controls. It outputs a simple star as score lines, following the SVG conventions below. The design is a reference, not a template; copy the parts a plugin uses.

## Plugin SDK

Plugins use the Cuttle Parameter Plugin SDK. The basic lifecycle is:

```html
<title>My Plugin</title>
<!-- ⚠️ URL might change before launch. -->
<script src="https://cuttle.xyz/parameter-plugin/v1.js"></script>
<script>
  render(); // draw with defaults first; don't wait for ready

  cuttle.ready((value = {}) => {
    // populate the UI from the saved value, then render()
  });

  saveButton.onclick = () => cuttle.setValueAndClose(value);
</script>
```

- `value` is `undefined` until the parameter has been saved once. Default it, and handle missing fields gracefully.
- The value must be plain JSON: strings, numbers, booleans, `null`, arrays, and plain objects. A field set to `undefined` or a function makes Cuttle ignore the whole value and the plugin stays open, with only a console warning. A `Map`, `Date`, or `Blob` won't survive being saved.
- Call `cuttle.setValueAndClose(value)` once, with the final value. The value is stored in the Cuttle project.

### Testing without Cuttle

Opened in its own browser tab, `cuttle.ready` is called right away with `undefined`, and `cuttle.setValueAndClose(value)` downloads `value._image` as a file, or the value as JSON if there is no `_image`. Open that file to check the plugin's output.

Inside an iframe that isn't Cuttle, such as an AI tool's preview pane, `cuttle.ready` is never called. That is why the UI should render with defaults first.

### Magic values

Cuttle treats two fields of the value specially:

- **`value._text`** — optional one-line summary used as the parameter's preview text in Cuttle.
- **`value._image`** — optional preview image, as a data URL (JPEG, PNG, or SVG). An SVG is also importable as vector artwork.

#### Returning an SVG as `_image`

Give the SVG `width` and `height` with a unit, `mm` or `in`, and use the same numbers in the `viewBox`:

```svg
<svg xmlns="http://www.w3.org/2000/svg" width="200mm" height="150mm" viewBox="0 0 200 150">
  …
</svg>
```

Every coordinate and stroke width in the SVG is then in that unit; here, millimeters. Without a unit on `width` and `height`, the physical size is ambiguous.

Encode the SVG string as a data URL with `encodeURIComponent`. The `xmlns` attribute is required.

```js
value._image = "data:image/svg+xml," + encodeURIComponent(svg);
```

Cuttle imports geometry only:

- `path`, `rect`, `circle`, `ellipse`, `line`, `polyline`, `polygon`, `g`, and `use` are imported, with their `transform`.
- `<text>`, `<image>`, clip paths, and gradients are skipped. `opacity`, `display`, `visibility`, and dashes are ignored, so hidden elements still import.
- Write `fill="none"` on every stroked shape. SVG's default fill is black, which imports as an engrave fill.

#### Laser operations: colors and stroke widths

Cuttle reads the laser operation from color and stroke width.

| Operation | SVG                                                                   |
| --------- | --------------------------------------------------------------------- |
| Cut       | red `#ff0000` hairline stroke, no fill                                |
| Score     | blue `#0000ff` hairline stroke, no fill                               |
| Engrave   | black `#000000` stroke at least 0.02 in wide, or black `#000000` fill |

Stroke widths depend on the unit on the SVG's `width` and `height`:

| Unit | Hairline (cut and score) | Minimum engrave stroke |
| ---- | ------------------------ | ---------------------- |
| `in` | 0.01                     | 0.02                   |
| `mm` | 0.254                    | 0.508                  |
| `px` | 0.96                     | 1.92                   |
| `pt` | 0.72                     | 1.44                   |

Any stroke thinner than the engrave minimum imports as a hairline.

An SVG in millimeters with each operation:

```svg
<svg xmlns="http://www.w3.org/2000/svg" width="200mm" height="150mm" viewBox="0 0 200 150">
  <rect x="0" y="0" width="200" height="150" fill="none" stroke="#ff0000" stroke-width="0.254"/> <!-- cut -->
  <path d="M25 75H175" fill="none" stroke="#0000ff" stroke-width="0.254"/> <!-- score -->
  <path d="M25 100H175" fill="none" stroke="#000000" stroke-width="1"/>    <!-- engrave line, 1 mm -->
  <circle cx="100" cy="40" r="12" fill="#000000"/>                         <!-- engrave fill -->
</svg>
```

## Reading plugin values

The plugin writes a JavaScript object to the Cuttle parameter. Its fields can be used in a normal component's expressions, such as `parameterValue.latitude`, or in a code component. This code component example renders the SVG from `_image`:

```js
if (!parameterValue) return;
const { _image } = parameterValue;
if (!_image) return;

const svgRenderer = getSVGFromURL(_image);
if (!svgRenderer) return;

return svgRenderer.render();
```

`parameterValue` stands for the name of the component's parameter. `getSVGFromURL` returns `undefined` while the SVG is loading, so the code returns early until it is ready.

## Libraries

Examples load third-party libraries from version-pinned CDNs, such as:

```html
<script src="https://unpkg.com/leaflet@1.9.4/dist/leaflet.js"></script>
```

## Hosting

Cuttle Parameter Plugins are ordinary web pages loaded from an HTTPS URL. For example,

`https://forresto.github.io/demo-cuttle-plugins/image-to-palette/index.html`

Options for hosting:

- [GitHub Pages](https://docs.github.com/en/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site) (free)
- [ChatGPT Sites](https://help.openai.com/en/articles/20001339-creating-and-using-chatgpt-sites) (with Work or Codex paid plans)

## Using this repository with AI

Give a coding agent or AI assistant this repository URL:

`https://github.com/forresto/demo-cuttle-plugins`

Then describe the Cuttle parameter and UI you want.

The agent should follow this workflow:

1. Inspect the closest example.
2. Preserve the Cuttle Parameter Plugin SDK lifecycle.
3. Adapt the value schema to the requested parameter.
4. Keep the plugin self-contained in one HTML file.
5. Add a README.md beside the plugin's index.html: live URL, the value written to Cuttle, how the UI behaves, and external dependencies.
6. Publish the plugin where it can be accessed with a stable HTTPS URL.
7. Return the live URL. If publishing to a stable HTTPS URL isn't possible, return the HTML code and a hint about hosting.
