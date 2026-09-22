# Cuttle Customizer Plugin Examples

Small, self-contained examples of HTML Input Plugins for [Cuttle](https://cuttle.xyz/).

The examples are intended to be useful as both **working plugins** and **recipes for AI assistants** that are helping someone build a new Cuttle plugin.

## Live examples

| Example | Description | Live |
|---|---|---|
| [OSM Place Picker](./example/osm-place-picker/) | Search OpenStreetMap, choose a place, then fine-tune its coordinates with a draggable Leaflet marker. | [Open](https://forresto.github.io/test-cuttle-plugins/example/osm-place-picker/) |

## Using this repository with ChatGPT

Give an AI assistant this repository URL:

https://github.com/forresto/test-cuttle-plugins

Then describe the Cuttle parameter and UI you want.

A useful workflow is:

1. Inspect the closest example.
2. Preserve the Cuttle Customizer SDK protocol.
3. Adapt the value schema to the requested parameter.
4. Keep the plugin self-contained in one HTML file unless there is a good reason not to.
5. Add a self-documenting README beside the example.
6. Publish it through GitHub Pages.
7. Return the live HTTPS URL.

Each example should document:
- its live URL
- its source file
- the value written to Cuttle
- how the UI behaves
- external dependencies
- important implementation decisions

## Plugin SDK

Plugins use the Cuttle Customizer Plugin SDK:

`https://cuttle.xyz/editor/customizer/v1.js`

The basic lifecycle is:

```js
cuttle.ready(value => {
  // populate the UI from the existing parameter value
});

cuttle.setValueAndClose(value);
```

## Hosting

The repository is published with GitHub Pages using the workflow in `.github/workflows/pages.yml`.

Static plugin files can therefore be used directly as HTTPS Cuttle Input Plugin URLs.

## Repository layout

```
/
├── README.md
├── LICENSE
├── example/
│   └── osm-place-picker/
│       ├── index.html
│       └── README.md
└── .github/
    └── workflows/
        └── pages.yml
```
