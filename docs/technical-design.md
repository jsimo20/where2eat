# Where2Eat: Technical Design (MVP)

Covers Phase 0 alpha through Phase 1 of [the PRD](../W2E_PRD_Hartford_Prototype.md) (v2.3: Android APK alpha, restaurants + bars, swipe deck, group matching, minimalist design).
Scenario IDs (S1..S48) reference [customer-journeys.md](customer-journeys.md). Decision IDs (D1..D9) are stable across revisions. v3 adds account-gated group swipe matching (rosters of 2 to 10) to MVP.

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

### D4. Availability is a provider abstraction with tiers computed at read time (unchanged)

- **Decision:** An `AvailabilityProvider` interface with pluggable sources: `ManualOverride` (alpha), `PatternProvider` (curated hours + day-of-week heuristics), later `PartnerProvider` (OpenTable/Resy if a partnership lands). Snapshots land in Redis and Postgres; the tier (A/B/C/D) is derived from snapshot freshness and source at read time (PRD 4.1). Tiers render as card badges and drive the primary action (book vs call).
- **Why:** No prototype gets OpenTable/Resy availability APIs. Ships the full tier UX now, running mostly on Tier C, and slots partner APIs in later without touching the UI. Scraping rejected: ToS risk for a partnership-dependent product.

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

### D9. Group matching: lobby model, unanimous match, leaderboard fallback (new in v3)

