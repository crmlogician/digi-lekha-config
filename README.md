# DigiLekha remote config

Display settings for the DigiLekha app: app icon choice, the downloaded splash and the bottom tabs.
The app fetches `app-config.json` in the background and applies it on the **next** launch.

**Public repo: never put secrets here.**

## Editing

1. Edit `app-config.json` and push to `feature/dev` (the app reads this branch for now; never push to `main` directly).
2. GitHub's raw cache can take ~5 minutes to serve the new file.
3. Open the app (refresh happens in the background), then close and reopen it to see the change.

## Rules the app enforces

| Field | Allowed | If invalid |
| --- | --- | --- |
| `version` | `1` | Whole file ignored |
| `appIcon.active`, `schedule[].value` | `default`, `diwali`, `dark` | Keeps the current icon / entry dropped |
| `schedule[].from` / `to` | `YYYY-MM-DD`, `from <= to`, inclusive | Entry dropped |
| `splash.enabled` | `true` / `false` | Treated as `true` |
| `splash.id` | Any text; change it to make the app download the image again | Keeps the current splash |
| `splash.image` | `https://` URL (e.g. a raw URL to `splash/*.png` in this repo) | Keeps the current splash |
| `splash.background` | Hex colour, e.g. `#FAF7F0` | Keeps the current colour |
| `splash.durationMs` | 500–5000 | Clamped |
| `nativeSplash.enabled` | `true` / `false`; `false` hides the built-in splash as soon as the app or the downloaded splash is ready | Treated as `true` |
| `nativeSplash.durationMs` | 0–3000: minimum time the built-in splash stays up after the app starts; the downloaded splash appears after it | Clamped |
| `tabs[].id` | `home`, `ledger`, `profile`, `reports` (no duplicates) | Tab skipped |
| `tabs[].icon` | `home`, `swap`, `person`, `stats` | Generic icon |
| number of tabs | 2–5 valid tabs | Default tabs |

## Splash images

- `splash/*.svg`: **animated** splashes (logo builds up, pixels fly out of the corner). Built by `python3 src/make_animated.py`; keep `durationMs` ≥ 1800 so the animation finishes.
- `splash/*.png`: images the app downloads (portrait, 1170×2532; PNG, JPEG, WebP or SVG, max 3 MB). Static fallbacks.
- `src/*.svg`: their sources. Render with `rsvg-convert -w 1170 -h 2532 src/<name>.svg -o splash/<name>.png`.
- To switch splash: point `splash.image` at the new file **and** change `splash.id` (the app only downloads again when the id changes).
