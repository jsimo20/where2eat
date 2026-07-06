# Where2Eat: Technical Design (MVP)

Covers Phase 0 alpha through Phase 1 of [the PRD](../W2E_PRD_Hartford_Prototype.md) (v2.3: Android APK alpha, restaurants + bars, swipe deck, group matching, minimalist design).
Scenario IDs (S1..S52; S20, S22, S24 retired) reference [customer-journeys.md](customer-journeys.md). Decision IDs (D1..D10) are stable across revisions.

---

## 1. Design goals, ranked

1. **The swipe must feel native.** 60fps gesture physics, zero-spinner photo loads. This is now the product bet; everything else bends to it.
2. **p95 under 2s for deck generation.** Everything on the hot path is precomputed or in our own database.
3. **Low cognitive load.** One decision per screen, progressive disclosure, two numbers max per card. Minimalism is a release gate, not a vibe.
4. **Honest degradation.** No external API failure ever blocks the deck (S23, S26). The floor is always a phone number.
5. **Anonymous-first core, account-gated social.** No account for the solo/couple experience; group matching (2 to 10 people) requires one (S48).
6. **One developer can build and run it.** Managed services, no ops burden, sideload-friendly builds.
7. **Multi-city ready, single-city built.** `city_id` on every content table; zero other speculative generality (YAGNI).

## 2. Key architecture decisions

Short ADR format: decision, why, alternative rejected. Revision status marked per decision.

### D1. Card copy is pre-authored, not generated at request time (unchanged, repurposed)

- **Decision:** Each venue carries editorially approved fragments per energy archetype ("why this place tonight" blurbs, with optional season/weather variants). Deck generation is deterministic assembly: filter, score, rank, attach the matching fragment and photos to each card.
- **Why:** With 25 to 100 curated venues, authoring is tractable (Claude drafts in the admin tool, editor approves). Buys: p95 well under 2s (no LLM on the hot path), zero per-request inference cost, editorial voice control, offline-friendly output, deterministic same-order decks per session (S13; match sessions run on this).
- **Rejected:** Per-request LLM generation. Latency and cost fight the 2s p95; request-time quality control contradicts the curated positioning.
- **v2 note:** title seeds and opening lines (for 3-act stories) are no longer authored in MVP; only `why_tonight` blurbs. Multi-act rendering is deferred (Appendix A).

### D2. Expo (React Native) client + Next.js API/admin (revised in v2)

- **Decision:** The user-facing client is an Expo (React Native, TypeScript) Android app. The Next.js app survives as the API layer and the web admin/curation tool, plus public landing pages: the invite quiz (S2) and the session-join landing (S42). Shared zod schemas/types live in a shared workspace package.
- **Why:** The swipe deck is the product. WebView wrappers (Capacitor, TWA) render gestures through the browser engine and get janky on mid-range Androids, exactly where a Tinder-feel dies. React Native drives native views with gesture handling on the UI thread (`react-native-gesture-handler` + `reanimated`), which is how the apps we're imitating actually feel. Expo makes APK builds and OTA updates one-command. React + TypeScript skills carry over from the web stack.
- **Rejected:** Capacitor (one codebase, but compromises the core interaction), Flutter (new language for no gain here), pure native Kotlin (kills iOS reuse later and slows a one-dev team).
- **Cost accepted:** two build targets (mobile client + web admin) in one monorepo.

### D3. Minimal anonymous server profile; accounts ship in MVP as the social gate (revised in v3)

- **Decision:** An opaque UUID profile row is created server-side on first quiz submit. It holds quiz answers, veto answers, and couple membership. No name, email, phone, or location. Plans and history live on-device with a server copy for couple sync; the server keeps only what pairing, blending, and sync require. Accounts (Supabase Auth) now ship in MVP because group matching requires them (D9, S48): signing up attaches auth + a display name to the same profile and migrates local history (S31). Account holders carry the product's only PII: auth email and display name.
- **Why:** Async invite (S2) and solo-partner-joins-later (S6) are impossible with purely on-device data; friends lists and cross-device match sessions are impossible without durable identity. The anonymous core stays anonymous; PII arrives only when the user opts into the social layer, and export/delete covers all of it (S33).

### D4. Availability: three verified states, demand-driven reservation checks (revised in v6: no guesses, no manual entry)

