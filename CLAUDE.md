# Where2Eat

Hartford, CT date-night app: quiz -> couple "blend" -> swipe deck of curated restaurant and bar cards (photos, fit score, menu, booking). Group swipe matching (2 to 10 people, account-gated) settles where friends eat. Prototype phase, unmonetized by design.

## Status

Design phase, v3 (swipe pivot + group matching). No code yet. Docs are the deliverable so far.

v2 pivot decisions (folded into PRD v2.2): Android app via sideloaded APK for alpha; restaurants + bars only; swipe-deck UX (Tinder/Hinge-style) instead of multi-act narrative itineraries, which are deferred; minimalist low-cognitive-load design as a release gate.

v3 decision (folded into PRD v2.3): group swipe matching in MVP: account-gated, rosters of 2 to 10, lobby model, unanimous match + leaderboard fallback, host locks the plan. Accounts (Supabase Auth) move into MVP build scope to gate it.

v4 decisions (folded into PRD v2.4): Explore map in MVP (pins over the deck pool, D10) and live reservation availability in MVP via demand-driven partner polling (D4). Partner API access is the MVP's one external dependency: applications at milestone 1, mock feed until credentials, manual + pattern tiers carry alpha regardless.

v5 decision (folded into PRD v2.5): matching is fully async. No lobby; host sets a close time; everyone swipes on their own schedule; results materialize at close as a leaderboard (unanimity highlighted); late joiners' vetoes shrink the deck monotonically. No live match popups.

## Key files

- `W2E_PRD_Hartford_Prototype.md`: the PRD (v2.3; version history in the doc header and git). Requirements source of truth. Stable filename on purpose: version bumps no longer rename the file.
- `docs/customer-journeys.md`: journey map, scenario catalog (S1..S52, S24 retired), PRD gap resolutions and deviations, scope cuts.
- `docs/technical-design.md`: architecture decisions (D1..D10), stack, data model, flow diagrams, card/UI design principles, match engine, availability tier process, API surface, build and sideload distribution plan. Appendix A preserves the deferred multi-act itinerary design.

Scenario IDs (S-numbers) and decision IDs (D-numbers) are the shared vocabulary; reference them in commits and issues. IDs are stable and never reused.

## Planned stack (per technical design)

- Mobile client: Expo (React Native, TypeScript), gesture-handler + reanimated swipe deck, expo-image, NativeWind, MMKV, expo-linking deep links, expo-updates OTA. Distributed as sideloaded APK for alpha.
- Backend + admin: Next.js on Vercel (API routes, curation admin, invite/session landing pages); Supabase Postgres + Auth (accounts gate matching) + Drizzle; Upstash Redis (availability cache, rate limiting).
- External, curation-time only: Google Places, Yelp Fusion, NWS weather. No external API calls on the request path.
- Monorepo: pnpm workspaces (`apps/mobile`, `apps/web`, `packages/shared`).

## Load-bearing design rules

- The swipe must feel native: gestures on the UI thread, images prefetched, no spinners mid-deck (D2). This is why the client is React Native, not a WebView wrapper.
- Card copy is pre-authored per venue x archetype (D1). No LLM on the request path; p95 < 2s depends on this.
- Vetoes (dietary, accessibility, budget ceiling) are hard filters: never scored, never relaxed, never overridable by user cuisine/drinks filters (S12, S36).
- Decks are deterministic per session (seeded): every roster member sees the same order; match sessions run on this (S13, S43).
- Group matching (D9, async as of v5): account-gated, rosters 2..10, host-set close time. Deck generated once at creation (host frame + invited vetoes, lowest ceiling); a link-joiner's vetoes shrink the deck monotonically, never reorder it. Results materialize at close (evaluated on read): leaderboard by right-count, fit tie-break, unanimity highlighted. Only the host locks a pick into the plan. Friends via invite link only; no chat, no search, no live match popups.
- Anonymous-first: opaque UUID profiles server-side, zero PII pre-account (D3). No device fingerprinting. Location never leaves the device.
- Availability tiers computed at read time from snapshot freshness (D4): pattern + manual writers always on, partner feeds demand-driven (refresh only what users are browsing) once access is granted. Never block a screen on an external call; stale "full" decays to Tier C after 60 min.
- Explore map (D10) is a projection of the deck, not a second engine: pins show exactly the current candidate pool (vetoes + filters respected), and map actions are normal swipes.
- Every gesture has a visible button twin; TalkBack pass gates each release (S35).
- Deferred: multi-act itineraries, map layers beyond pins (routes, heatmaps, weather overlay), event APIs, push notifications (top fast-follow for matching), group chat (never in prototype), friend search (link-only adds), booking-completion tracking, iOS, Play Store, public web client.

## Secrets posture

No secrets exist yet. When code starts: `.env` gitignored, `.env.example` committed, gitleaks pre-commit hook before first commit. APK signing keystore (`*.jks`) never in git, backed up to the password manager.

## Commands

None yet. Will be populated at build milestone 1 (scaffold).
