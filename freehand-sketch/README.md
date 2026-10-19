# Freehand Sketch

A Cuttle Parameter Plugin for drawing freehand strokes using [perfect-freehand](https://github.com/steveruizok/perfect-freehand).

## Live plugin

After publishing this folder with GitHub Pages:

https://forresto.github.io/demo-cuttle-plugins/freehand-sketch/index.html

## What it does

- Large responsive canvas supporting mouse, touch, and stylus input.
- Uses perfect-freehand to turn sampled points (including pressure) into filled stroke outlines.
- Stroke size, smoothing, and pressure-thinning controls.
- Undo the last stroke or clear the canvas.
- Saves original point arrays and SVG output.
- Does not union, merge, or boolean-process the outlines.

## Saved Cuttle value

```js
{
  strokes: [
    [[x, y, pressure], ...], // one array per stroke; logical 1000 × 750 coordinates
    ...
  ],
  size: 12,
  smoothing: 0.5,
  thinning: 0.5,
  svg: "<svg ...><path d=\"M...Z M...Z\" ... /></svg>",
  _image: "data:image/svg+xml,...",
  _text: "2 freehand strokes"
}
```

The SVG has one `<path>` element. Its `d` attribute concatenates the outline data for each stroke, so there are multiple subpaths within one compound path. No union is performed; Cuttle can handle that downstream.

The SVG uses `width="200mm"`, `height="150mm"`, and `viewBox="0 0 1000 750"`. Coordinates are stored in the canvas's logical coordinate system, independent of its displayed CSS size.

When reopened, the plugin restores the point arrays and drawing settings, then regenerates the outlines.

## Dependencies

- Cuttle Parameter Plugin SDK v1
- perfect-freehand 1.2.2 (ES module from unpkg)
