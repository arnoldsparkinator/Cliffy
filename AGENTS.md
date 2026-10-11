# AGENTS.md — Cliffy

Project brief for AI coding agents (and me). Keep this updated as the app evolves.

## What this is
**Cliffy** — a single-page bad-weather watch for a **40-mile radius of Tunbridge Wells, UK**.
It flags heavy rain / flood risk, high winds, and freezing temperatures, is searched by **UK postcode**,
lists bad-weather events by postcode (last 14 days + next 14 days), and shows **live Environment Agency
flood warnings**. Units are UK: °C, mm, mph.

## Where it lives
- Local working copy: `C:\Users\Alex\cliffy`
- GitHub (source of truth): https://github.com/arnoldsparkinator/Cliffy
- Live site (GitHub Pages): https://arnoldsparkinator.github.io/Cliffy/
- GitHub account/handle: `arnoldsparkinator`

## Architecture
- Everything is in `index.html` — HTML + CSS + vanilla JS, no build step, no dependencies.
- Fixed service centre: Tunbridge Wells `lat 51.132, lon 0.263`, radius `40` miles (haversine).
- Data sources (all free, no API key, CORS `*`):
  - **Postcodes**: `https://api.postcodes.io` — `/postcodes/{pc}` (lookup), `/postcodes/{partial}/autocomplete`,
    `/postcodes?lon=&lat=` (reverse / nearest).
  - **Weather**: Open-Meteo `https://api.open-meteo.com/v1/forecast` with
    `daily=precipitation_sum,wind_gusts_10m_max,temperature_2m_min,weather_code,precipitation_probability_max`,
    `current=temperature_2m,wind_gusts_10m,weather_code,precipitation`,
    `past_days=14&forecast_days=15&timezone=Europe/London&wind_speed_unit=mph`
    (use `wind_speed_unit=mph` so gusts come back in mph).
  - **Floods (live)**: Environment Agency `https://environment.data.gov.uk/flood-monitoring/id/floods`
    with `?lat=..&long=..&dist=64&min-severity=3`. **`dist` is in KILOMETRES** (64 km ≈ 40 miles).
    `min-severity=3` returns in-force warnings only (severityLevel 1 = severe, 2 = warning, 3 = alert; 4 = no longer in force).
    Item fields: `severity`, `severityLevel`, `floodAreaID`, `floodArea.{label,county,riverOrSea}`, `description`, `message`, `timeRaised`.
- Saved postcodes, alert thresholds and the selected postcode are stored in `localStorage` (key `cliffy.v1`).
- `.nojekyll` is present so GitHub Pages serves the file as-is.

## Conventions / gotchas
- Tap handling on mobile uses the `onTap()` helper (pointer events) — do NOT switch these
  back to plain `click`, or iOS swallows the first tap while the keyboard is open.
- "Today" is derived from the API's location-local date (`data.current.time`), not the browser date.
- postcodes.io lookups need the space stripped (e.g. `TN1 1BT` -> `TN11BT`); display with the space.
- No comments in code unless truly necessary. Keep it a single file.

## Editable alert thresholds (persisted)
Defaults: rainfall alert `10` mm / severe `25` mm; gusts alert `40` mph / severe `55` mph;
min temp freeze `0` °C / hard freeze `-3` °C. Editable in the sidebar.

## Deploy
Push to `main`; GitHub Pages rebuilds automatically in ~1 minute. The URL never changes.
```
.\backup.ps1 "your message"     # stages, commits, pushes
```
(Or: `git add -A; git commit -m "..."; git push`.)

## Backup / recovery
- Only **pushed** code exists in the cloud — run `backup.ps1` after each work session.
- opencode chat history lives only on this machine (`C:\Users\Alex\.local\share\opencode`).
  Export important sessions with `/export` and store the Markdown in the private
  `opencode-notes` repo.
- Keep GitHub 2FA recovery codes off this laptop.
