# DigiLekha remote config

Display settings for the DigiLekha app: app icon choice, the downloaded splash and the bottom tabs.
The app fetches `app-config.json` in the background and applies it on the **next** launch.

**Public repo: never put secrets here.**

## Editing

1. Edit `app-config.json` and push to `feature/dev` (the app reads this branch for now; never push to `main` directly).
2. GitHub's raw cache can take ~5 minutes to serve the new file.
3. Open the app (refresh happens in the background), then close and reopen it to see the change.

## How the splash works

```
native launch screen  →  splash  →  app
(plain navy, in the app;    (animated; from this repo once
 can't change from here)     downloaded, else built into the app)
```

| Situation on the phone | Splash shown |
| --- | --- |
| First install / nothing downloaded yet | The app's **built-in** animated splash (same as `splash/default.svg`) |
| Splash from this repo downloaded | **This repo's** splash (`splash.image`) |
| `splash.enabled: false` | **No splash**: launch screen, then the app |

- `nativeSplash` only controls how long the plain launch screen stays (`enabled: false` = as short as possible). iOS / Android always show it briefly while the app starts.
- Changes apply on the **second** launch after GitHub serves the new file (launch 1 downloads, launch 2 shows). GitHub's raw cache can add ~5 minutes.

## How the app icon works

The app ships with three icons: `default` (navy), `dark` and `diwali`. New artwork needs an app release; this file only picks between them.

| `appIcon` in this file | Phone in light mode | Phone in dark mode |
| --- | --- | --- |
| `"active": "default"`, no schedule for today | **default** icon | **dark** icon |
| `"active": "diwali"` (or `"dark"`) | that icon | that icon |
| A `schedule` entry whose dates include today | that entry's icon (wins over `active`) | that entry's icon |

Example: Diwali icon for a week, then back to normal:

```json
"appIcon": {
  "active": "default",
  "schedule": [{ "value": "diwali", "from": "2027-10-25", "to": "2027-11-01" }]
}
```

- The icon changes on the **next launch** after the app has the new file (so usually the 2nd launch after a change here, like the splash).
- **iOS** changes it when the app opens and shows its own message: *"You have changed the icon for DigiLekha"* (Apple requires it). **Android** changes it when the app goes to the background; nothing is shown.
- Light / dark follows the phone's appearance at that launch.

## Rules the app enforces

| Field | Allowed | If invalid |
| --- | --- | --- |
| `version` | `1` | Whole file ignored |
| `appIcon.active`, `schedule[].value` | `default`, `diwali`, `dark` | Keeps the current icon / entry dropped |
| `schedule[].from` / `to` | `YYYY-MM-DD`, `from <= to`, inclusive | Entry dropped |
| `splash.enabled` | `true` / `false`; `false` = **no splash at all** (not even the app's built-in one) | Treated as `true` |
| `splash.id` | Any text; change it to make the app download the image again | Keeps the current splash |
| `splash.image` | `https://` URL to a PNG, JPEG, WebP or SVG (e.g. a raw URL to `splash/*.svg` in this repo) | Keeps the current splash |
| `splash.background` | Hex colour, e.g. `#0B1F3A` (shown around the image) | Keeps the current colour |
| `splash.durationMs` | 500–5000 | Clamped |
| `nativeSplash.enabled` | `true` / `false`; `false` = hide the native launch screen as soon as the splash (or app) is ready | Treated as `true` |
| `nativeSplash.durationMs` | 0–3000: minimum time the native launch screen stays up after the app starts (only when `enabled` is `true`); the splash appears after it | Clamped |
| `tabs[].id` | `home`, `ledger`, `profile`, `reports` (no duplicates) | Tab skipped |
| `tabs[].icon` | `home`, `swap`, `person`, `stats` | Generic icon |
| number of tabs | 2–5 valid tabs | Default tabs |

## Splash images

- `splash/*.svg`: **animated** splashes (logo builds up, pixels fly out of the corner). Built by `python3 src/make_animated.py`; keep `durationMs` ≥ 1800 so the animation finishes.
- `splash/default.svg` is also built into the app (`src/features/splash/bundled-splash.svg` in digi-lekha). If you change it here, copy it there too so first-install users see the same splash.
- `splash/*.png`: static versions (portrait, 1170×2532). Any splash file: PNG, JPEG, WebP or SVG, max 3 MB.
- `src/*.svg`: their sources. Render with `rsvg-convert -w 1170 -h 2532 src/<name>.svg -o splash/<name>.png`.
- To switch splash: point `splash.image` at the new file **and** change `splash.id` (the app only downloads again when the id changes).
