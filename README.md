# Relay — PS-08 "One City, One Journey"

One Metro + BRTS + AMTS itinerary for an Ahmedabad trip: every leg, every transfer walk, the waits,
and one combined fare estimate, with the assumptions visible. Built from `relay_hackathon_plan.html`.

> **All network data is an illustrative sample.** Stop coordinates, timings, interchange walks and fare
> bands are synthetic fixtures, not official operator data. The UI labels them as such.
> Vehicle positions are *simulated* from the sample timetable; nothing is live and no ticketing is implied.

## Run it (VS Code)

1. `File → Open Workspace from File…` → `relay.code-workspace` (or just open this folder).
2. Terminal: `npm start` → http://localhost:5173  (Node 18+, **no npm install needed**; zero dependencies).
   Or press **Ctrl/Cmd+Shift+B** (task "Relay: start app").
3. Click **Load the demo trip**: Vastral → Narendra Modi Stadium, 08:30 (Metro → BRTS → AMTS, two transfer walks).
4. Press ▶ under the map to watch the simulated vehicles, including the one you board (ringed).

Opening `public/index.html` directly (file://) also works with the schematic map.

Other commands: `npm test` (10 engine tests), `npm run evaluate` (sample-trip results in the terminal),
`npm run build:data` (regenerate `public/data.js` + CSVs from `data/network.json`).

## Google Maps (optional)

1. In Google Cloud, enable **Maps JavaScript API** and **Directions API**; create an API key and restrict it to
   the HTTP referrer `http://localhost:5173/*`.
2. `cp .env.example .env` and set `GOOGLE_MAPS_API_KEY=...` (the server injects it via `/config.js`; keep `.env` out of git).
3. Restart `npm start`. You now get the Google basemap with route polylines, simulated vehicle markers, and a
   **Verify walks with Google** button that replaces each sample walking time with Google's walking duration
   and re-plans (those walks are then badged "Verified with Google Maps").

No key, no network, or a rejected key → the app automatically falls back to the built-in schematic map.

## What it does (mapped to the plan)

| Plan step | Where |
|---|---|
| Enter trip (fixture locations, no geocoding) | `public/app.js` form, `data/network.json → places` |
| Build candidate legs, connect with walking edges | `src/engine.js → enumeratePaths` |
| Rank (fastest / fewest transfers / lowest fare) | `src/engine.js → RANK` |
| Fare estimate per operator + combined total | `src/engine.js → simulate`, `data.fares` |
| Explain each transfer (walk minutes, status, step-free "not verified") | itinerary list in the UI |
| Missing data shown as a gap, never zero | unmeasured walk `M_OHC ↔ B_DDW` → "Needs data check" option |
| Fare-split bonus (by fare paid vs by km), labelled a policy proposal | "Operator fare split" panel |
| Evidence the approach works | **Pilot results** tab, `npm run evaluate` |

Waits come from the sample timetable (headway + service window), so the "wait" you see is for a specific simulated vehicle.

## Sample dataset (`data/`)

| File | Content |
|---|---|
| `network.json` | **Source of truth**: 34 stops, 7 lines (1 Metro, 1 BRTS, 5 AMTS), walk links, 8 places, fare bands |
| `stops.csv`, `line_stops.csv`, `walk_links.csv`, `fare_bands.csv` | Flat exports of the above, for review/editing in a spreadsheet |
| `sample_trips.csv` | 60 synthetic commuter trips (seeded, reproducible), morning and evening peaks |

Current results on the sample trips (illustrative): 100% planned end-to-end; 87% multi-operator, 25% use all three modes;
~38% of door-to-door time is walking + waiting that a vehicle-time-only estimate misses.

## Swapping in verified data

Edit `data/network.json` (or the CSVs and adapt `build-data.js`), then `npm run build:data`.
Set a walk's `status` to `"verified"` only when it has been checked; set `min` to `null` if unmeasured.
Replace `fares.*.bands` with the official tariff and update `source`.

## Layout

```
relay.code-workspace   .vscode/ (tasks, launch)   server.js   package.json   .env.example
src/engine.js          routing, timetable, fares, fare split, evaluation (browser + Node)
public/                index.html, styles.css, app.js, map.js, config.js, + generated data.js, engine.js
data/                  network.json + CSVs
scripts/               build-data.js, evaluate.js
test/engine.test.js
```

## Limits (by design)

No live vehicle feeds, ticketing, payments or accessible-routing guarantees. Walking times are estimates.
Next step for a pilot: one validated corridor with real fare tables and interchange links from the three data owners.

