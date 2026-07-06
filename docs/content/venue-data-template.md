# Venue Data Sheet (template)

One sheet per venue; this is what the milestone 1 seed script ingests and what the admin form mirrors. Fields map 1:1 to the `venues` table in technical-design.md section 5.

## Identity
- **name**:
- **venue_type**: restaurant | bar | hybrid
- **neighborhood**: (one of the quiz Q1 areas, or inner-ring town)
- **address / lat / lng**:
- **phone**:
- **website**:

## Discovery data
- **cuisines[]** (restaurants) or **drink_tags[]** (bars: cocktails | wine | beer | dive | rooftop):
- **price_tier**: 1..4
- **hours**: per-day open/close (source: posted schedule; refreshed nightly after ingest)
- **google_place_id**: (resolves ratings + hours + photos)
- **yelp_business_id**: (resolved via business-match on ingest)

## The user-facing surfaces
- **menu_url**: REQUIRED. Acceptance rule: a URL that renders a readable current menu on a phone (venue site page or PDF). Facebook-only or third-party-aggregator menus are last resort; if truly nothing exists, note it and the card falls back to website then phone.
- **booking_platform + booking_url**: opentable | resy | tock | none (walk-in/phone)
- **photos**: 3 to 5, admin-picked from Google Places photo API / Yelp photo URLs (attribution retained). Alt text REQUIRED per photo (publish is blocked without it). Order matters: photo 1 is the card front.
- **parking_notes**: one line ("Lot behind, meters free after 6")
- **dress_code**: only if the venue actually enforces one

## Veto-relevant flags (these gate hard filters; be conservative: only set true if confident)
- **dietary_flags**: vegetarian / vegan / gluten_free / halal / kosher
- **wheelchair_access**: entrance AND seating
- **accessible_restroom**:
- **noise_level**: quiet | moderate | loud (drives the sound-sensitive veto and the cozy scorer)
- **lighting**: dim | warm | bright

## Editorial
- **signature_dishes[]**: 1 to 3
- **authenticity_note**: one sentence, factual (who runs it, what tradition, since when)
- **editorial_sources**: links to CT Bites / Courant / Hartford Mag / Infatuation coverage (24-month window for Phase 1 criteria)
- **has_patio**: plus any seasonal notes

## Curation checklist (Phase 1 bar; alpha may waive rating thresholds for balance)
- [ ] 4.3+ count-weighted rating across Google + Yelp, 200+ combined reviews
- [ ] Active reservation system OR consistent phone/walk-in service
- [ ] Dinner (or bar evening) service 5+ nights/week
- [ ] Editorial mention within 24 months
- [ ] No more than one ownership/concept change in 18 months
- [ ] Not a national chain (CT mini-chains of 5 or fewer locations OK)
- [ ] menu_url passes the phone-readability rule
- [ ] 3+ photos picked with alt text
- [ ] All 5 archetype blurbs drafted and approved
