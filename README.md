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
| `tabs[].id` | `home`, `ledger`, `profile`, `reports` (no duplicates) | Tab skipped |
| `tabs[].icon` | `home`, `swap`, `person`, `stats` | Generic icon |
| number of tabs | 2–5 valid tabs | Default tabs |

## Splash images

- `splash/*.png`: images the app downloads (portrait, 1170×2532; PNG, JPEG or WebP, max 3 MB).
- `src/*.svg`: their sources. Render with `rsvg-convert -w 1170 -h 2532 src/<name>.svg -o splash/<name>.png`.
- To switch splash: point `splash.image` at the new file **and** change `splash.id` (the app only downloads again when the id changes).
