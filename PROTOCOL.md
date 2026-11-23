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
  "type": "personal",
  "accent": "#dc2626",
  "icon": "🎄",
  "travelers": [],
  "dates": { "start": "2026-12-25", "end": "2026-12-30" },
  "segments": [ ... ]
}
```

### Segment types

| type | Key fields |
|------|-----------|
| `flight` | airline, flightNumber, from{code,city,terminal,gate,time}, to{...}, duration, distance_mi, aircraft, class, seat, confirmation, flightyUrl, notes |
| `hotel` | name, address, phone, checkIn, checkOut, nights, roomType, confirmation, notes |
| `car` | company, class, pickup{location,address,time}, dropoff{location,address,date,time}, confirmation, notes |
| `activity` | name, time, address, duration, confirmation, notes |
| `restaurant` | name, time, address, phone, confirmation, notes |
| `note` | title, notes |

---

## Naming Conventions

**Trip names** — route-style for all trips: `SFO–BUR`, `SFO–GSO` (en-dash, uppercase IATA)
- If destination airport is TBD, use city name temporarily until confirmed

**Subtitles:** descriptor or event name — `Christmas 2026`, `Contracts Council`, `Bora In-Person`. For TripIt imports, use a shortened version of the PDF label.

**Icon:** optional emoji on the card and detail hero. Special trips should always have one; personal and work trips may or may not. Up to two emojis is fine (e.g. `🌺💍` for a honeymoon in Hawaii).

**Seat format:** range only — no names
```
8A–8D
```

**Travelers:** field kept as empty array `[]` — names are not stored in trip data

**Trip type** — required field on every trip; drives filter pill and card stripe color
- `"work"` → Packers gold `#FFB612`
- `"personal"` → Packers green `#2d9e5a`
- `"special"` → cyan `#06b6d4` — notable personal experiences: Hawaii, international/OCONUS, Disneyland, Alaska cruise, Depeche Mode concerts, milestone celebrations. NOT Christmas/Easter/regular personal trips or Vegas (personal). When in doubt: if it's the kind of trip you'd frame a photo from, it's probably special.

**Destination** — always set to the hotel city (`"Tempe, AZ"`, `"Chantilly, VA"`). Displayed on card below subtitle. For trips with multiple hotel cities, use the primary meeting location.

**Accent colors** — used only in the detail hero; card stripe always uses type color
- `work` → `#FFB612`
- `special` → always `#06b6d4` (no thematic colors — consistent cyan throughout)
- `personal` → pick per occasion (`#dc2626` Christmas, `#2d9e5a` default green, etc.)

**Tab labels** — `Current Trips` (upcoming/active) and `Past Trips` (archived). Internal JS values remain `upcoming` / `archive`.

---

## Rendering Rules

- Date ranges show day-of-week with numeric date: `Mon 12/14/26 – Wed 12/16/26` (card and hero, via `formatDateDOW`)
- `address` on hotel/car segments → auto-linked to Google Maps
- `distance_mi` on flight segments → stored for future analytics (great circle mi; estimated from route data when not on confirmation)
- `aircraft` on flight segments → stored when available from TripIt/confirmation (omit if unknown)
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

## Stats Tab

The Stats tab is a live analytics view built from the archived trip data.

**Filters:** type pills (Work / Personal / Special) + year pills, same bar as Past Trips. Type pills are **multi-select** — you can combine Personal + Special to see all leisure travel. "All" resets. Year remains single-select.

**Tiles** (Miles Flown / Flights Taken / Trips Completed / States Visited): when any filter is active, each tile shows `X% of total` in small text below the big number.

**Trips by Type card:** full-width stacked bar + count/miles/% rows. Visible only when no type filter is active (all three types showing).

**Route stats count individual flight legs**, not round trips or trips. SFO↔OGG showing "3 flights" is correct if you flew that corridor 3 times as individual legs — a return via SJC instead of SFO creates a separate OGG–SJC route entry.

**Flighty Passport globe:** base64-embedded day/night images with 🌙/☀️ toggle. Date-stamped bottom-right. Update the stamp string and re-embed the images when refreshing (~annually).

---

## TripIt Import Workflow

TripIt exports trips as PDFs (`File > Print/Export` in TripIt app). PDF format: trip header with dates, then segments (flights, hotel, car) each with confirmation, seat, aircraft, distance, duration.

**Import checklist:**
1. Trip name: `SFO–XXX` route-style (outbound destination airport code)
2. Subtitle: shortened PDF label (e.g., "2026.01.22, Contracts Council" → `"Contracts Council"`)
3. `type: "work"` for all BCD-booked corporate travel; check destination for `"special"` on personal imports (Hawaii, international, Disneyland, etc.)
4. Add `aircraft` and `distance_mi` to each flight segment from PDF; estimate distance if missing (great circle)
5. Connection flights: one segment per leg (same day, sequential times)
6. Return from different airport (e.g., BUR vs LAX): use outbound destination for trip name, correct departure code on return segment
7. Partial hotel data (1 night captured, trip is 5 days): note gaps in `notes` field

**Known routes (mi):** SFO–PHX 650 · SFO–LAX 337 · BUR–SFO 326 · SFO–IAD 2412 · SFO–EWR 2558 · SFO–ORD 1841 · ORD–PWM 897 · SFO–DEN 965 · DEN–SFO 965 · PWM–DEN 1776 · DTW–IAD 382 · SFO–DTW 2073 · SFO–MCO 2441 · SJC–LGB 323 · SFO–GSO 2455 · DCA–SFO 2436 · SJC–LAX 307 · SJC–LHR 5353 · LHR–DUB 279 · OAK–LAS 407 · SJC–BUR 291 · SFO–LAS 570 · LAX–OAK 337 · SAN–SJC 428 · SFO–CLT 2297 · CLT–HHH 212 · SFO–SAN 447 · MCO–IAD 751 · SFO–OGG 2398 · OGG–SJC 2334 · SFO–SNA 400 · SFO–SEA 679 · SFO–MSY 1926 · SJC–LAS 406 · SFO–CPH 5367 · OAK–BUR 326 · SFO–YYZ 2497 · YYZ–CPH 3990 · CPH–YUL 3735 · YUL–SFO 2540 · SFO–PSP 418 · SFO–LIH 2449 · LIH–SFO 2449 · SJC–SAN 448 · SAN–SJC 448 · SFO–BUR 326 · SFO–JAC 846 · JAC–SFO 846 · SFO–ONT 390 · ONT–SFO 390

---

## Keeping the Dashboard in Sync

The home dashboard (`~/home-dashboard/dashboard.html`) has a **Booked Flights** panel on the Travel tab with its own `BOOKED_TRIPS` array (line ~2789). It is a separate data store and does not auto-update from the itinerary.

**After any trip add/edit here, ask:** "Want me to update the dashboard's Booked Flights panel to match?"

Dashboard leg format (for reference):
```js
{ flight: "UA 2764", route: "SFO→BUR", dep: "4:21 PM", arr: "5:36 PM", date: "Dec 25" }
```

Trip fields: `destination`, `wikiTitle` (Wikipedia article name for photo fetch), `start`, `end`, `legs[]`, `activities[]`. TBD trips take `tbd: true, featured: true`.

---

## Deployment

- Repo: `github.com/trvanlin/itinerary` (public)
- Live: `trvanlin.github.io/itinerary`
- Branch: `main` → GitHub Pages auto-deploys on push
- Push method: GitHub Desktop (git credentials not available in Claude terminal sessions)
- Multiple commits per session is fine — push once at the end

