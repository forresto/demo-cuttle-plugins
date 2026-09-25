# Map Picker

A Cuttle Customizer Plugin that turns an OpenStreetMap area into laser-ready street map vectors.

## Live plugin

https://forresto.github.io/demo-cuttle-plugins/map-picker/

## Source

[index.html](./index.html)

## What it does

1. Search for a place (↓ to move into the results, Enter to choose).
2. Pan and scroll the Leaflet map to frame the area inside the inset frame.
3. Set the aspect ratio and detail level.
4. When the map settles, vector tiles are simplified into an SVG drawn over the map in place.
5. **Save** calls `cuttle.setValueAndClose(...)`.

## Cuttle value

```js
{
  query: "Portland, Oregon",
  location: "Portland, Multnomah County, Oregon, United States",
  latitude: 45.5202,
  longitude: -122.6742,
  areaWidthKm: 3.22,     // frame width
  areaDisplayUnit: "mi", // "mi" | "km" – display only
  aspect: [4, 3],
  detail: "light",       // "light" | "medium" | "high"
  _previewText: "Portland, Multnomah County, Oregon, United States",
  _previewImage: "data:image/svg+xml,..."
}
```

`_previewImage` is the output: a unitless SVG (longer side 180) that the template scales. Colors follow Cuttle conventions: red cut border, blue score (local roads, buildings), black engrave (major roads, water). `latitude`, `longitude`, and `areaWidthKm` restore the framing.

## Dependencies

- Cuttle Customizer Plugin SDK v1
- Leaflet 1.9.4
- OpenStreetMap raster tiles, Shortbread vector tiles, and Nominatim search

## Notes

Tile requests are kept polite: at most 24 vector tiles per render, 4 at a time, nothing new once a render is stale, and a 100-tile cache. Search runs only on Enter or the Search button.
