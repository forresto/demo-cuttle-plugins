> ⚠️ pre-release preview, won't work quite yet 😉

# Cuttle Parameter Plugin Examples

Small, self-contained examples of Parameter Plugins for [Cuttle](https://cuttle.xyz/).

The examples are both **working plugins** and **recipes for AI assistants** that are helping someone build a new Cuttle plugin.

Start a chat with this repository as the reference: [ChatGPT](https://chatgpt.com/?q=Help%20me%20design%20and%20build%20a%20Cuttle%20Parameter%20Plugin%20hosted%20html%20page.%20Here%20is%20a%20reference%3A%20https%3A%2F%2Fgithub.com%2Fforresto%2Fdemo-cuttle-plugins) · [Claude](https://claude.ai/new?q=Help%20me%20design%20and%20build%20a%20Cuttle%20Parameter%20Plugin%20hosted%20html%20page.%20Here%20is%20a%20reference%3A%20https%3A%2F%2Fgithub.com%2Fforresto%2Fdemo-cuttle-plugins) · [Gemini](https://share.gemini.google/AHC303T1Rqwk). Or give a coding agent `https://github.com/forresto/demo-cuttle-plugins`, then describe the Cuttle parameter and UI you want.

## Examples

- [example-plugin](./example-plugin/): the starting point. A complete Cuttle SDK lifecycle and a visual reference for plugin UI (colors, layout, native controls). Outputs a star as score lines. A reference, not a template; copy the parts a plugin uses.
- [Coordinates Picker](./coordinates-picker/) ([live](https://forresto.github.io/demo-cuttle-plugins/coordinates-picker/index.html)): search OpenStreetMap, choose a place, fine-tune coordinates with a draggable marker.
- [Freehand Sketch](./freehand-sketch/) ([live](https://forresto.github.io/demo-cuttle-plugins/freehand-sketch/index.html)): doodle with "perfect-freehand" strokes.
- [Image to Palette](./image-to-palette/) ([live](https://forresto.github.io/demo-cuttle-plugins/image-to-palette/index.html)): extract a color palette from an image.
- [Map Picker](./map-picker/) ([live](https://forresto.github.io/demo-cuttle-plugins/map-picker/index.html)): frame an OpenStreetMap area and save it as laser-ready SVG vectors.

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

- **`value._text`**: optional one-line summary used as the parameter's preview text in Cuttle.
- **`value._image`**: optional image output, as a data URL (JPEG, PNG, or SVG). Cuttle shows it as the parameter's preview, and an SVG is importable as vector artwork. This data URL is the only copy of the SVG in the value; don't also save the SVG, or its markup, under another key.

#### Returning an SVG as `_image`

- Give the SVG `width` and `height` in `mm`, and use the same numbers in the `viewBox`. Every coordinate and stroke width is then in millimeters.
- The `xmlns` attribute is required.
- Encode the SVG string with `encodeURIComponent`:

  ```js
  value._image = "data:image/svg+xml," + encodeURIComponent(svg);
  ```

- Cuttle imports geometry only. `path`, `rect`, `circle`, `ellipse`, `line`, `polyline`, `polygon`, `g`, and `use` are imported, with their `transform`. `<text>`, `<image>`, clip paths, and gradients are skipped. `opacity`, `display`, `visibility`, and dashes are ignored, so hidden elements still import.
- Write `fill="none"` on every stroked shape. SVG's default fill is black, which imports as an engrave fill.

#### Laser operations: colors and stroke widths

Cuttle reads the laser operation from color and stroke width (in mm):

| Operation              | SVG                                                           |
| ---------------------- | ------------------------------------------------------------- |
| Cut                    | red `#ff0000` hairline stroke (`0.254`), no fill              |
| Score (vector-engrave) | blue `#0000ff` hairline stroke (`0.254`), no fill             |
| Engrave                | black `#000000` stroke at least `0.508` wide, or black `#000000` fill |

Any stroke thinner than the engrave minimum imports as a hairline.

An SVG with each operation:

```svg
<svg xmlns="http://www.w3.org/2000/svg" width="200mm" height="150mm" viewBox="0 0 200 150">
  <rect x="0" y="0" width="200" height="150" fill="none" stroke="#ff0000" stroke-width="0.254"/> <!-- cut -->
  <path d="M25 75H175" fill="none" stroke="#0000ff" stroke-width="0.254"/> <!-- score -->
  <path d="M25 100H175" fill="none" stroke="#000000" stroke-width="1"/>    <!-- engrave line, 1 mm -->
  <circle cx="100" cy="40" r="12" fill="#000000"/>                         <!-- engrave fill -->
</svg>
```

## Using the value in Cuttle (for humans)

Once a plugin has saved its value, the parameter holds that JavaScript object. Its fields can be used in a normal component's expressions, such as `parameterValue.latitude`, or in a code component. This code component renders the SVG from `_image`:

```js
if (!parameterValue) return;
const { _image } = parameterValue;
if (!_image) return;

const svgRenderer = getSVGFromURL(_image);
if (!svgRenderer) return;

return svgRenderer.render();
```

`parameterValue` stands for the name of the component's parameter. `getSVGFromURL` returns `undefined` while the SVG is loading, so the code returns early until it is ready.

## Building a plugin

A plugin is an ordinary web page loaded from an HTTPS URL, for example `https://forresto.github.io/demo-cuttle-plugins/image-to-palette/index.html`. [GitHub Pages](https://docs.github.com/en/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site) (free) and [ChatGPT Sites](https://help.openai.com/en/articles/20001339-creating-and-using-chatgpt-sites) (Work or Codex paid plans) both work.

The agent should follow this workflow:

1. Inspect the closest example.
2. Preserve the Cuttle Parameter Plugin SDK lifecycle.
3. Adapt the value schema to the requested parameter.
4. Keep the plugin self-contained in one HTML file. Load third-party libraries from version-pinned CDNs, such as `https://unpkg.com/leaflet@1.9.4/dist/leaflet.js`.
5. Add a README.md beside the plugin's index.html: live URL, the value written to Cuttle, how the UI behaves, and external dependencies.
6. Publish the plugin where it can be accessed with a stable HTTPS URL.
7. Return the live URL. If publishing to a stable HTTPS URL isn't possible, return the HTML code and a hint about hosting.
