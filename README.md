# TAKT — Berlin transfer timer

**Every ride drawn on one clock — the gap is your wait.**

TAKT answers a question every Berlin commuter knows: *which tram should I catch so I'm not
standing on the U-Bahn platform for five minutes?* It pulls live BVG/VBB real-time data and
draws every upcoming option as a strand on a shared time axis — ride, transfer walk, **wait
(visible as a hatched gap, color-coded)**, onward ride — so the best connection is obvious at
a glance.

## Features

- 🎨 **Marey-style journey timeline** — each option is a strand on one shared time scale with a
  live "NOW" line sweeping across; the transfer wait is literally a gap you can see
- 🟢 **Wait-first UI** — the recommended ride is a hero card with its wait as a giant number
  ("take this one"); every other option shows its wait as a big color-coded stat
  (green ≤ 2 min, amber 3–4, red ≥ 5) for at-a-glance comparison
- 🚪 **Door-to-door** — "leave by", a live *leave now!* nudge, and arrival "at the door"
  including your walk times
- 📡 **Real-time** — delays, cancellations, and night-line fallbacks (when the U-Bahn stops,
  TAKT correctly offers the night bus); auto-refresh every 60 s and on screen wake
- 🔁 **Both directions** — one tap swaps the commute home
- 🌍 **Fully generic** — any start/transfer/destination stop in the VBB network (tram, U-Bahn,
  S-Bahn, bus, ferry, regional) via live autocomplete; routes are shareable as links
- 🪶 **Zero stack** — one static HTML file, vanilla JS, no build step, no server, no API key,
  no tracking (config lives in `localStorage`); PWA manifest for "Add to Home Screen"
- ♿ Honors `prefers-reduced-motion`; official Berlin line colors (U5 brown, U8 navy, …)

## Run it

```bash
python3 -m http.server 8000   # → http://localhost:8000
```

or just open `index.html`. First visit shows the route settings prefilled with an example
(Straßmannstr. → U Frankfurter Tor → U Museumsinsel); pick your own stops and save.

## Host it

It's three static files (`index.html`, `manifest.json`, `icon.svg`) — GitHub Pages, Netlify,
Cloudflare Pages, or any web server. No environment, no secrets.

## How it works

Live data comes from [`v6.bvg.transport.rest`](https://v6.bvg.transport.rest), a free,
CORS-enabled community REST wrapper around the BVG/VBB HAFAS API (4 requests per refresh,
well within its 100 req/min limit):

1. `GET /stops/{start}/departures?direction={transfer}` — every vehicle from the start stop that passes the transfer station
2. `GET /stops/{transfer}/arrivals` — matched by `tripId` for each vehicle's live arrival at the transfer (falls back to `GET /trips/{tripId}` for long rides)
3. `GET /stops/{transfer}/departures?direction={destination}` — the first onward departure after *arrival + transfer walk* is the connection; the difference is the platform wait
4. `GET /stops/{destination}/arrivals` — matched by `tripId` for the final arrival time

All four run in parallel; real-time `when` is preferred, planned times are the fallback.
The timeline scale is computed per refresh, re-laid-out on resize, and the NOW markers
advance between refreshes.

---

Built in Berlin. Data © VBB/BVG via the community API — be kind to it.
