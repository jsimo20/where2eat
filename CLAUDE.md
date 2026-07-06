# Where2Eat

Hartford, CT date-night app: quiz -> couple "blend" -> swipe deck of curated restaurant and bar cards (photos, fit score, menu, booking). Prototype phase, unmonetized by design.

## Status

Design phase, v2 (swipe pivot). No code yet. Docs are the deliverable so far.

v2 pivot decisions (folded into PRD v2.2): Android app via sideloaded APK for alpha; restaurants + bars only; swipe-deck UX (Tinder/Hinge-style) instead of multi-act narrative itineraries, which are deferred; minimalist low-cognitive-load design as a release gate.

## Key files

- `W2E_PRD_Hartford_Prototype_v2.2.md`: the PRD (v2.2, amended for the swipe pivot). Requirements source of truth.
- `docs/customer-journeys.md`: journey map, scenario catalog (S1..S40, S24 retired), PRD gap resolutions and deviations, scope cuts.
- `docs/technical-design.md`: architecture decisions (D1..D8), stack, data model, flow diagrams, card/UI design principles, API surface, build and sideload distribution plan. Appendix A preserves the deferred multi-act itinerary design.

Scenario IDs (S-numbers) and decision IDs (D-numbers) are the shared vocabulary; reference them in commits and issues. IDs are stable and never reused.

## Planned stack (per technical design)

- Mobile client: Expo (React Native, TypeScript), gesture-handler + reanimated swipe deck, expo-image, NativeWind, MMKV, expo-updates OTA. Distributed as sideloaded APK for alpha.
- Backend + admin: Next.js on Vercel (API routes, curation admin, invite-landing web quiz); Supabase Postgres + Drizzle; Upstash Redis (availability cache, rate limiting).
- External, curation-time only: Google Places, Yelp Fusion, NWS weather. No external API calls on the request path.
- Monorepo: pnpm workspaces (`apps/mobile`, `apps/web`, `packages/shared`).

## Load-bearing design rules

- The swipe must feel native: gestures on the UI thread, images prefetched, no spinners mid-deck (D2). This is why the client is React Native, not a WebView wrapper.
- Card copy is pre-authored per venue x archetype (D1). No LLM on the request path; p95 < 2s depends on this.
- Vetoes (dietary, accessibility, budget ceiling) are hard filters: never scored, never relaxed, never overridable by user cuisine/drinks filters (S12, S36).
- Decks are deterministic per couple + date (seeded), which the future couple-match feature depends on (S13).
- Anonymous-first: opaque UUID profiles server-side, zero PII pre-account (D3). No device fingerprinting. Location never leaves the device.
- Availability tiers computed at read time from snapshot freshness (D4). Tier C is the MVP workhorse.
- Every gesture has a visible button twin; TalkBack pass gates each release (S35).
- Deferred: multi-act itineraries, explore/map mode, Tier A live availability, event APIs, couple swipe-matching, iOS, Play Store, public web client.

## Secrets posture

No secrets exist yet. When code starts: `.env` gitignored, `.env.example` committed, gitleaks pre-commit hook before first commit. APK signing keystore (`*.jks`) never in git, backed up to the password manager.

## Commands

None yet. Will be populated at build milestone 1 (scaffold).
