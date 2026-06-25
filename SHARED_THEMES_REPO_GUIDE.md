# Populating `masselink/MasselGUARD-themes` — instructions for an agent

> **Hand this whole file to a fresh Claude session** whose working directory is a clone of
> <https://github.com/masselink/MasselGUARD-themes>. It is a complete, self-contained spec
> for laying out the shared-themes repository so the MasselGUARD app can download themes
> **without** pulling a repository `.zip` — each theme's files are fetched individually over
> raw HTTPS. Fonts and images are first-class parts of a theme.

---

## 1. Why not a zip

The app must be able to list the available themes, show them, and download only the files of
the theme(s) the user picks — fetching each file directly from
`raw.githubusercontent.com`. That keeps downloads small and the repo browsable/inspectable on
GitHub (no opaque archive). So the repo is driven by a **manifest** (`index.json`) plus one
folder per theme.

## 2. Repository layout (flat per theme)

```
/                         (repo root)
  index.json              ← manifest; the app reads this FIRST
  README.md               ← human description (not downloaded by the app)
  shared-themes/
    <theme-id>/           ← one folder per theme; <theme-id> is lowercase, no spaces
      theme.json          ← REQUIRED — the theme definition
      logo.png            ← optional image asset
      bg.jpg              ← optional background image
      tray-connected.ico  ← optional tray icons
      tray-disconnected.ico
      MyFont-Regular.ttf  ← optional font(s)
      MyFont-Bold.ttf
      preview-dark.png    ← optional screenshot for a future gallery (not installed)
```

**Keep every file a theme needs FLAT in that theme's folder** (no sub-folders for fonts or
images). Two reasons:

- The app's font loader only scans the theme folder's top level for `*.ttf`/`*.otf`.
- The app rejects any asset path that is rooted, UNC, or contains `..` (it must stay inside
  the theme folder), so reference assets by plain relative filename only.

## 3. `index.json` (the manifest)

Root-level file the app downloads first. Schema:

```json
{
  "schema": 1,
  "themes": [
    {
      "id": "neon-night",
      "name": "Neon Night",
      "description": "Cyberpunk dual-variant theme with a neon accent.",
      "author": "masselink",
      "tags": ["dark", "light", "neon", "cyberpunk", "high-contrast"],
      "path": "shared-themes/neon-night",
      "files": [
        "theme.json",
        "logo.png",
        "bg.jpg",
        "Orbitron-Regular.ttf",
        "Orbitron-Bold.ttf"
      ],
      "previewDark":  "shared-themes/neon-night/preview-dark.png",
      "previewLight": "shared-themes/neon-night/preview-light.png"
    }
  ]
}
```

Field rules:

| Field | Required | Meaning |
|---|---|---|
| `schema` | yes | Manifest version. Use `1`. |
| `themes[]` | yes | One entry per theme. |
| `id` | yes | Install folder name. Lowercase, `[a-z0-9-]`, no spaces. Must match the folder under `shared-themes/`. |
| `name` | yes | Display name (any language/script). |
| `description` | no | One-line description (shown on the browser card). |
| `author` | no | Credit (shown as "by …"). |
| `tags[]` | recommended | Short lowercase keywords for the browser's filter chips + search (e.g. `dark`, `light`, `minimal`, `high-contrast`, `colorful`, `translucent`). Include `dark` and/or `light` to say which variants the theme ships. |
| `path` | yes | Theme folder relative to repo root, e.g. `shared-themes/neon-night`. |
| `files[]` | yes | **Every file the app must download**, relative to `path`. MUST include `theme.json` and **every font and image the theme references**. If it isn't listed here it won't be downloaded. |
| `previewDark` | recommended | Repo-root-relative screenshot of the theme's **dark** variant, shown on the browser card. Not installed into the theme folder. |
| `previewLight` | recommended | Repo-root-relative screenshot of the theme's **light** variant. The browser's Dark/Light header toggle switches between the two; if only one is supplied it is shown for both. |

**The `files[]` list is the contract.** The downloader does not crawl the folder — it fetches
exactly the files you list. Forgetting a font or background image means that theme installs
broken. Double-check every filename referenced inside `theme.json` appears in `files[]`.
(Preview images are referenced by `previewDark`/`previewLight` and are *not* part of
`files[]` — they are displayed in the browser, never installed.)

### Preview images (for the in-app Theme Browser)

The app shows a **wide theme browser** pop-up: a card per theme with its preview image, name,
description and tags, a text search, clickable tag chips, and a **Dark / Light toggle in the
header** that flips every card between its `previewDark` and `previewLight` image.

- Provide both `previewDark` and `previewLight` where the theme meaningfully differs between
  variants (most do). A real screenshot of MasselGUARD using the theme is ideal.
- The card preview area is ~272 × 158 px and scales the image with `UniformToFill`. Supply a
  **16:10-ish landscape PNG/JPG** (e.g. 640 × 400 or larger); keep each well under a few
  hundred KB so the grid loads quickly.