- **Decision:** A match session is host-created and account-gated. Invitees gather in a lobby (from the friends list or a share link); the host starts the session, which locks the roster (2 to 10). Everyone swipes the same seeded deck; the server evaluates on every swipe ingest, and a venue with a right-swipe from every locked participant surfaces as a match. Matches are immutable once surfaced (a later undo adjusts counts, never revokes the celebration). If nothing goes unanimous, the host ends the session and gets a leaderboard ranked by right-swipe count with fit score as tie-break. Only the host converts a match or leaderboard pick into the plan. Friends are added by invite link only; there is no chat, no search, no contact upload.
- **Why each piece:** the lobby lock kills every join-after-match consistency bug in one move. Unanimity is the only match rule that needs no explanation and no arguing; the leaderboard is the honest outcome for rosters of 6+, where unanimity gets rare. Host authority means one person books (a table for 8 doesn't need a committee). Link-only friends and zero free-text keep the abuse/moderation surface near nil at alpha scale.
- **Rejected:** majority-vote thresholds (arbitrary and argues with itself), live-only synchronous sessions (friends trickle in over an hour), client-computed matches (match logic belongs inside the trust boundary), group chat (the group already has one; it's called their text thread).

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
| Deep links | expo-linking + Android App Links | Friend and session invites open straight into the lobby (S42). |
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
        MATCH[Match sessions: friends, lobby, progress poll, results]
        PLANS[Shortlist + plans, offline via MMKV + image cache]
        OTA[expo-updates OTA channel]
    end

    subgraph Web["Next.js on Vercel"]
        RL[Rate limit middleware]
        API[API routes: profiles, quiz, couples, invites, blends, decks, swipes, sessions, friends, plans, feedback, flags, export]
        INV[Invite landing pages: partner quiz, session join]
        ADM[Admin: venue CRUD, fragment authoring, photo picks, overrides, event flags, flag queue, learning inspector]
    end

    subgraph Data["Data layer"]
        PG[(Postgres via Supabase: venues, fragments, profiles, couples, blends, sessions, swipes, plans, feedback)]
        RD[(Upstash Redis: availability snapshots, weather cache, rate limits)]
    end

    subgraph Jobs["Cron jobs (Vercel Cron)"]
        J1[Nightly: Places + Yelp refresh, hours and closed checks, photo URIs, curation-bar and quality alerts]
        J2[Hourly: NWS weather pull for Hartford]
        J3[Hourly: invite expiry sweep at 48h]
    end

    subgraph Ext["External (never on the request path except deep links)"]
        GP[Google Places API]
        YF[Yelp Fusion API]
        NWS[NWS weather API]
        CL[Claude API: blurb drafting]
        BK[Booking deep links: OpenTable, Resy, tel:]
        MP[Maps deep links + in-app menu browser]
    end

    DECK & MATCH & PLANS --> RL
    INV --> RL
    RL --> API
    API --> PG & RD
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
    VENUES ||--o{ AVAILABILITY_SNAPSHOTS : has
    PROFILES ||--o{ QUIZ_RESPONSES : submits
    COUPLES ||--o{ INVITES : issues
    PROFILES }o--o{ COUPLES : members
    COUPLES ||--o{ BLENDS : computes
    SESSIONS ||--o{ SWIPES : records
    PROFILES ||--o{ SWIPES : casts
    VENUES ||--o{ SWIPES : targets
    SESSIONS ||--o{ SESSION_PARTICIPANTS : seats
    USERS ||--o{ SESSION_PARTICIPANTS : joins
    SESSIONS ||--o{ MATCHES : surfaces
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
- **sessions**: `id, kind (solo|couple|match), couple_id (nullable), host_user_id (nullable), city_id, story_input (jsonb: archetype, filters, area, party_size), seed, status (open|started|ended|expired), planned_for, created_at`. One table for every swiping context; the seed makes each deck deterministic and replayable (S13, S43). Match sessions follow the lobby lifecycle (D9) and expire at the end of the planned night (S45).
- **session_participants**: `session_id, user_id, state (invited|joined), joined_at`. Locked at host start; swipes from outside the locked roster are rejected (S41).
- **matches**: `id, session_id, venue_id, matched_at`. One row per unanimous venue, immutable once surfaced (S44); the host locks one into the plan.
- **friendships**: `id, user_low_id, user_high_id, status (active|blocked), blocked_by (nullable), created_at`. Created only via accepted invite links; canonical low/high ordering prevents duplicate pairs (S47).
- **friend_invites**: `id, inviter_user_id, token (unique), status, expires_at, created_at`. Same share-sheet pattern as partner invites.
- **swipes**: `id, session_id, profile_id, venue_id, direction (left|right|undo), position, created_at`. Carries the swiper: match evaluation runs on ingest, and learning nudges accrue to the individual (D6, D9). Powers dedupe (S17) too. No dwell-time or other surveillance-shaped fields.
- **plans**: `id, couple_id (nullable), session_id (nullable), venue_ids[] (one, or dinner + drinks pair), planned_for (date), party_size, solo_planned (bool), status (saved|completed_presumed|completed_confirmed|abandoned), snapshot (jsonb: rendered cards at save time), created_at`. Group plans come from a locked match session with party size prefilled from the roster; the snapshot keeps a saved plan immutable even if venue data changes later (S25).
- **feedback**: `id, plan_id, profile_id, would_repeat (bool, nullable), moods[], created_at` (S27). Group plans prompt every participant; each answer feeds that person's own learning.
- **profile_affinities**: `profile_id, attribute_affinities (jsonb), updated_at`. Swipe nudges accrue to the swiper (D6).
- **couple_learning**: `couple_id, venue_boosts (jsonb), updated_at`. Feedback boosts stay couple-level (D6).
- **availability_snapshots**: `id, venue_id, source (partner|manual|pattern), status, slot_at, party_size, observed_at`. Redis hot copy, Postgres history.
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
    H --> I[Annotate availability tier per card]
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

### 6.4 Availability tier decision (S19..S23): unchanged from v1

```mermaid
flowchart TD
    A[Read freshest snapshot for venue + time window] --> B{Source partner or manual, under 5 min old?}
    B -->|Yes, available| TA["Tier A: 'Available now' badge + one-tap booking"]
    B -->|No| C{Snapshot 5 to 60 min old?}
    C -->|Yes, available| TB["Tier B: 'Last confirmed N min ago' + booking link"]
    C -->|No| D{Known unavailable tonight?}
    D -->|Yes| TD["Tier D: 'unlikely tonight' badge, down-ranked, next-best same-category ranked above (S22)"]
    D -->|No| TC["Tier C: pattern line from hours + history, 'Usually has tables Tue at 7' + tap to call"]
```

Tier is computed at read time from data freshness, never stored as a fact about the venue. MVP reality: Tier C most of the time, B/A only via manual override (alpha) or a future partner feed.

### 6.5 Group match session (S41..S48): new in v3

```mermaid
flowchart TD
    A["Host creates session: roster from friends list or share link, area + story + date, account required"] --> B["Lobby: joiners appear; quiz completion required to join (vetoes needed)"]
    B --> C{Host starts}
    C --> D["Roster locked (2..10). Deck generated: vetoes unioned across roster, lowest budget ceiling, host's area + story, session seed"]
    D --> E[Everyone swipes the same deck independently]
    E --> F{Server, on each swipe ingest: venue right-swiped by all locked participants?}
    F -->|Yes| G["Match surfaced: 'It's a dinner'. Live popup if screen open (state poll ~7s), banner on next open. Immutable once surfaced; keep swiping or lock it in"]
    F -->|No| E
    G --> I
    E --> H{Host ends, deck exhausted for all, or planned night arrives}
    H --> I["Results: matches first, then leaderboard by right-swipe count, fit score breaks ties"]
    I --> J["Host locks a pick -> plan for the roster, party size prefilled; large parties default to the call action"]
```

Match evaluation is server-side only (trust boundary): `matched(venue) := every locked participant's latest swipe on venue is right`. An undo before a match is surfaced removes the right; after surfacing, the match row is immutable and only leaderboard counts move (S44). Clients poll `GET /api/sessions/:id` (~7s) while the session screen is open; there is no push in MVP, so closed-app participants catch up via next-open banners (top fast-follow: expo-notifications).

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
- Tier badge only when it earns attention (available now / unlikely tonight).

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
| `POST /api/swipes` | Batched swipe events (fire-and-forget from client queue); carries swiper identity, triggers match evaluation for match sessions |
| `POST /api/friends/invites` / `POST /api/friends/invites/:token/accept` | Add-friend-via-link lifecycle (S47) |
| `GET /api/friends` / `DELETE /api/friends/:id` / `POST /api/friends/:id/block` | Friends list, remove, block (S47) |
| `POST /api/sessions` / `POST /api/sessions/:token/join` / `POST /api/sessions/:id/start` | Match session lifecycle: create with roster/area/story, join lobby, host-start locks roster (S41, S42) |
| `GET /api/sessions/:id` | Session state: roster, progress, matches, leaderboard. Polled ~7s while the session screen is open (S44, S45) |
| `POST /api/sessions/:id/lock` | Host-only: convert a match or leaderboard pick into the plan (D9) |
| `POST /api/plans` / `GET /api/plans` | Create from shortlist pick; list for couple sync (S38) |
| `POST /api/plans/:id/feedback` | Would-repeat + moods (S27), accepts offline replays |
| `GET /api/venues/:id` | Detail-sheet payload |
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
- Every session and friend action is authorization-checked server-side: roster membership on swipe ingest, host identity on start/end/lock, friendship status on session invites. Blocked users cannot rejoin or re-add (S47).
- Rate limiting from day one; zod at every boundary; sanitized errors; fail closed on auth errors. Social caps: rosters of 10, friend-invite issuance throttled per user, session join tokens dead after roster lock.
- Secrets in env vars only; `.env.example` committed, `.env` and `*.jks` (signing keystore) gitignored; gitleaks pre-commit hook when the repo initializes.
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
- **Availability tiers:** freshness-boundary tests (4:59 vs 5:01 vs 60:01) and outage degradation (S19..S23).
- **Match engine:** table-driven: unanimity detected on the Nth right, undo before vs after surfacing, non-roster swipe rejected, multiple matches, leaderboard order + fit tie-break, host-only start/end/lock, roster caps at 2 and 10, group veto union + lowest ceiling (S41..S48).
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
5. **Availability + feedback:** provider abstraction (manual + pattern), tier badges and D-swap ranking, feedback prompt, learning weights.
6. **Pairing + polish:** async invite lifecycle with 48h sweep, metrics counters, TalkBack pass, OTA channel setup, alpha install on both phones.
7. **Accounts + friends:** Supabase Auth, anonymous-history migration on signup, display names, add-friend-via-link, remove/block, export/delete extended to social data.
8. **Match sessions:** lobby + roster lock, group deck rules (veto union, lowest ceiling, host frame), match engine + results leaderboard, session deep links, state polling, plan handoff. First group test: James + spouse + friends.

The couple alpha can ship after milestone 6; matching (7 and 8) lands right behind it. That sequencing keeps the core loop validating on real date nights while the social layer is built.

Phase 1 adds: 100-venue catalog + blurbs, beta-couple and friend-group APK distribution.

## 15. Open questions and risks

1. **Photo licensing.** Google Places photos must be served via Google's media endpoint with attribution, and long-term storage is restricted; Yelp photos require attribution and link-back. Plan: hot-link per ToS with nightly URI refresh + on-device caching, attribution rendered on the gallery. Confirm compliance details before Phase 1; fallback for alpha is hand-collected photos for 25 venues.
2. **Partner availability APIs** (OpenTable/Resy): almost certainly unavailable to a prototype. Tier C is the workhorse; Tier A UX ships dormant.
3. **Third-way matrix content**: 20 archetype pairings need curated blend-zone definitions before alpha (S10).
4. **Blurb authoring throughput**: ~500 approved blurbs for Phase 1; Claude-drafted, human-approved; needs the editorial contributor budgeted in the PRD.
5. **Menu URL coverage**: some Hartford spots only have PDF menus or Facebook pages. Curation rule needed: what counts as an acceptable `menu_url`, and the fallback chain (S39).
6. **Yelp Fusion pricing**: Yelp has been moving Fusion toward paid plans. Load is tiny (one match call per venue, ~100 refresh calls nightly), but confirm tier and cost before milestone 1; fallback is the manual curation-time check.
7. **EAS build quotas**: free tier has monthly build limits; local Gradle builds are the pressure valve. Decide at milestone 1.
8. **Weather-radius interaction** (S15): shipped as a scoring nudge, not a hard cut. Validate in a real Hartford winter.
9. **Match rule at large rosters**: unanimity gets rare as rosters grow; at 6+ the leaderboard is probably the real product and the match moment is a bonus. Validate with the first friend-group test before investing in match-moment polish.
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
