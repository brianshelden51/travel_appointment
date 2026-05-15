# FlightCal — Claude Context

## What this is
A single-file static web app (`index.html`) that parses flight confirmation emails via the Claude API and generates `.ics` calendar files for Apple Calendar. No backend, no build step — all CSS and JS are inline.

**Live URL:** https://brianshelden51.github.io/travel_appointment/ (GitHub Pages, `main` branch, HTTPS enforced)

## Architecture rules
- Stay single-file. No build pipeline, no external dependencies, no backend.
- Anthropic API key stored in localStorage; never sent anywhere except `api.anthropic.com`.
- Claude model: `claude-sonnet-4-6` with prompt caching on the system prompt.

## Parser rules (system prompt)
- `terminal` = **departure terminal only** — not the arrival terminal. Codeshare emails often list both; the arrival terminal must never bleed into the departing leg. This was a real bug with Air France LEX→DTW→CDG.
- `checkin_hours` = detected from email content (e.g. "check-in opens 30 hours before departure"); default 24 if not stated. No hardcoded airline names.

## ICS / alarm rules
- **Alarm formula:** `triggerMinutes = checkinHours * 60 + 30` — fires 30 min *before* the check-in window opens.
- **First leg:** check-in reminder alarm at `triggerMinutes` before departure.
- **All other legs:** 1-hour departure alarm (`PT60M`).
- **Timezones:** `AIRPORT_TZ` maps IATA → IANA. Unknown airports use floating time with a warning.

## Apple Calendar location
- `LOCATION` = airport name + terminal (no postal address) — clean for non-Apple clients.
- `X-APPLE-STRUCTURED-LOCATION;VALUE=URI;X-TITLE="…":geo:lat,lon` — pins the event in Apple Calendar. `AIRPORT_COORDS` has lat/lon for all 70 airports.
- **Known limitation:** Without `X-APPLE-MAPKIT-HANDLE`, the pin is anonymous (no POI card). Getting real handles requires Apple MapKit JS. **Pending:** Brian's Apple Developer account reactivation → one-time lookup script to pre-bake handles for all airports and hardcode them alongside `AIRPORT_COORDS`.

## Planned expansion
Car rentals, trains, hotels. Strategy:
- **Fixed location types** (airports, train stations): pre-bake MapKit handles.
- **Arbitrary locations** (hotels, rental offices): street address + geo URI is sufficient — unique addresses geocode reliably without a handle.

## Airport data tables
Three parallel lookup tables keyed by IATA code:
- `AIRPORT_TZ` — IATA → IANA timezone string
- `AIRPORT_ADDR` — IATA → address array `[name, street?, city+zip, country]`
- `AIRPORT_COORDS` — IATA → `[lat, lon]`

All three should stay in sync when adding new airports.
