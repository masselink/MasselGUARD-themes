# MasselGUARD-themes

Shared themes for the [MasselGUARD](https://github.com/masselink/MasselGUARD) WireGuard VPN
client with automations.

The MasselGUARD app browses and installs themes directly from this repository over raw HTTPS —
no repository `.zip` download. The app reads [`index.json`](index.json) first, then fetches only
the files of the theme(s) the user installs. See
[`SHARED_THEMES_REPO_GUIDE.md`](SHARED_THEMES_REPO_GUIDE.md) for the full spec.

## Layout

```
index.json            manifest the app reads first
themes/
  <theme-id>/
    theme.json        the theme definition (required)
    <assets…>         optional fonts/images, flat in the folder
```

## Available themes

| ID | Name | Variants | Description |
|---|---|---|---|
| `neon-night` | Neon Night | dark · light | Cyberpunk theme with a vivid neon accent. |
| `nord-aurora` | Nord Aurora | dark · light | Cool arctic palette with frost blues. |
| `solarized` | Solarized | dark · light | Warm, low-glare palette for long sessions. |
| `forest-mist` | Forest Mist | dark · light | Calm forest greens with a mossy accent. |

## Installing a theme

In the app: **Settings → Appearance → Browse…**, pick a theme, and click **Install**.

## Adding a theme

1. Create `themes/<id>/theme.json` (start from one of the themes here; keep every asset
   reference a plain relative filename, flat in the folder).
2. Add any fonts/images into the same folder.
3. Add an entry to [`index.json`](index.json) listing **every** file the app must download
   (`theme.json` plus all referenced fonts/images) and its `tags`.
4. Optionally add `preview-dark.png` / `preview-light.png` to the folder and reference them via
   `previewDark` / `previewLight` (these are shown in the browser, not installed).
5. Commit to `main`.

The theme file format and full key reference are documented in
[`SHARED_THEMES_REPO_GUIDE.md`](SHARED_THEMES_REPO_GUIDE.md).
