# OSM Place Picker

A Cuttle Customizer Plugin that turns a place search into geographic coordinates.

## Live plugin

https://forresto.github.io/test-cuttle-plugins/example/osm-place-picker/

## Source

[index.html](./index.html)

## What it does

1. The user types a place name.
2. Pressing **Search** queries OpenStreetMap Nominatim.
3. The user chooses a result.
4. A Leaflet map appears with a draggable marker.
5. Dragging the marker updates the coordinates.
6. Clicking the map moves the marker.
7. **Save** calls `cuttle.setValueAndClose(...)`.

When the plugin opens again, `latitude` and `longitude` restore the map and marker.

## Cuttle value

The plugin writes:

```js
{
  query: "Helsinki",

  latitude: 60.169897,
  longitude: 24.938416,

  direction1: "N",
  degrees1: 60,
  minsec1: "10' 12"",

  direction2: "E",
  degrees2: 24,
  minsec2: "56' 18""
}
```

**Important:** `latitude` and `longitude` are the authoritative values. The DMS fields are derived output only. DMS is never parsed back into coordinates.

`minsec1` and `minsec2` use whole seconds. `degrees1` and `degrees2` are numeric values; the UI adds the `°` symbol when displaying coordinates.

## Dependencies

- Cuttle Customizer Plugin SDK v1
- Leaflet 1.9.4
- OpenStreetMap map tiles
- OpenStreetMap Nominatim search

## Notes for adapting this example

Keep the Cuttle SDK lifecycle:

```js
cuttle.ready(value => {
  // restore state
});

cuttle.setValueAndClose(value);
```

For geographic data, keep full-precision decimal coordinates as the source of truth and derive any human-readable representation from them.

The public Nominatim service is used here for a simple user-triggered search. Do not turn this into keystroke-by-keystroke autocomplete against the public service.
