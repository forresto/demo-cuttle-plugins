# Map Picker

A Cuttle Customizer Plugin that turns an OpenStreetMap area into laser-ready street map vectors.

## Live plugin

https://forresto.github.io/demo-cuttle-plugins/map-picker/

## Source

[index.html](./index.html)

## What it does

1. Search for a place (↓ to move into the results, Enter to choose).
2. Pan and scroll the Leaflet map to frame the area inside the inset frame.
3. Set the aspect ratio and detail level, and choose a laser operation for each layer.
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
  layers: {              // "cut" | "score" | "engrave" | "fill" | "off"
    majorRoads: "engrave",
    minorRoads: "score",
    paths: "off",
    railways: "off",
    water: "engrave",
    waterways: "score",
    buildings: "score",
    parks: "off",
    border: "cut"
  },
  _previewText: "Portland, Multnomah County, Oregon, United States",
  _previewImage: "data:image/svg+xml,..."
}
```

`_previewImage` is the output: a unitless SVG (longer side 180) that the template scales. `latitude`, `longitude`, and `areaWidthKm` restore the framing. Values saved without `layers` use the defaults shown above.

## Layers

Each layer comes from the Shortbread vector tiles and gets one operation, in Cuttle's colors:

| Operation | SVG |
|---|---|
| Cut | red `#ff0000` stroke |
| Score | blue `#0000ff` stroke |
| Engrave | black `#000000` stroke, as a thick line at the layer's width |
| Engrave fill | black `#000000` fill, no stroke (areas only: water, buildings, parks & woods) |
| Off | omitted |

| Layer | Source |
|---|---|
| Major roads | `streets` motorway to tertiary, including links |
| Local roads | `streets` unclassified, residential, living street, service, pedestrian, busway |
| Paths | `streets` footway, path, cycleway, steps, track |
| Railways | `streets` rail, tram, light rail, subway, etc., except tunnels |
| Water | `ocean` and `water_polygons` |
| Rivers & streams | `water_lines` centerlines, stopped at the shore of water areas (drawn whole when Water is off) |
| Buildings | `buildings`, only when the area is small enough for zoom 14 tiles |
| Parks & woods | `land` park, forest, wood, grass, meadow, golf course |
| Border | the frame rectangle (no fill) |

Runways, taxiways, and unknown street kinds are dropped.

Areas are merged first: the pieces of a lake, park, or building from neighboring tiles become one shape, with holes and islands kept. Then they are drawn differently depending on the operation. A stroke operation draws open outlines with no edges along the frame, so a cropped lake does not repeat the border. Shapes fully inside the frame stay closed. **Engrave fill** clips shapes closed at the frame, since those edges bound the filled region. All rings of a layer go in one path with the nonzero fill rule, so islands and courtyards stay unfilled.

## Dependencies

- Cuttle Customizer Plugin SDK v1
- Leaflet 1.9.4
- [Clipper2](https://github.com/AngusJohnson/Clipper2) via `clipper2-ts` 2.0.1-18 (Boost license) for merging, clipping, and simplifying
- OpenStreetMap raster tiles, Shortbread vector tiles, and Nominatim search

## Notes

Tile requests are kept polite: at most 24 vector tiles per render, 4 at a time, nothing new once a render is stale, and a 100-tile cache. Search runs only on Enter or the Search button.
