# Competitive Landscape: Where2Eat

**Date: July 6, 2026.** Competitive intel goes stale fast; re-run this scan before any launch-phase decision. Web-researched (not from model memory); sources at the bottom. Liveness of the small indie apps is unverified beyond their pages resolving today.

**Question this informs:** who else is solving "where should we eat" for couples and friend groups, whether our PRD's value prop (low cognitive load, swipe-first, couples/groups, Hartford) holds up, and what we can build as a durable moat.

**TL;DR:** the swipe-to-decide genre exists and is crowded at the bottom (half a dozen indie clones, none funded, none differentiated), while the funded players (Beli, Corner, Cobble) and the incumbents (Google, Yelp, OpenTable/Amex) all orbit the same job without owning the *decision moment*. Nobody combines group consensus mechanics + real curation + a small metro. The industry is structurally hostile to a national venture play (that is why the genre is a graveyard) but friendly to exactly what we are: an unmonetized, two-person, hyper-local prototype whose moat is curation density and trust, not the swipe.

---

## 1. Competitive set

### Direct: swipe/vote-to-decide apps (same problem, same mechanism)

| App | Platform | Model | What they have | What they lack |
|---|---|---|---|---|
| **Sawa Eats** | iOS + Android | Free forever, session codes | Group sessions, unanimous match, Reddit-sourced city guides, 127 cities | No curation (generic listings), unanimity-only (no fallback when nobody matches), no taste profile, no vetoes |
| **Hangrily** | Web-first, no account | Free | Zero-friction entry ("decide in 2 min"), strong SEO landing pages per use case | No native app feel, no persistence, no learning, no local depth |
| **Food Match, Food Swiper, Food Flip, Tine, SwipeEats, Dishy, Food Flock** | Mostly iOS | Free | Genre filler: swipe + match on Google/Yelp places data | Everything else; indistinguishable from each other |
| **Munch** (2021) | iOS | — | Press coverage at launch (AlleyWatch) | Apparent graveyard case; the genre's usual outcome |

Pattern: every direct competitor is generic-data + unanimity-match + national-by-default. None has raised meaningful money. None curates. None handles dietary/accessibility as hard constraints. None is in Hartford in any meaningful sense.

### Adjacent: funded products circling the same job