- **Decision:** Availability is exactly three user-facing states, each backed by something we verified. **Closed**: the venue's posted schedule (nightly refresh, J1) says it isn't open tonight; closed venues never enter a tonight deck. **Open, walk-in or call**: open per schedule but no verified seats, whether because the venue takes no reservations, a fresh check found none, or we couldn't check. Guidance: "try walking in or call the restaurant"; if the venue has a reservation platform, its link stays available in the detail sheet without an availability claim. **Open, reservations available**: a reservation-platform check (OpenTable and peers) fresh within 30 minutes found seats for tonight at the session's party size: badge + one-tap prefilled booking. State is computed at read time, never stored as a venue fact, and never affects ranking.
- **Demand-driven checks (S51):** deck generation and detail-sheet opens enqueue background reservation checks for surfaced venues whose latest check is older than 5 minutes; a queue worker asks the partner API and writes the result. Screens render instantly from cache; the reservations badge appears in place when fresh data lands. No external calls on the request path, rate limits spent only where users are looking.
- **Honest dependency:** partner API access is granted, not built. Applications (OpenTable affiliate/partner program, Yelp Guest Manager, SevenRooms, Tock, Resy/Amex) go out at milestone 1. Until access lands, the reservations state simply never appears: open venues show walk-in/call guidance plus their booking links, which is true and still useful.
- **Removed in v6:** pattern-inferred user messaging ("usually has tables Tue at 7" is a guess we won't present as knowledge), manual admin confirms (ops burden, cut to keep it simple), stale-claim messaging ("last confirmed 23 min ago"), availability-based down-ranking, and automated swap suggestions. The daily schedule refresh and a live seat check are the only two things we let ourselves assert.
- **Rejected:** scraping partner availability (ToS breach for a partnership-dependent product), full-catalog polling on a timer (burns rate limits on venues nobody is browsing).

### D5. Venue-to-venue distances (deferred in v2)

- Was: precompute drive/walk times between venues for act chaining. Deferred with multi-act itineraries (Appendix A). Distance shown on cards is user-to-venue, computed client-side from device location (if granted) or neighborhood centroid, never sent to the server.

### D6. Deterministic learning, no ML (revised in v3: swipe nudges follow the swiper)

- **Decision:** Explicit feedback (S27) writes strong boosts at the couple level; swipes write weak nudges keyed to the individual swiper's profile (S29). Couple/solo decks combine the relevant profiles' affinities; group decks apply no personal weights at all (S43). Small weights, feedback dominates, everything visible in the admin tool.
- **Why:** At 5 to 100 couples there is no training data, and rules stay debuggable and explainable ("because you loved X"). Swipes now have individual identity (match sessions require it), so tastes accrue to the person: your friend group's session doesn't pollute your date-night profile, and group decks stay fair and identical for everyone.

### D7. Sideloaded APK with OTA updates (new in v2)

- **Decision:** Alpha distribution is a signed APK, sideloaded (no Play Store). Built via EAS Build (`preview` profile produces an installable APK) or locally via `expo prebuild` + Gradle. JS-level changes ship over the air with `expo-updates`, so testers reinstall the APK only when native dependencies change.
- **Why:** Matches the stated plan (sideload now, launch strategy later). OTA turns the alpha iteration loop from "re-send an APK and re-install" into "restart the app", which matters when the testers are James and spouse on date night.
- **Details:** signing keystore generated once, stored outside git (`*.jks` gitignored per house rules); testers enable "install unknown apps" once; distribution via EAS internal-distribution link or direct file share.

### D8. The deck replaces itinerary assembly as the core engine output (new in v2)

- **Decision:** `generateDeck(blend, story, filters, context)` returns up to ~30 ranked venue cards. The old pipeline survives intact up through scoring; what changed is the output shape (ranked cards, not assembled 3-act stories) and one new input (user filters, S36).
- **Why:** Locks the UI the PM wants to validate while preserving the engine investment. When multi-act itineraries return, they compose on top of the same filter/score core.

### D9. Group matching: fully async, deadline-bound, leaderboard-first (revised in v5)

- **Decision:** A match session is host-created and account-gated, with a close time (`closes_at`) set at creation (presets: 1h / 3h / this evening / 24h). The deck is generated once, at creation, from the host's frame (area + story) and the vetoes of everyone on the invited roster; the session seed fixes the order permanently. Participants join (roster or share link) and swipe on their own schedule any time before the deadline. A link-joiner's vetoes disqualify venues monotonically: the deck shrinks for everyone, never reorders or grows, and already-cast swipes on a disqualified venue are discarded at results (and disclosed there). The session closes at the deadline or on host early-close; close is evaluated on read, so it is exact to the minute without a high-frequency cron. Results materialize at close: venues ranked by right-swipe count, blend fit as tie-break, unanimity among actual swipers highlighted at the top. Undo works freely until close; swipes are rejected after. Only the host locks a pick into the plan. Friends are added by invite link only; no chat, no search, no contact upload.
- **Why:** nobody is on the app at the same time; the v3 lobby model quietly assumed otherwise. A deadline replaces both the lobby and "everyone done" detection in one move; monotonic disqualification preserves veto sanctity without regenerating a deck under people mid-swipe; leaderboard-at-close is a single rule that behaves identically for 2 people and 10. Host authority still means one person books.
- **Rejected:** lobby + roster lock (v3: required near-synchronous joining, impractical for real groups), live mid-session match popups (synchronous assumption; the unanimity celebration moved into the results screen), majority-vote thresholds (still arbitrary), client-computed results (trust boundary), deck regeneration on join (a rug-pull under anyone mid-swipe).

### D10. Explore map is a view over the deck, not a second engine (new in v4)

- **Decision:** A full-screen map toggle (react-native-maps, Google provider) whose pins are exactly the current session's candidate pool: same vetoes, same filters, same story, same scoring. Tap a pin for its card in a bottom carousel (carousel swipes pan the map); pass/shortlist from the map count as normal swipes; tap-through lands on the same detail sheet. Location permission optional (defaults to Hartford center; position never leaves the device).
- **Why:** browsing spatially is a real mode ("what's near the theater?"), but a second ranking surface would fork the product logic and the cognitive model. One engine, two projections keeps vetoes unbreakable (a vegan-hostile venue can't sneak in via the map) and keeps swipe data unified.
- **Rejected:** split-screen map+list (phone real estate, violates one-decision-per-screen), Mapbox (fine product, but Google's provider pairs with our Places data and the dev's likely familiarity), map-first UX (the deck is the bet; the map is disclosure of geography).

## 3. Stack

| Layer | Choice | Why it fits |
|-------|--------|-------------|
| Mobile client | Expo (React Native, TypeScript), expo-router | Native gesture feel (design goal 1), one-command APK, OTA updates, React skills carry over. |
| Swipe deck | react-native-gesture-handler + react-native-reanimated | Gestures and spring physics run on the UI thread, not the JS thread: this is the 60fps difference. |
| Images | expo-image | Built-in disk/memory cache, prefetch API (next 3 cards), blurhash placeholders (S40). |
| Mobile styling | NativeWind (Tailwind syntax for RN) + custom design tokens | Keeps the Tailwind mental model across web admin and app; tokens enforce the minimalist system (section 9). |
| Local storage | MMKV (key-value) + expo-image cache | Profile pointer, saved plans, shortlist, offline cache (S25). MMKV is the fast RN-standard store. |
| API + admin | Next.js 15+ (App Router, TypeScript) on Vercel | API routes for the app, web admin for curation, invite-landing quiz page (S2). |
| Database | Postgres (Supabase) | Relational fits the model; Supabase bundles auth and storage. |
| Auth | Supabase Auth (MVP) | Group matching is account-gated (D9); anonymous UUID profiles link to the account on signup. Vetted library, never roll our own (house rule). |
| Deep links | expo-linking + Android App Links | Friend and session invites open straight into the session (S42). |
| Maps | react-native-maps (Google provider) | Explore mode (D10). Android SDK key is billing-enabled but free-tier generous; restricted by package name + SHA-1. |
| ORM | Drizzle | Type-safe schema in TypeScript; schemas shared with the client via the monorepo package. |
| Cache / rate limit | Upstash Redis + @upstash/ratelimit | Availability snapshots, weather cache, day-one rate limiting (PRD requirement). |
| Weather | NWS API (api.weather.gov) | Free, no key, US government, hourly forecast for the Hartford grid. Open-Meteo fallback. |
| Venue data | Google Places API, curation-time only | Enrichment on ingest + nightly hours/closed/photo-URI refresh. Never called on the request path. |
| Ratings | Yelp Fusion API, curation-time only | Second rating source for the Google+Yelp aggregate and the automated curation-bar check (open question 7). |
| Content authoring | Claude API (batch, in the admin tool) | Drafts card blurbs per venue x archetype from venue facts + voice guide; editor approves before publish. |
| Analytics | Server-side aggregate counters (`metrics_daily`) | Aggregate-only, no individual tracking, no third-party SDK in the app. |
| Testing | Vitest (engine), Maestro (mobile e2e), Playwright (admin e2e), manual TalkBack pass | Blend + deck engines are table-driven-test targets; a11y is a release gate. |

Dropped from v1: Serwist/PWA (replaced by the native app), Radix (web-only; admin can keep it), Plausible (web analytics; the app reports to our own counters).

## 4. System architecture

```mermaid
flowchart TB
    subgraph Client["Android app: Expo / React Native"]
        DECK[Swipe deck: cards, filters, detail sheet]
        EXPL[Explore map: pins over the current deck pool]
        MATCH[Match sessions: friends, async swiping, progress, results]
        PLANS[Shortlist + plans, offline via MMKV + image cache]
        OTA[expo-updates OTA channel]
    end

    subgraph Web["Next.js on Vercel"]
        RL[Rate limit middleware]
        API[API routes: profiles, quiz, couples, invites, blends, decks, swipes, sessions, friends, plans, feedback, flags, export]
        INV[Invite landing pages: partner quiz, session join]
        ADM[Admin: venue CRUD, fragment authoring, photo picks, event flags, flag queue, learning inspector]
    end

    subgraph Data["Data layer"]
        PG[(Postgres via Supabase: venues, fragments, profiles, couples, blends, sessions, swipes, plans, feedback)]
        RD[(Upstash Redis: reservation checks, weather cache, rate limits)]
    end

    subgraph Jobs["Jobs: Vercel Cron + demand-driven queue"]
        J1[Nightly: Places + Yelp refresh, hours and closed checks, photo URIs, curation-bar and quality alerts]
        J2[Hourly: NWS weather pull for Hartford]
        J3[Hourly: invite expiry sweep at 48h]
        J5[Queue worker: reservation checks for surfaced venues with stale data]
    end

    subgraph Ext["External (never on the request path except deep links)"]
        GP[Google Places API]
        YF[Yelp Fusion API]
        NWS[NWS weather API]
        CL[Claude API: blurb drafting]
        PA[Partner availability APIs: OpenTable / Resy / etc., when access is granted]
        BK[Booking deep links: OpenTable, Resy, tel:]
        MP[Maps deep links + in-app menu browser]
    end

    DECK & EXPL & MATCH & PLANS --> RL
    INV --> RL
    RL --> API
    API --> PG & RD
    API -. enqueue stale refresh .-> J5
    J5 --> PA & RD & PG
    ADM --> PG
    ADM --> CL
    J1 --> GP & YF & PG
    J2 --> NWS & RD
    J3 --> PG
    DECK -- outbound actions --> BK & MP
```

Key property preserved from v1: the request path (quiz, blend, deck, swipes, plans, feedback) touches only Postgres and Redis. External APIs feed the data layer via jobs; booking, maps, and menus are client-side deep links out. That is what makes S26 (full external outage, deck still generates) true by construction. Card photos are the one nuance: the client loads them from the source CDNs (Google/Yelp) with on-device caching, so a photo-CDN outage degrades to cached images, never a blocked deck.

## 5. Data model

```mermaid
erDiagram
    CITIES ||--o{ VENUES : contains
    VENUES ||--o{ VENUE_FRAGMENTS : has
    VENUES ||--o{ RESERVATION_CHECKS : has
    PROFILES ||--o{ QUIZ_RESPONSES : submits
    COUPLES ||--o{ INVITES : issues
    PROFILES }o--o{ COUPLES : members
    COUPLES ||--o{ BLENDS : computes
    SESSIONS ||--o{ SWIPES : records
    PROFILES ||--o{ SWIPES : casts
    VENUES ||--o{ SWIPES : targets
    SESSIONS ||--o{ SESSION_PARTICIPANTS : seats
    USERS ||--o{ SESSION_PARTICIPANTS : joins
    USERS ||--o{ FRIENDSHIPS : links
    COUPLES ||--o{ PLANS : makes
    SESSIONS ||--o| PLANS : locks
    PLANS ||--o| FEEDBACK : receives
    PROFILES ||--o| PROFILE_AFFINITIES : accumulates
    COUPLES ||--o| COUPLE_LEARNING : accumulates
    CITIES ||--o{ EVENTS_LOCAL : hosts
    USERS ||--o{ PROFILES : claims
```

Tables and load-bearing columns (Drizzle schema will be the source of truth; this is the design intent):

- **cities**: `id, slug, name, timezone, active`.
- **venues**: `id, city_id, venue_type (restaurant|bar|hybrid), name, slug, neighborhood, lat, lng, price_tier (1..4), cuisines[], drink_tags[] (cocktails|wine|beer|dive|rooftop), phone, website, menu_url, booking_platform, booking_url, hours (jsonb), parking_notes, dress_code, has_patio, noise_level, lighting, wheelchair_access, accessible_restroom, dietary_flags (jsonb: vegan, vegetarian, gluten_free, halal, kosher), google_place_id, yelp_business_id, ratings (jsonb: per-source rating, review_count, observed_at), rating_aggregate, photos (jsonb: ordered URIs, source, attribution, blurhash), editorial_sources (jsonb), signature_dishes[], authenticity_note, curation_tier (1..4), status (draft|active|closed|flagged)`. `venue_type` + `drink_tags` power the drinks filter (S36); `menu_url` is a curation requirement (S39); `photos` are admin-picked, 3 to 5 per venue, URIs refreshed by J1 (S40). `yelp_business_id` resolved once via Yelp business-match; `rating_aggregate` is the review-count-weighted Google+Yelp average; J1 alerts when a venue drops below the curation bar (4.3 aggregate, 200+ combined reviews).
- **venue_fragments**: `id, venue_id, archetype (5 base + blend zones), kind (why_tonight), season_tag, weather_tag, text, status (draft|approved), author`. Card engine only reads `approved`. (Title/opening kinds return with multi-act, Appendix A.)
- **profiles**: `id (uuid), created_at, last_seen_at`. Deliberately nothing else (D3).
- **quiz_responses**: `id, profile_id, version, personality (jsonb: neighborhood, archetype, radius, budget_comfort, structure), vetoes (jsonb: dietary[], accessibility[], budget_ceiling), submitted_at`. Versioned (S8); Q5 structure options narrowed per journeys gap 9.
- **couples**: `id, profile_a_id, profile_b_id (nullable), mode (pass_phone|async|solo), created_at`.
- **invites**: `id, couple_id, token (unique), status (pending|accepted|expired|revoked), expires_at, created_at`. 48h expiry via J3 (S3, S4).
- **blends**: `id, couple_id, version, vetoes (jsonb), soft (jsonb), rotation (jsonb: next_primacy), score (0..100), summary, computed_at`. Append-only for auditability.
- **sessions**: `id, kind (solo|couple|match), couple_id (nullable), host_user_id (nullable), city_id, story_input (jsonb: archetype, filters, area, party_size), seed, status (open|closed|expired), planned_for, closes_at (nullable), results (jsonb, materialized at close), created_at`. One table for every swiping context; the seed makes each deck deterministic and replayable (S13, S43). Match sessions close at `closes_at` or host early-close, evaluated on read for exactness; `results` snapshots the leaderboard, unanimity flags, and discarded-swipe disclosures (S44, S45).
- **session_participants**: `session_id, user_id, state (invited|joined), joined_at`. Swipes accepted only from invited or joined participants while the session is open; a join applies that person's vetoes, shrinking the deck for everyone (S41, S43).
- **friendships**: `id, user_low_id, user_high_id, status (active|blocked), blocked_by (nullable), created_at`. Created only via accepted invite links; canonical low/high ordering prevents duplicate pairs (S47).
- **friend_invites**: `id, inviter_user_id, token (unique), status, expires_at, created_at`. Same share-sheet pattern as partner invites.
- **swipes**: `id, session_id, profile_id, venue_id, direction (left|right|undo), position, created_at`. Carries the swiper: results are computed from these at session close, and learning nudges accrue to the individual (D6, D9). Powers dedupe (S17) too. No dwell-time or other surveillance-shaped fields.
- **plans**: `id, couple_id (nullable), session_id (nullable), venue_ids[] (one, or dinner + drinks pair), planned_for (date), party_size, solo_planned (bool), status (saved|completed_presumed|completed_confirmed|abandoned), snapshot (jsonb: rendered cards at save time), created_at`. Group plans come from a locked match session with party size prefilled from the roster; the snapshot keeps a saved plan immutable even if venue data changes later (S25).
- **feedback**: `id, plan_id, profile_id, would_repeat (bool, nullable), moods[], created_at` (S27). Group plans prompt every participant; each answer feeds that person's own learning.
- **profile_affinities**: `profile_id, attribute_affinities (jsonb), updated_at`. Swipe nudges accrue to the swiper (D6).
- **couple_learning**: `couple_id, venue_boosts (jsonb), updated_at`. Feedback boosts stay couple-level (D6).
- **reservation_checks**: `id, venue_id, source (opentable|resy|yelp_gm|...), slot_at, party_size, seats_found (bool), checked_at`. Redis hot copy, Postgres history. Replaces the tier-era availability_snapshots: reservation platforms are the only writer (D4); Closed/Open comes straight from `venues.hours`.
- **events_local**: `id, city_id, date, name, area (bushnell|peoplesbank|trinity_health|other), start_time, expected_impact` (S16).
- **content_flags**: `id, subject_type, subject_id, reason, status, created_at` (S34).
- **users** (MVP): Supabase Auth row + `display_name` (shown in lobbies and friends lists; the only PII besides the auth email); `profiles.user_id` nullable FK links the anonymous history on signup (S31, S48).
- **metrics_daily**: `date, metric, value`. Aggregate-only counters.

Dropped from v1: `itineraries`, `itinerary_acts`, `venue_distances` (deferred with multi-act, Appendix A). Renamed in v3: `swipe_sessions` generalized to `sessions` + `session_participants`.

## 6. Core flows

### 6.1 Async invite pairing (S2, S3, S4, S6): unchanged from v1

```mermaid
sequenceDiagram
    participant A as Partner A (app)
    participant API as API
    participant DB as Postgres
    participant B as Partner B (web quiz page)

    A->>API: POST /quiz (answers + vetoes)
    API->>DB: create profile A, quiz_response
    A->>API: POST /couples (mode async) + POST /invites
    API->>DB: couple (B null), invite token, expires 48h
    API-->>A: share URL
    A->>B: shares link via device share sheet (SMS or email, user-sent)
    B->>API: GET /invite/:token
    API-->>B: web quiz UI (no APK required, token validated, single use)
    B->>API: POST /quiz + accept invite
    API->>DB: create profile B, attach to couple, invite accepted
    API->>API: compute blend
    API-->>B: blend summary + score
    Note over A: next open: "blend ready" banner
    Note over API: Cron J3: token past 48h and pending -> expired; A sees solo-or-resend prompt on next open
```

### 6.2 Deck generation pipeline (S12..S18, S36)

```mermaid
flowchart TD
    A["Input: blend or solo profile, tonight's story, user filters, party size"] --> B[Load context: weather cache, event flags, rotation state, learning weights, recent swipes]
    B --> C["HARD FILTER: city, venue_type + cuisine/drinks filters, open tonight, dietary union, accessibility union, budget ceiling, radius"]
    C --> D{Enough cards?}
    D -->|No| E["Offer soft relaxations, user-approved: radius expand, widen neighborhoods, clear filters. Vetoes never relax."]
    E --> C2{Still near empty?}
    C2 -->|Yes| F[Honest thin-deck screen: open count + closest misses with what blocked each]
    C2 -->|No| G
    D -->|Yes| G[SCORE each candidate]
    G --> H["Rank top ~30, light diversity shuffle within score bands, seeded by couple + date"]
    H --> I[Annotate availability state per card]
    I --> J["Attach card content: photos, blurb fragment for archetype and weather, rating aggregate, fit score, menu and booking actions"]
    J --> K[Return deck, log aggregate metrics]
```

Scoring (deterministic, tunable weights in config):

| Component | Range | Notes |
|-----------|-------|-------|
| Archetype fit | 0..40 | Direct match, or third-way blend-zone match (S10) |
| Neighborhood fit | 0..15 | Rotation-aware when disjoint (S11) |
| Price fit vs budget comfort | 0..15 | Comfort zone is soft; the ceiling was already a hard filter |
| Quality | 0..10 | From curation tier + rating_aggregate (Google + Yelp, count-weighted) |
| Context | -10..+10 | Patio boost or suppression by weather (S15), event-night proximity (S16) |
| Learning | -10..+10 | Feedback boosts strong, swipe nudges weak (S29) |
| Novelty | -5..+5 | Recently-passed venues down-ranked; sign flips for "low-key & familiar" (S17) |

The card's displayed **fit score** is the total normalized to a percentage. Tie-breaks and the diversity shuffle use the session seed, so everyone in a session sees the same deck order (S13, S43): the mechanism match sessions run on. Group decks (kind = match) skip the learning and rotation components and score from the host's story + area frame, roster-unioned vetoes, and venue quality (D9).

### 6.3 Swipe session (S35, S37, S38)

```mermaid
flowchart TD
    A[Deck loaded, first card full screen] --> B{Gesture or button}
    B -->|Left / pass| C[Log swipe, next card, prefetch card n+3 images]
    B -->|Right / shortlist| D[Log swipe, subtle shortlist count tick, next card]
    B -->|Tap| E[Detail sheet: gallery, blurb, hours, tier, menu, book or call, flag]
    E -->|Dismiss| B
    E -->|Book now| H
    B -->|Undo| F[Reverse last swipe, restore card]
    C & D --> G{Cards left?}
    G -->|Yes| B
    G -->|No| I["End card: passed n, shortlisted m; review shortlist / widen filters / expand radius"]
    I --> H[Shortlist compare view]
    H --> J["Pick winner or dinner + drinks pair -> plan created"]
    J --> K["Plan screen: book / call / directions / menu, cached offline"]
```

Swipes batch-post to the API (fire-and-forget with local queue) so gesture latency never waits on the network.

### 6.4 Availability states (S19, S21, S23): simplified in v6

```mermaid
flowchart TD
    A[Venue surfaced: deck, detail, map, or plan] --> B{Open tonight per posted schedule? Nightly refresh J1}
    B -->|No| CL["CLOSED: excluded from tonight's deck; labeled plainly anywhere else it appears"]
    B -->|Yes| C{Reservation check fresh within 30 min found seats?}
    C -->|Yes| RS["OPEN, RESERVATIONS AVAILABLE: badge + one-tap booking, date/time/party prefilled"]
    C -->|No| WI["OPEN, WALK-IN OR CALL: 'try walking in or call the restaurant'; platform link in detail sheet without a claim"]
```

**The determination process (v6):**

1. **Two data sources, nothing else.** The nightly refresh (J1) pulls each venue's posted schedule: that alone decides Closed vs Open. Reservation-platform checks (OpenTable and peers, when access exists) decide whether verified seats exist tonight. No manual entry, no user-feedback signal, no pattern inference anywhere in availability.
2. **Checks are demand-driven, never blocking:** deck generation and detail-sheet opens enqueue reservation checks for surfaced venues whose latest check is older than 5 minutes (J5). Screens render from cache instantly; the reservations badge appears in place when fresh data lands (S51).
3. **State math at read time**, per venue for tonight's window and party size: not open per schedule -> **Closed**. Open + a check fresh within 30 minutes found seats -> **Open, reservations available**. Open + anything else (walk-in venue, fresh check found none, no access, check failed) -> **Open, walk-in or call**. A fresh no-seats result sharpens the detail copy ("no online reservations left tonight") but stays within the same state.
4. **What we never do:** claim seats from stale data, guess from history, or move a venue's rank because of availability. Boundary tests pin the thresholds (29:59 vs 30:01 assertion window, 5:00 refresh trigger; section 12).

MVP reality by phase: before partner access there is no reservations badge anywhere; every open venue reads "walk in or call" with its booking link one tap away, which is true and still useful. After access, the badge lights up for partner-bookable venues while someone is actually browsing them.

### 6.5 Group match session (S41..S48): revised in v5, fully async

```mermaid
flowchart TD
    A["Host creates session: roster or share link, area + story + planned night, close time (1h / 3h / this evening / 24h)"] --> B["Deck generated once at creation: host frame + invited participants' vetoes, lowest budget ceiling, session seed"]
    B --> C["Everyone swipes on their own schedule, any time before the deadline"]
    C --> D{Someone joins via link?}
    D -->|Yes| E["Their vetoes disqualify venues for everyone: deck shrinks monotonically, never reorders; prior swipes on removed venues are discarded and disclosed at results"]
    E --> C
    D -->|No| C
    C --> F{Deadline passes or host closes early}
    F --> G["Session closed: evaluated on read, exact to the minute"]
    G --> H["Results materialize: leaderboard by right-swipe count, fit tie-break; unanimous-among-swipers highlighted; participation stats"]
    H --> I["Host locks a pick -> plan for the roster, party size prefilled; large parties default to the call action"]
```

Results are computed server-side only (trust boundary), once, at close: rank venues by count of participants whose latest swipe is right, fit score breaking ties, venues right-swiped by every actual swiper flagged unanimous, veto-disqualified venues excluded regardless of votes. Undo is unrestricted until close; swipes and undos after close are rejected. Clients poll `GET /api/sessions/:id` (~7s) for progress while the session screen is open; results reach closed apps via next-open banners (push is the top fast-follow), and in practice the host announces the winner in the group's own text thread.

## 7. Blend engine (PRD 2.1, S9..S12): unchanged from v1

Pure function: `(quizA, quizB | null, rotationState) -> Blend`. The most heavily unit-tested module.

1. **Vetoes:** `dietary = union(A, B)`, `accessibility = union(A, B)`, `budget_ceiling = min(A, B)`. Hard filters only, never scored, never relaxed, and user filters (S36) cannot override them.
2. **Archetype:** equal -> keep. Different -> `THIRD_WAY[a][b]`, a curated 5x5 matrix returning 1 or 2 blend zones. Content deliverable before alpha.
3. **Neighborhood:** intersection if non-empty, else union with rotation primacy deciding tonight's lean, shown openly.
4. **Radius:** `min(A, B)` with an "expand if needed" toggle feeding the relax ladder.
5. **Structure:** narrowed Q5 (dinner only / dinner then drinks / drinks then dinner / just drinks). Disagreement = deck includes both types with the lean shown.
6. **Compatibility score:** weighted overlap (archetype 30, budget 20, neighborhood 20, radius 15, structure 15). Shown on the blend screen only, never on cards.
7. **Solo:** blend = A's profile + `solo` flag; partner-joins-later recomputes (S6).

## 8. Card content

Front of card (the whole interface while swiping):
- Full-bleed photo (first of 3 to 5, admin-picked).
- Name; one metadata line: cuisine or drink tag, neighborhood, price tier, distance from user.
- Two numbers max: rating (Google+Yelp aggregate) and fit %.
- One-line blurb: the `why_tonight` fragment for the blend's archetype (weather/season variant if tagged).
- Reservations badge only when a fresh check verified seats; nothing else makes an availability claim on the card front.

Detail sheet (tap): photo gallery, full blurb, hours tonight, availability line, menu button (in-app browser on `menu_url`), book or call, directions, accessibility and dietary icons, flag action.

Authoring workflow: admin picks a venue, hits "Draft with Claude" (venue facts + archetype voice guide in, 3 drafts out), edits, approves. 100 venues x ~5 archetypes is ~500 blurbs; drafting assistance makes this days of editorial work, and it is the Phase 1 content-critical path.

## 9. UI design principles (low cognitive load, glossy)

These are testable rules, not taste. A screen that violates one needs a written reason.

1. One primary decision per screen. The deck asks exactly one question: this place, tonight, yes or no.
2. Max 5 information elements on a card front (photo, name, meta line, scores, blurb). Everything else is progressive disclosure behind the tap.
3. Two numbers max per card. Rating and fit. No review counts, no distances-to-the-decimal, no badge soup.
4. Photography is the interface: full-bleed images, text on a bottom scrim gradient, no chrome while swiping.
5. Dark-first palette (the app is used on evenings out), one accent color, two type scales per screen, 8pt spacing grid.
6. Motion communicates state: spring physics on swipe, haptic tick on decisions, no decorative animation.
7. Gestures always have visible button equivalents (S35). Accessibility is not a mode.
8. Empty and thin states are honest and specific ("7 places open tonight that fit"), never fake-infinite.

## 10. API surface

All routes rate-limited (Upstash sliding window per IP + profile UUID). zod validation at every boundary, schemas shared with the client. No PII in logs.

| Route | Purpose |
|-------|---------|
| `POST /api/profiles` | Create anonymous profile (UUID stored in MMKV client-side) |
| `POST /api/quiz` | Submit quiz + vetoes |
| `POST /api/couples` | Create couple (pass_phone, async, solo) |
| `POST /api/invites` / `GET /api/invites/:token` / `POST /api/invites/:token/accept` | Async pairing lifecycle; GET serves the web quiz page (S2..S4) |
| `POST /api/blends/compute` | Recompute blend (also server-triggered on quiz submit) |
| `POST /api/decks/generate` | The hot path: story + filters in, ranked card deck out |
| `POST /api/swipes` | Batched swipe events (fire-and-forget from client queue); carries swiper identity; rejected after session close |
| `POST /api/friends/invites` / `POST /api/friends/invites/:token/accept` | Add-friend-via-link lifecycle (S47) |
| `GET /api/friends` / `DELETE /api/friends/:id` / `POST /api/friends/:id/block` | Friends list, remove, block (S47) |
| `POST /api/sessions` / `POST /api/sessions/:token/join` / `POST /api/sessions/:id/close` | Match session lifecycle: create with roster, frame, and close time; join any time before the deadline; host early-close (S41, S42, S45) |
| `GET /api/sessions/:id` | Session state: roster, progress, viewer-filtered deck, results once closed. Close is evaluated on read, so results appear the moment the deadline passes (S44, S45) |
| `POST /api/sessions/:id/lock` | Host-only: convert a result into the plan (D9) |
| `POST /api/plans` / `GET /api/plans` | Create from shortlist pick; list for couple sync (S38) |
| `POST /api/plans/:id/feedback` | Would-repeat + moods (S27), accepts offline replays |
| `GET /api/venues/:id` | Detail-sheet payload |
| `GET /api/venues/:id/availability` | Current state from Redis (Closed / walk-in / reservations); enqueues a background reservation check when stale. Never blocks on an external call (S51) |
| `POST /api/flags` | Content flag (S34) |
| `GET /api/export` / `POST /api/delete` | Data portability + erasure (S33) |
| `/admin/*` | Venue CRUD (incl. photo picks + menu_url), blurb authoring with Claude drafts, availability overrides, event flags, flag queue, learning-weight inspector. Supabase Auth + allowlist. |

## 11. Non-functional requirements

### Interaction performance (design goal 1)
- Gestures on the UI thread via reanimated worklets; the JS thread is never in the swipe's critical path.
- Images: prefetch next 3 cards (expo-image), blurhash placeholders, explicit dimensions. Target: no visible loading state during normal swiping (S40).
- Swipe logging is async-batched; a dead network cannot add gesture latency.

### Deck generation (p95 < 2s)
- Hot path = 2 or 3 Postgres queries over at most 100 venues + Redis reads. Target p95 server time under 300ms; the rest absorbs mobile network latency.
- `Server-Timing` headers + a p95 counter in `metrics_daily` from day one; the 2s number is a PRD commitment.

### Offline (S25, S26)
- Saved plans snapshot into MMKV at save time; their photos are already in the expo-image disk cache.
- Feedback and swipes queue locally and replay on reconnect.
- tel:, maps, and menu deep links are OS-level; menu needs connectivity and degrades to the cached phone number.

### Accessibility (PRD section 9, release gate)
- Every gesture has a button twin (S35); cards expose accessibility labels reading name, cuisine, rating, fit, and blurb in one utterance; detail sheet is a proper focus trap.
- Dark-first palette maintains 4.5:1 body contrast; font scaling honored to 200% (RN `allowFontScaling`, layout tested at max).
- Alt text is a required admin field per photo (can't publish without it).
- Release gate: manual TalkBack pass over quiz, deck, detail, plan; automated checks via eslint-plugin-react-native-a11y in CI (RN's automated a11y tooling is weaker than web axe, so the manual pass is the real gate).

### Security and data governance
- Anonymous UUIDs, no PII server-side pre-account (D3). Device location, if granted, is used on-device for distance display only and never sent to the server.
- Account holders carry the product's only PII: auth email (Supabase) and display name (shown in lobbies and friends lists). Export and delete cover both, plus friendships, sessions, and swipes (S33). Friends are added by link, never by contact upload; the server never sees an address book.
- Every session and friend action is authorization-checked server-side: participant membership and open-session status on swipe ingest, host identity on close/lock, friendship status on session invites. Blocked users cannot rejoin or re-add (S47).
- Rate limiting from day one; zod at every boundary; sanitized errors; fail closed on auth errors. Social caps: rosters of 10, friend-invite issuance throttled per user, session join tokens dead after roster lock.
- Secrets in env vars only; `.env.example` committed, `.env` and `*.jks` (signing keystore) gitignored; gitleaks pre-commit hook when the repo initializes. Google Maps Android key restricted by package name + SHA-1; partner API credentials server-side only, never shipped in the APK.
- APK signing keystore backed up outside the repo (losing it means testers reinstall from scratch).

### Analytics (privacy stance)
- No third-party analytics SDK in the app. Server-side aggregate counters only.

| PRD metric | Instrumentation |
|------------|-----------------|
| 40% of first-time users book or save within 7 days | Counter: outbound booking tap OR plan created, keyed by profile-week bucket, aggregated |
| Action in under 5 minutes | Timer from session start to first plan/booking tap, aggregate histogram |
| 60% "would repeat" | `feedback.would_repeat` ratio |
| 99% booking-integration uptime | Availability provider health checks + cron alerting |
| p95 < 2s deck generation | Server-Timing percentiles |
| (new) Deck engagement | Aggregate swipe-through rate and shortlist rate per deck, no per-user drill-down |

## 12. Testing strategy

- **Blend engine:** table-driven unit tests: veto unions, budget min, all 20 divergent archetype pairs, rotation advancement, solo recompute (S6, S8..S12).
- **Deck engine:** fixture city of ~15 synthetic venues (restaurants + bars); tests for scoring, filter stacking (filters never bypass vetoes), determinism (same seed, same deck), relax ladder order, thin-deck path, novelty decay (S12..S17, S36, S37).
- **Availability states:** closed-by-schedule edges (late close, midnight rollover), seats-assertion freshness boundary (29:59 vs 30:01), refresh trigger at 5:00, and no-access / check-failure degradation to walk-in-or-call (S19, S21, S23, S51).
- **Match engine:** table-driven: results at exact close time (read-time evaluation, no cron dependency), host early-close, leaderboard order + fit tie-break, unanimity-among-swipers highlight, late-join veto disqualification (deck shrinks monotonically, prior swipes on removed venues discarded and disclosed), undo honored up to close, swipes rejected after close and from non-participants, roster caps at 2 and 10, lowest ceiling (S41..S48).
- **Mobile e2e (Maestro):** onboarding modes, swipe-shortlist-plan happy path, undo, filter apply/clear, airplane-mode plan reading, feedback replay on reconnect (S1, S5, S25, S27, S35..S38).
- **Admin e2e (Playwright):** venue CRUD with required alt text and menu_url, blurb approve flow.
- **A11y:** eslint a11y rules in CI + manual TalkBack pass per release (the gate).

## 13. Build and distribution (alpha)

1. Monorepo (pnpm workspaces): `apps/mobile` (Expo), `apps/web` (Next.js API + admin), `packages/shared` (zod schemas, types, scoring constants).
2. APK: EAS Build `preview` profile (or local `expo prebuild` + Gradle when avoiding build-queue limits). Output is a signed, installable APK.
3. Signing: one release keystore, generated at milestone 1, never in git, backed up to the password manager.
4. Install: testers enable "install unknown apps", install from the EAS internal-distribution link or a shared file.
5. Iteration: JS/content changes ship via `expo-updates` OTA on app restart; new APK only for native dependency changes.
6. Deferred to the launch-strategy conversation: Play Store listing, iOS build, public web client.

## 14. Build plan (Phase 0, roughly 6 milestones)

1. **Scaffold:** monorepo, CI (lint, typecheck, test, a11y lint), Drizzle schema + migrations, seed script with the alpha venues (restaurants AND bars from day one so the drinks filter is real), first sideloadable hello-world APK to prove the pipeline.
2. **Onboarding + blend:** quiz UI (5 questions + hard-requirements step), anonymous profiles, pass-the-phone flow, blend engine with its full test table, invite-landing web quiz.
3. **The deck:** generation endpoint, swipe UI with gesture physics and image prefetch, filters, card fronts. Blurbs authored for the alpha venues (editorial sprint alongside). This is the milestone that proves or kills the UI bet, so it comes before availability and feedback polish.
4. **Detail, shortlist, plans:** detail sheet (gallery, menu in-app browser, booking deep links), shortlist compare, plan creation, offline snapshot.
5. **Availability + feedback:** the three-state model (schedule-based Closed/Open + partner-ready reservation checks), demand-driven check worker against a mock feed, reservations badge + walk-in/call guidance, prefilled booking deep links, feedback prompt, learning weights. Partner API applications go out at milestone 1 so approval lag overlaps the build.
6. **Pairing + polish:** async invite lifecycle with 48h sweep, metrics counters, TalkBack pass, OTA channel setup, alpha install on both phones.
7. **Accounts + friends:** Supabase Auth, anonymous-history migration on signup, display names, add-friend-via-link, remove/block, export/delete extended to social data.
8. **Match sessions:** deadline lifecycle (create / join / close-on-read), group deck rules (veto union at creation, monotonic disqualification on join, lowest ceiling, host frame), results materialization + leaderboard UI, session deep links, progress polling, plan handoff. First group test: James + spouse + friends.
9. **Explore map:** react-native-maps toggle, pins over the live deck pool, bottom card carousel with pass/shortlist parity, location-optional centering (S49, S50).

The couple alpha can ship after milestone 6; matching (7 and 8) lands right behind it. That sequencing keeps the core loop validating on real date nights while the social layer is built.

Phase 1 adds: 100-venue catalog + blurbs, beta-couple and friend-group APK distribution.

## 15. Open questions and risks

1. **Photo licensing.** Google Places photos must be served via Google's media endpoint with attribution, and long-term storage is restricted; Yelp photos require attribution and link-back. Plan: hot-link per ToS with nightly URI refresh + on-device caching, attribution rendered on the gallery. Confirm compliance details before Phase 1; fallback for alpha is hand-collected photos for 25 venues.
2. **Partner availability access is the MVP's one true external dependency** now that live tiers are in scope. Candidates, roughly in order of plausibility: OpenTable affiliate/partner program, Yelp Guest Manager, SevenRooms, Tock, Resy (Amex) partner relations. Applications out at milestone 1; the pipeline runs against a mock feed until credentials exist. Real risk that none approve for a pre-launch prototype: the reservations state then simply never appears, and every open venue shows walk-in/call guidance with its booking link, which is also the permanent fallback if access is later revoked.
3. **Third-way matrix content**: 20 archetype pairings need curated blend-zone definitions before alpha (S10).
4. **Blurb authoring throughput**: ~500 approved blurbs for Phase 1; Claude-drafted, human-approved; needs the editorial contributor budgeted in the PRD.
5. **Menu URL coverage**: some Hartford spots only have PDF menus or Facebook pages. Curation rule needed: what counts as an acceptable `menu_url`, and the fallback chain (S39).
6. **Yelp Fusion pricing**: Yelp has been moving Fusion toward paid plans. Load is tiny (one match call per venue, ~100 refresh calls nightly), but confirm tier and cost before milestone 1; fallback is the manual curation-time check.
7. **EAS build quotas**: free tier has monthly build limits; local Gradle builds are the pressure valve. Decide at milestone 1.
8. **Weather-radius interaction** (S15): shipped as a scoring nudge, not a hard cut. Validate in a real Hartford winter.
9. **Close-time defaults**: the deadline is now the core matching mechanic. Validate the preset list (1h / 3h / this evening / 24h) and the default with the first friend group, and whether the unanimity highlight at results is enough of a moment without live popups.
10. **Notification gap**: no push in MVP, so matching runs on ~7s polling while the session screen is open plus next-open banners. Workable for same-evening decisions; if sessions stall waiting on closed-app friends, expo-notifications is the first fast-follow.
11. **Group booking reality**: parties of 7+ often can't book via OpenTable/Resy deep links at all. The plan screen defaults large rosters to the call action; worth validating how that feels.

---

## Appendix A: Deferred multi-act itinerary engine (design essence, for when it returns)

Preserved so the v1 thinking isn't lost when the aspiration (PRD section 3: full-evening orchestration) comes back:

- Itinerary = ordered acts (dinner anchor + add-ons) selected from the same filter/score core that now powers the deck; add-ons chosen within a drive-time budget using a precomputed `venue_distances` table (drive/walk minutes per venue pair, filled at curation time via Google Routes API, pairs limited to 30 min crow-flies).
- Time-block assembly: arrival default 7pm, 90 min dinner, drive + 10 min park buffer between acts.
- Narrative rendering: title seed + archetype opening line + per-act `why_tonight` fragments + computed logistics footer (total travel, parking notes, dress code, weather note, availability aggregate "2 of 3 stops confirmed": retired scenario S24).
- Fragment kinds `title_seed` and `opening` return to `venue_fragments.kind` at that point.
- The deck then becomes the browsing surface and itineraries become a composition layer on top: swipe to build your own evening, or accept a suggested arc.