- Keep preview files inside the theme's folder (e.g. `shared-themes/<id>/preview-dark.png`) so the
  repo stays tidy, and reference them by their repo-root-relative path.

## 4. How the app consumes it (so you populate it correctly)

This is **implemented** in the app (Settings → Appearance → **Browse…**). For repo URL
`https://github.com/masselink/MasselGUARD-themes` it tries the default branch `main` (then
`master`) and:

1. `GET https://raw.githubusercontent.com/masselink/MasselGUARD-themes/main/index.json`
2. Renders a card per theme — `previewDark`/`previewLight` image (Dark/Light header toggle),
   `name`, `author`, `description`, `tags` chips + search.
3. On **Install**, for each `f` in `t.files`:
   `GET …/main/{t.path}/{f}` → save to `<install>/shared_themes/{t.id}/{f}`.

So: **filenames are case-sensitive** on raw.githubusercontent.com, paths are forward-slash,
and the default branch should be `main` (`master` also works). The app validates `id` and each
file path and will skip anything that tries to escape `shared_themes/<id>/`.

## 5. The `theme.json` format

Themes use MasselGUARD's **unified dual-variant** format: structural settings at the root,
colours split into `"dark"` and `"light"` sections. The full key reference lives in the app
repo at [`shared_themes/THEME_INFO.md`](https://github.com/masselink/MasselGUARD/blob/main/shared_themes/THEME_INFO.md);
copy concrete, fully-populated examples from the shipped themes
(`shared_themes/blueongrey/theme.json`, `highcontrast/theme.json`, `glass/theme.json`).

Minimum a good theme should set:

- Root: `name`, `creator`, `description`, `appName`, `type` (`"dark"`/`"light"`),
  `fontFamily`, `fontSize`, `cornerRadius`, the title/status-bar fields,
  `windowOpacity`, `panelOpacity`, and (if used) `logo`/`backgroundImage`/`appIcon`/
  `trayIconConnected`/`trayIconDisconnected` as **plain relative filenames**.
- Both `dark` and `light` with the full colour set (window/surface/card/border/accent,
  text primary+muted, success/danger/highlight, error(+bg)/warning(+bg), list hover/selected,
  log timestamp, and the four `colorTray*` keys). Either variant may be omitted — the app
  HSL-inverts the other — but shipping both gives predictable results.

### Fonts

- Put the `.ttf`/`.otf` file(s) at the theme folder root and list them in `files[]`.
- Set `"fontFamily"` to the font's **family name** (e.g. `"Orbitron"`), not the filename.
  The app loads folder fonts as private fonts (no install needed) and falls back to an
  installed font of the same name.
- Only ship fonts whose license permits redistribution (SIL OFL, Apache, etc.).

### Images

- `logo`, `backgroundImage`, `appIcon`, `trayIconConnected`, `trayIconDisconnected` are
  plain relative filenames pointing at files in the same folder (listed in `files[]`).
- Icons: `.ico` preferred for tray/app icon; `.png` works for logo/background.

## 6. Step-by-step for the populating session

1. Create `shared-themes/<id>/` for each theme; author `theme.json` (start from a shipped example,
   keep all asset references relative and flat).
2. Add any font/image files into the same folder.
3. Add 1–2 preview screenshots per theme (`preview-dark.png`, `preview-light.png`) in the
   theme folder.
4. Build/refresh `index.json` so each theme entry lists **every** file in its folder that the
   app needs (`theme.json` + all referenced fonts/images) plus `tags`, `previewDark` and
   `previewLight`.
5. Validate (see checklist) and commit to the **`main`** branch.

## 7. Validation checklist (must all pass)

- [ ] `index.json` at repo root, `schema: 1`, parses.
- [ ] Each `themes[].path` folder exists and contains `theme.json`.
- [ ] Each `files[]` entry exists (case-sensitive) and includes `theme.json`.
- [ ] Every font/image named in a `theme.json` is in that theme's `files[]`.
- [ ] `previewDark`/`previewLight` paths (when set) exist and point inside the theme folder;
      they are **not** in `files[]`.
- [ ] `tags` are short lowercase keywords; include `dark`/`light` to advertise variants.
- [ ] No `..`, rooted, or UNC asset paths anywhere; `id` is a safe single folder name
      (no separators) and matches the folder, and is unique.
- [ ] Fonts are redistributable; `fontFamily` is the family name, font files are at folder root.
- [ ] Default branch is `main`.

---

### App side — already implemented

`Services/ThemeDownloadService.cs` reads `index.json` (`FetchManifestAsync`), fetches preview
images (`FetchRawAsync`) and installs a theme's `files[]` over raw HTTPS into
`shared_themes/<id>/` (`InstallThemeAsync`); `Views/ThemeBrowserWindow` is the wide pop-up.
A legacy whole-repo `.zip` download path is retained only for arbitrary `.zip`/repo URLs typed
manually — the manifest path above is the supported route for this repository.