| Product | Funding/scale | What they are | Overlap with us |
|---|---|---|---|
| **Beli** | $5.3M; 75M+ reviews logged (claims to have passed Yelp's count); Fast Company 2026 innovative-companies list | Gen Z restaurant ranking/social diary; taste profiles, friend match scores, **Group Dinner AI matching** | The biggest adjacent threat. They own taste data + social graph and are inching toward decisioning. But list-first, big-metro-first, thin in Hartford |
| **Corner** | $3.75M; ~55k users, 450 cities | Map-first social discovery, AI semantic search (built on Claude) | Discovery, not decision. NYC/SF/global-city density |
| **Cobble** | $3M; ~7 major metros | Couples swipe-to-match on *date ideas*; has pivoted toward "AI Group Planner" | Validated the couples-decision mechanic; also demonstrated that hand-curated content stops scaling at big metros. Hartford will never be on their map |
| **The Infatuation** (Chase-owned) | Corporate | Editorial reviews + guides | Big metros only; editorial trust model worth copying at local scale |

### Incumbent features (not products)

- **Google Maps** has had group polling since 2018: share a list, friends thumbs-up/down, "Top Voted" wins. It is buried, list-shaped, and joyless, and Google has not invested in it — but it proves the incumbent *could* occupy the space any time.
- **Yelp** shipped an AI chatbot (April 2026) and licenses data to OpenAI; its Guest Manager reservations business grew 553% since integrating Google Reserve.
- **OpenTable / Resy + Tock (Amex, $400M acquisition)**: own the booking rails we deep-link into. OpenTable lists just **18 restaurants in Hartford** — the big platforms visibly under-serve this market.

### Substitutes (the real competition)

1. **The group chat + Google Maps.** Free, installed, good enough. Our true rival is this default behavior, not any app.
2. **AI chatbots.** ChatGPT/Gemini/AI Overviews are absorbing "where should we eat" as a query. Chatbot referrals to restaurant sites are projected to be the #2 traffic source (behind Google) by end of 2026. AI chat does personalization and dialogue well; it does not do shared group state, a binding consensus ritual, or fresh small-market data.
3. **TikTok/Instagram.** 38% of Gen Z restaurant discovery starts on TikTok; 93% of diners check 3+ digital touchpoints before choosing.
4. **Local editorial.** CTbites, Hartford Magazine, r/Connecticut. Trusted, but static lists — they inform, they don't decide.
5. **Non-consumption.** The couple's rotation of the same three spots. Probably our largest "competitor" by share.

---

## 2. Feature comparison

Our column is **planned, not shipped** — treat it as the spec bar, not a claim.

| Capability | Where2Eat (planned) | Swipe clones (Sawa etc.) | Beli | Google Maps | Cobble | AI chat |
|---|---|---|---|---|---|---|
| Swipe decision ritual | Strong | Strong | Absent | Absent | Strong (date ideas) | Absent |
| Async group consensus w/ deadline + fallback | Strong (leaderboard at close) | Weak (unanimity or nothing) | Weak (AI group match) | Weak (buried poll) | Weak | Absent |
| Couple blend / taste profile | Strong | Absent | Strong | Weak | Adequate | Adequate (per-user) |
| Hard vetoes (dietary, accessibility, budget) | Strong (never relaxed) | Absent | Absent | Weak (filters) | Absent | Adequate (if prompted) |
| Curated small-metro content | Strong (Hartford depth) | Absent | Weak | Weak | Absent (big metros only) | Weak (stale) |
| Verified availability (no guesses) | Strong (3 states) | Absent | Absent | Adequate (hours often stale) | Absent | Weak (hallucination risk) |
| Booking handoff | Adequate (deep links) | Weak | Adequate | Strong | Adequate | Adequate (agents emerging) |
| Social graph / review corpus | Absent by design | Absent | Strong | Strong | Weak | Strong (via licensed data) |
| Coverage breadth | 1 metro | ~100+ cities shallow | National deep in big metros | Global | 7 metros | Global shallow |

The last two rows are where we lose on paper and it does not matter: we are not competing on corpus or coverage. The first six rows are the product.

---

## 3. Porter's Five Forces

Assessed for our arena: consumer dining-decision apps, viewed from a Hartford-first entrant. (Framework note: Porter rates how hostile an industry's economics are via five pressures; a "high" force squeezes profit/viability. Adapted here for an unmonetized prototype, where "profit" reads as "sustained usage and defensibility.")

| Force | Rating | Evidence | Implication for us |
|---|---|---|---|
| **Rivalry among existing competitors** | Low–Moderate | Many direct players, all tiny and undifferentiated; no category king; nobody local to Hartford. Beli is the only rival with real momentum, and it is adjacent, not direct | The category is winnable locally. Watch Beli only |
| **Threat of new entrants** | High | A clone is a weekend project (Places API + swipe UI); the genre respawns constantly | The swipe mechanic is NOT a moat. Defensibility must live in data, content, trust, and local density |
| **Supplier power** | High | Google Places + Yelp Fusion set API pricing/terms; Amex now owns Resy *and* Tock; OpenTable gates partner/affiliate access; eventually Play Store terms | Already partially mitigated by design: curation-time-only API calls, first-party content layer, availability states that degrade honestly without partners. Keep it that way |
| **Buyer power** | High | Users pay nothing, switch instantly, and default to free substitutes; expectations set by TikTok-grade polish | Win on felt quality + trust. Unmonetized-by-design neutralizes the pricing dimension for now |
| **Threat of substitutes** | **Very High (dominant force)** | Group chat + Maps polling; AI chat absorbing discovery queries; TikTok owning inspiration | Do not compete on *recommendations* — that layer is being commoditized by AI. Compete on the *decision ritual* and *verified local truth*, which chatbots and feeds structurally lack |

Net read: hostile industry for a venture-scale national bet (hence the graveyard), tolerable for a hyper-local unmonetized prototype. The two killer forces (entrants, substitutes) weaken sharply at local depth: a clone can copy the swipe in a weekend but not 25 verified Hartford venues with authored content, and an AI chatbot can suggest but cannot host the group's shared decision.

---

## 4. Positioning

Everyone else answers **"what's good?"** We answer **"where are *we* going tonight?"**

- Beli: "For food lovers who want to track and rank everything" (identity/social).
- Corner: "Digitized word of mouth on a map" (discovery).
- Cobble: "Stop saying 'I don't know, you pick'" (decision — closest to ours, but content-shallow outside 7 metros and drifting to AI planning).
- Swipe clones: "Tinder for restaurants" (mechanic-as-positioning, which is why none of them stick).
- Positioning statement for us: *For couples and friend groups in Greater Hartford who waste the best part of the evening deciding, Where2Eat is a decision app that turns "where should we eat" into a settled plan in under two minutes, with places that are verified open, bookable, and safe for everyone's constraints. Unlike national apps with stale listings and buried polls, every venue is locally curated and nothing is ever guessed.*

Unclaimed position we can own: **the honest one.** Every competitor shows stale hours, closed venues, or hallucinated claims; our three-state availability rule and never-relaxed vetoes are a trust posture nobody in the set can match without re-architecting.

---

## 5. Moat assessment (honest ranking)

What is NOT a moat: the swipe UX, the blend quiz, being first. All cloneable in days, and this genre respawns clones annually.

What is buildable as a moat, in order:

1. **Curation density per market.** 25+ verified venues × 5 archetype blurbs × weather variants, verified menu URLs, honest availability. Cheap for us (one market, two people, Claude-assisted authoring), uneconomic for any national player to replicate in market #150. The candidate-venue verification work in `docs/content/` *is* the moat under construction.
2. **Trust as brand.** Vetoes never violated; availability never guessed. In a metro of ~1.2M, word-of-mouth compounds; one "it said open but was closed" erases ten good sessions. This is a policy moat: real only if we never break it.
3. **Local network effects (weak but real).** Group matching is invite-only, so each session seeds accounts inside one metro. Beli proved local density (campuses) beats national breadth. 2–10 person rosters in one city is the same dynamic deliberately scoped down.
4. **Swipe/taste graph per market (long-term).** Accumulated per-profile and couple-level learning (D6) that no entrant has on day one. Only materializes if retention works; do not count on it yet.

### Nightmare scenarios

- **Beli ships "decide tonight" sessions** on top of its taste graph. Defense: they are ranking-first and big-metro-first; their Hartford data is thin; our window is to own the market before it is worth their attention. Watch their releases quarterly.
- **Google surfaces Maps group polling prominently, AI-assisted.** No defense at national scale; local defense is curation + vetoes + ritual that Maps will not build for Hartford. Google's record of abandoning social dining features is long.
- **AI agents book dinner end-to-end** (query → reservation) faster than our flow. Partial defense: group consensus is a coordination problem, not a query; agents have no shared state between four people. Revisit if that changes.

---

## 6. Strategic implications

What this scan says about the PRD (mostly: it holds):

- **Keep as differentiators, not table stakes:** three-state honest availability, hard vetoes, pre-authored local content, async deadline + leaderboard fallback (every clone does unanimity-only, which fails exactly when groups are picky — our fallback is a genuine mechanical edge).
- **Accelerate:** venue verification (it is moat construction, not chores); the alpha trust bar (one stale-hours incident in alpha costs disproportionately).
- **Do not build:** review corpus, social feed, national coverage, AI chat interface. Competing where Beli/Yelp/ChatGPT are strong is the losing move; we integrate around them (deep links out, Places/Yelp data in).
- **Positioning language for any public surface:** decision, not discovery. "Settles it" beats "discover great restaurants" (which is claimed by everyone and means nothing now).
- **Watch list (check quarterly):** Beli group features, Cobble's AI Group Planner trajectory, Google Maps polling UI changes, Yelp AI chatbot scope, OpenTable/Amex partner-API terms (supplier power over our D4).

---

## Sources

- Direct competitors: [Sawa Eats](https://getsawa.app/), [Hangrily](https://hangrily.app/group-restaurant-decision), [Food Match](https://apps.apple.com/us/app/food-match-find-where-to-eat/id1473298709), [Food Swipe](https://apps.apple.com/us/app/food-swipe-find-restaurants/id6748421252), [Tine](https://apps.apple.com/us/app/tine-group-dining-planner/id1464574308), [SwipeEats](https://swipeeats.app/), [Food Flip](https://apps.apple.com/us/app/food-flip-decide-where-to-eat/id6740217074), [Food Flock](https://www.foodflockapp.com/), [Munch launch coverage (AlleyWatch, 2021)](https://www.alleywatch.com/2021/02/munch-group-dining-discovery-app-chris-desantis/), [swipe-dining trend piece (Resident, Jan 2026)](https://resident.com/resource-guide/2026/01/08/dinner-decisions-made-easy-swipe-your-way-to-agreement)
- Adjacent: [Beli (Wikipedia)](https://en.wikipedia.org/wiki/Beli_(app)), [Beli on Fast Company/Food Network coverage](https://www.foodnetwork.com/fn-dish/news/beli-app-trend), [Beli app](https://beliapp.com/), [Corner funding coverage](https://benzatine.com/news-room/corner-the-gen-z-map-app-revolutionizing-local-discoveries-with-38-million-in-funding), [Corner (App Store)](https://apps.apple.com/us/app/corner-curate-share-places/id1668282277), [Cobble](https://www.trycobble.com/), [Cobble $3M raise (GDI)](https://www.globaldatinginsights.com/news/cobble-the-app-for-deciding-date-activities-raises-3-million/), [Cobble: AI Group Planner (App Store)](https://apps.apple.com/us/app/cobble-ai-group-planner/id1472811526)
- Incumbents & substitutes: [Google Maps group polling (Digital Trends)](https://www.digitaltrends.com/phones/google-maps-group-polling/), [reservation platforms compared 2026 (Unstar)](https://unstar.app/blog/opentable-resy-yelp-tock-restaurant-reservation-apps-ranked-2026), [Bloomberg on OpenTable vs Resy/Yelp/SevenRooms (Apr 2026)](https://www.bloomberg.com/news/features/2026-04-17/opentable-vs-resy-yelp-sevenrooms-why-some-top-restaurants-are-switching), [Yelp AI chatbot (Washington Post, Apr 2026)](https://www.washingtonpost.com/business/2026/04/21/yelp-artificial-intelligence-chatbots-google-search/ea70f81e-3d71-11f1-bb46-ed564688d953_story.html), [ChatGPT and restaurants (Restaurant Business)](https://www.restaurantbusinessonline.com/technology/chatgpt-will-almost-feed-you-now), [how guests discover restaurants 2026 (Toast)](https://pos.toasttab.com/blog/data/how-guests-discover-new-restaurants), [Gen Z TikTok vs Google data (ALM)](https://almcorp.com/blog/gen-z-tiktok-google-preference-drop-2026-data/), [restaurant discovery stack (FSR)](https://www.fsrmagazine.com/feature/a-new-discovery-stack-emerges-for-restaurants/), [OpenTable Hartford listings](https://www.opentable.com/neighborhood/us/connecticut/hartford-restaurants), [CTbites](https://www.ctbites.com/)
