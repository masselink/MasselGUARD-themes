# MasselGUARD-themes

Shared themes for the [MasselGUARD](https://github.com/masselink/MasselGUARD) WireGuard VPN
client with automations.

The MasselGUARD app browses and installs themes directly from this repository over raw HTTPS —
no repository `.zip` download. The app reads [`index.json`](index.json) first, then fetches only
the files of the theme(s) the user installs. See [`THEME_EXAMPLE.md`](THEME_EXAMPLE.md) for the
full field reference, manifest schema, and a copy-paste template.

## What a theme controls

A theme is a JSON file (plus optional images/fonts) that drives almost everything about how the
app looks — not just colours:

- **Colours** — 21 semantic keys (window background, surface, card, border, accent, text, tray,
  etc.), set independently for a `dark` and a `light` variant. Either variant can be omitted and
  the app derives it automatically, but shipping both gives predictable results. Three of the
  keys (list hover, tray hover, highlight) support an alpha byte for translucency.
- **Typography** — a body `fontFamily` (private `.ttf`/`.otf` files in the folder work too, no
  install needed) plus an optional distinct `headerFontFamily` for the title bar and section
  headers. A single base `fontSize` drives four scaled tiers used throughout the app (Tiny /
  Small / body / Header).
- **Window chrome** — corner radius, title-bar height and visibility of its icon/app-name, window
  and panel opacity (translucency, used by the Glass theme).
- **Status bar** — visibility of the whole bar and its WiFi/tunnel indicators.
- **Assets** — logo, app icon, tray icons (connected/disconnected), and a background image, each
  **dark/light aware** (two independent images per asset). All are optional; the app falls back
  to a built-in shield icon and no background when unset.
- **Custom variables** — an open `variables` map for anything not covered above.

Every field is documented, with a full working example, in [`THEME_EXAMPLE.md`](THEME_EXAMPLE.md).

## Layout

```
index.json            manifest the app reads first
themes/
  <theme-id>/
    <theme-id>-theme.json   the theme definition (required, e.g. neon-night-theme.json)
    <assets…>               optional fonts/images, flat in the folder
```

## Available themes

| ID | Name | Variants | Description |
|---|---|---|---|
| `blueongrey` | Blue on grey | dark · light | Flat grey theme with a blue accent and sharp corners. No distractions. |
| `dracula` | Dracula | dark · light | The iconic Dracula palette — purple, pink and cyan on deep night blue. |
| `flat` | Flat | dark · light | Bold flat-UI colours with sharp corners and a crisp industrial sans. |
| `forest-mist` | Forest Mist | dark · light | Calm forest greens with a soft mossy accent. |
| `glass` | Glass | dark · light | Frosted-glass look — a translucent window with see-through panels and soft rounded corners. |
| `highcontrast` | High Contrast | dark · light | True high-contrast theme for low-vision use — black/white with strong, vivid accents. |
| `low-contrast` | Low Contrast | dark · light | Soft, muted greys with gentle contrast and a serif face — easy on tired eyes. |
| `monokai` | Monokai | dark · light | The classic Monokai code palette — vivid green, pink and cyan on warm charcoal. |
| `neon-night` | Neon Night | dark · light | Cyberpunk theme with a vivid neon accent. |
| `nord-aurora` | Nord Aurora | dark · light | Cool arctic palette with frost blues and aurora accents. |
| `red-velvet` | Red Velvet | dark · light | Plush deep-crimson velvet with gold accents and a soft napped backdrop. |
| `solarized` | Solarized | dark · light | Warm, low-glare palette tuned for long sessions. |
| `unifi` | UniFi | dark · light | UniFi-inspired blue console look. Not affiliated with or endorsed by Ubiquiti; UniFi is a trademark of Ubiquiti Inc. |

## Installing a theme

In the app: **Settings → Appearance → Browse…**, pick a theme, and click **Install**. If a theme
you already installed later changes here, the button switches to **Update** the next time you
open the browser.

## Adding or updating a theme

1. Create or edit `themes/<id>/<id>-theme.json` (start from one of the themes here; keep every
   asset reference a plain relative filename, flat in the folder).
2. Add any fonts/images into the same folder.
3. Add or update its entry in [`index.json`](index.json) listing **every** file the app must
   download (the `<id>-theme.json` file plus all referenced fonts/images), its `tags`, and
   preview images (`previewDark` / `previewLight`, relative to the theme folder).
4. **Bump that entry's `version`** to the current ISO 8601 UTC timestamp (e.g.
   `2026-07-15T13:57:17Z`) — this is how the app knows an already-installed theme has an update
   waiting.
5. Validate the JSON and commit to `main`.

The manifest schema, the full `theme.json` key reference, and a complete copy-paste template are
all in [`THEME_EXAMPLE.md`](THEME_EXAMPLE.md).
