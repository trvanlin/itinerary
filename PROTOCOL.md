# Itinerary Protocol

Working guide for Claude when updating `trvanlin.github.io/itinerary`.

---

## The Workflow

1. Thor pastes a confirmation (email text, PDF, app screenshot — any format)
2. Claude parses it and updates the JSON block in `index.html`
3. Thor pushes via GitHub Desktop → live in ~60 seconds

---

## Data Model

All trip data lives in the `<script id="trip-data">` block at the top of `index.html`.

### Trip

```json
{
  "id": "unique-slug",
  "name": "SFO–BUR",
  "subtitle": "Christmas 2026",
  "destination": "Burbank, CA",
  "accent": "#dc2626",
  "travelers": ["Thor", "Ali", "Madie", "Skylar"],
  "dates": { "start": "2026-12-25", "end": "2026-12-30" },
  "segments": [ ... ]
}
```

### Segment types

| type | Key fields |
|------|-----------|
| `flight` | airline, flightNumber, from{code,city,terminal,gate,time}, to{...}, duration, class, seat, confirmation, flightyUrl, notes |
| `hotel` | name, address, phone, checkIn, checkOut, nights, roomType, confirmation, notes |
| `car` | company, class, pickup{location,address,time}, dropoff{location,address,date,time}, confirmation, notes |
| `activity` | name, time, address, duration, confirmation, notes |
| `restaurant` | name, time, address, phone, confirmation, notes |
| `note` | title, notes |

---

## Naming Conventions

**Trip names**
- Route-style for work/commuter hops: `SFO–BUR` (en-dash, uppercase IATA)
- City name for leisure/family trips: `Chicago`, `Los Angeles`

**Subtitles:** descriptor or occasion — `Christmas 2026`, `Work Trip`, `Long Weekend`

**Seat format:** range only — no names
```
8A–8D
```

**Travelers:** field kept as empty array `[]` — names are not stored in trip data

**Accent colors**
- Work trips: indigo `#6366f1`
- Family/leisure: pick something fitting; SFO–BUR Christmas uses `#dc2626`

---

## Rendering Rules

- `address` on hotel/car segments → auto-linked to Google Maps
- `flightyUrl` on flight segments → renders "Flighty ↗" link
- `notes` → renders only if non-empty; keep factual/operational, not editorial
- `confirmation` → click-to-copy pill

---

## Lessons Learned

**Rental car companies**
- Green badge + "Executive Area" = **National**, not Hertz
- National class field: `"Executive Area · Midsize"` (not vehicle model)
- National car pickup/dropoff: add `address` field for Maps link (BUR = `2501 N Hollywood Way, Burbank, CA 91505`)

**Flighty links**
- Cannot be generated from flight number + date — they're app-generated hashes
- Format: `https://live.flighty.app/[hash]`
- Thor shares from Flighty app → pastes link here → stored as `flightyUrl` on the segment
- Flights without a flightyUrl simply show nothing (renderer is conditional)

**Hotel checkout vs. return flight**
- Checkout date and return flight date may differ — add both as confirmed, don't assume they match

**Airport timing**
- BUR is small; a 6-minute gap between landing and car pickup is fine

**Trip end dates**
- Use actual checkout/final day, not the return flight date if they differ

---

## Deployment

- Repo: `github.com/trvanlin/itinerary` (public)
- Live: `trvanlin.github.io/itinerary`
- Branch: `main` → GitHub Pages auto-deploys on push
- Push method: GitHub Desktop (git credentials not available in Claude terminal sessions)
- Multiple commits per session is fine — push once at the end
