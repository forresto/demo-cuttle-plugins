# Freehand Sketch

A Cuttle Parameter Plugin for drawing freehand strokes using [perfect-freehand](https://github.com/steveruizok/perfect-freehand).

## Live plugin

After publishing this folder with GitHub Pages:

https://forresto.github.io/demo-cuttle-plugins/freehand-sketch/index.html

## What it does

- Large responsive canvas supporting mouse, touch, and stylus input.
- Uses perfect-freehand to turn sampled points (including pressure) into filled stroke outlines.
- Input points are thinned to a minimum spacing based on stroke size, so stylus and mouse produce similar point density.
- Stroke size, smoothing, and pressure-thinning controls.
- Eraser tool (button, `E` key, or stylus eraser) that cuts strokes where it passes.
- Undo the last change or clear the canvas.
- Strokes may extend past the page; the canvas zooms out (around the page centre) to keep them visible and editable.
- Saves original point arrays and an SVG image.
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
  _image: "data:image/svg+xml,...", // <svg ...><path d="M...Z M...Z" ... /></svg>
  _text: "2 strokes"
}
```

The SVG has one `<path>` element. Its `d` attribute concatenates the outline data for each stroke, so there are multiple subpaths within one compound path. No union is performed; Cuttle handles that.

Coordinates are stored in the page's logical 1000 × 750 coordinate system (200 × 150mm), independent of the canvas's displayed CSS size, and may fall outside it.

Cuttle centres the image, so the SVG is framed by a zoom factor `z` around the page centre rather than a tight bounding box: `width="200z mm"`, `height="150z mm"`, and a 4:3 `viewBox` of `1000z × 750z` centred on (500, 375). `z` is the smallest factor that contains every stroke outline, so a small sketch gives a smaller image and off-page ink gives a larger one (an empty sketch uses `z = 1`).

When reopened, the plugin restores the point arrays and drawing settings, then regenerates the outlines.

## Dependencies

- Cuttle Parameter Plugin SDK v1
- perfect-freehand 1.2.2 (ES module from unpkg)
