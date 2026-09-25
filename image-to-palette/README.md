# Image to Palette

Cuttle Customizer plugin that extracts a color palette from an uploaded image.

## Live

https://forresto.github.io/demo-cuttle-plugins/image-to-palette/

## Value

The plugin saves a value like:

```js
{
  count: 6,
  colors: ["#ffffff", "..."],
  _previewImage: "data:image/svg+xml,..."
}
```

The source image is used only while the plugin is open and is not saved in the parameter value.

## Implementation

The image is resized to a maximum width of 400 px, then colors are extracted from the resulting pixel data using an adaptation of Google's [Art Palette](https://github.com/googleartsculture/art-palette/tree/master/palette-extraction) implementation.

The number of colors defaults to 6 and can be changed before saving.

The implementation is largely based on [Palette-Based Photo Recoloring (Chang and al. 2015)](http://gfx.cs.princeton.edu/pubs/Chang_2015_PPR/chang2015-palette_small.pdf), section 3.2.

See the [plugin source](./index.html) for the complete self-contained implementation.
