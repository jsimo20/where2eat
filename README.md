# Where2Eat

Date-night decision app for Hartford, CT. Take a 2-minute quiz with your partner, get a "blend" of your combined tastes, then swipe through a curated deck of restaurant and bar cards picked for tonight: big photos, a fit score, the menu one tap away, book or call straight from the card.

**Status:** design phase. No code yet; the documents are the current deliverable.

## Docs

- [Product requirements (PRD)](W2E_PRD_Hartford_Prototype_v2.1_1.md)
- [Customer journeys and MVP scenario catalog](docs/customer-journeys.md)
- [Technical design](docs/technical-design.md)

## Planned stack

Expo (React Native) Android client distributed as a sideloaded APK for alpha; Next.js API + curation admin on Vercel; Supabase Postgres + Drizzle; Upstash Redis. External APIs (Google Places, Yelp Fusion, NWS weather) touch the system at curation time only, never on the request path.

## Prototype scope

Restaurants and bars in the Hartford metro, hand-curated (25 venues for alpha, 100 for beta). Unmonetized by design until the core experience is validated.
