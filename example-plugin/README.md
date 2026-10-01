# Example Plugin

A minimal Cuttle Customizer Plugin example and visual reference for plugin UI.

The directory includes a Cuttle SDK lifecycle example, colors, layout, native
controls, and a responsive iframe test. The design is a reference, not a
template. Copy the parts a plugin uses rather than adopting every style.

## Form and output patterns

- Use a 16px base for labels and controls, with smaller readable helper text.
- Arrange labels and controls with CSS grid; stack rows in narrow iframes.
- Text fields use white backgrounds, visible neutral borders, useful
  placeholders, and blue focus outlines. Numeric controls and selects stay gray.
- Compact actions such as Search use the neutral default button style.
- Save is the only `.primary` button, with a full-width blue treatment.
- Normal informational notes are subdued, unboxed read-only text. Warnings and
  errors keep distinct treatments near the relevant input.
- An **output raft** groups a concise representation of the value with Save:
  palette swatches, coordinates, or a generated-map summary. Keep credits outside.

## Files

- [index.html](./index.html) — plugin page and
  visual design reference
- [responsive-test.html](./responsive-test.html) — the same page shown at several iframe sizes
