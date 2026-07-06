# Where2Eat Product Requirements Document

**Version 2.6 - Hartford Prototype**
**Date: July 4, 2026** (v2.5 through v2.2: July 4, 2026; v2.1: May 11, 2026)

> **v2.2 revision.** Three decisions supersede parts of v2.1 for the prototype:
>
> 1. **Platform:** Android app (Expo / React Native), distributed as a sideloaded APK for alpha testing. Launch and deployment strategy will be revisited once the alpha experience is satisfactory.
> 2. **Scope:** restaurants and bars only. Multi-act narrative itineraries and full-evening orchestration are deferred, not abandoned (see Section 3.5 and Phase 3).
> 3. **Core UX:** a swipe deck of photo-forward venue cards (Tinder/Hinge-style) with fit scores, one-tap menus, cuisine/drinks filters, and a minimalist low-cognitive-load design standard.
>
> Amended sections: 1.1 (quiz Q5), 3 (core experience), 4 (coverage, menus), 5 (mode naming), Technical Requirements (platform, performance), Launch Strategy, Success Metrics. Companion design docs: `docs/customer-journeys.md`, `docs/technical-design.md`.

> **v2.3 revision.** Group swipe matching added to MVP: account holders can run a match session with their partner or friends (rosters of 2 to 10) where everyone swipes the same deck; unanimous right-swipes surface as a match, a ranked-overlap leaderboard is the fallback, and the host locks the pick into the plan. Friends are added by invite link only. Accounts remain optional for the core solo/couple experience and are required for matching. New Section 3.6; amended Sections 1.3, User Account System, Data Governance, Success Metrics.

> **v2.4 revision.** Map/Explore mode and live reservation availability move into MVP. The map (Section 5) is a full-screen toggle showing exactly the deck's candidate pool as pins with a card carousel; ranking layers (routes, heatmaps) stay Phase 2. Live availability ships with demand-driven partner polling (Section 4.2). Honest dependency, stated once: partner API access (OpenTable, Resy, Yelp Guest Manager, SevenRooms, Tock) is granted by those platforms, not built by us. Applications go out at build milestone 1 and the software is partner-ready from day one.

> **v2.5 revision.** Group matching is now fully asynchronous: the lobby/roster-lock model is replaced by a host-set close time. Participants join and swipe on their own schedule before the deadline; results are computed at close as a ranked-overlap leaderboard with unanimous picks highlighted; a late joiner's vetoes remove venues from contention without reshuffling anyone's deck. Live mid-session match popups are dropped. Amended Sections 3.6 and Success Metrics.

> **v2.6 revision.** Availability simplified from four tiers to three verified states: **Closed** (posted schedule, daily refresh), **Open, no reservations** (guidance: try walking in or call), and **Open, reservations available** (OpenTable and other integrated tools check seats on the user's behalf). Removed on purpose: pattern-inferred claims ("usually has tables"), stale-cache claims ("confirmed 23 min ago"), manual availability entry, feedback-driven availability, availability-based ranking, and automated swaps. Sections 4.1 and 4.2 rewritten.

---

## Product Vision

A Hartford, Connecticut–focused date night planning platform that eliminates decision fatigue by providing curated, narrative-driven evening itineraries. Restaurant discovery is the anchor for the prototype, with the broader vision extending to full evening orchestration (sports, events, music, art) in later phases.

## Core User Problem

Couples experience decision paralysis when planning date nights, particularly around the question "where do you want to go for dinner?" Current solutions require juggling multiple apps, endless scrolling, and result in suboptimal experiences.

## Target Users

**Primary:** Couples in the Hartford metro area, ages 25–45, who dine out 2+ times per month and value curated experiences over exhaustive search.

**Secondary:** Solo planners organizing a night for themselves and a partner who hasn't onboarded.

**Beta Users:** James and spouse as primary test case and product validation.

## Prototype Scope Note

This document defines the **prototype phase**. Monetization (ads, premium tiers, affiliate revenue) is intentionally **out of scope**. The prototype focuses purely on validating: can we generate narrative itineraries that couples in Hartford actually want to use? Revenue models are deferred until after product–market fit is demonstrated.

---

## Core Features

### 1. Onboarding & Personalization

#### 1.1 The Quiz

A 5-question personality quiz:

- **Neighborhood preference** — Downtown, West End, Asylum Hill, Frog Hollow, Parkville, plus West Hartford's Blue Back Square / Park Road corridor.
- **Energy archetype** — cozy & romantic, lively & social, cultural & curious, low-key & familiar, adventurous & new.
- **Travel radius tolerance** — 5–10 min, 15–20 min, 30+ min, anywhere for the right experience.
- **Budget comfort zone** — $, $$, $$$, $$$$, depends on occasion.
- **Preferred evening structure** (narrowed in v2.2 to restaurant + bar scope): dinner only, dinner then drinks, drinks then dinner, just drinks. Live music, show, and late-night structures return with Phase 3 orchestration.

#### 1.2 Onboarding Modes

The system supports three onboarding paths so couples can pair however suits them:

- **Pass-the-phone:** Both partners answer back-to-back on one device. Blend is computed immediately.
- **Async invite:** Partner A completes the quiz, then shares an invite link via SMS or email. Partner B takes the quiz independently on their own device. Blend is computed when both quizzes are submitted.
- **Solo mode:** A single user completes the quiz and plans for themselves and their partner. The system flags the itinerary as "solo-planned" and uses Partner A's quiz as the sole input.

If the async invite is unanswered after 48 hours, the system prompts Partner A to either proceed in solo mode or resend the invite.

#### 1.3 Account Stance

No account is required for the core solo/couple experience. Preferences and history are stored locally on-device for anonymous users. After three completed plans, the system offers optional account creation to enable multi-device sync. This is additive, never a paywall.

Group matching (Section 3.6, v2.3) is the deliberate exception: it requires an account, because friends lists, display names, and cross-device sessions need durable identity. Tapping into matching from an anonymous state routes through account creation, migrating the local history along the way.

---

### 2. The Date Night Blend (Compatibility Methodology)

When both partners have completed the quiz, the system computes a **Date Night Blend** — the joint preference profile that drives recommendations.

#### 2.1 Resolution Hierarchy

The blend resolves preferences in this strict order:

1. **Vetoes (hard constraints, no override):**
   - Dietary restrictions (allergies, religious, ethical) — the union of both partners' restrictions applies.
   - Accessibility needs — the union of both partners' needs applies.
   - Absolute budget ceiling — the lower of the two partners' caps.

2. **Blended (soft preferences, averaged or harmonized):**
   - Energy archetype — if both partners agree, use it; if they diverge, identify a "third way" archetype (e.g., one wants romantic, one wants social → "intimate booth in a lively room") and surface 1–2 of these blend zones.
   - Neighborhood preference — prefer overlap; if disjoint, alternate primacy across sessions.
   - Travel radius — use the shorter radius by default, with an "expand if needed" toggle.
   - Evening structure — prefer overlap; if disjoint, present one option for each partner's preference.

3. **Rotation (for unresolved aesthetic differences):**
   - If preferences can't be harmonized into a single blend, the system rotates primacy across sessions: tonight honors Partner A's pull, next session honors Partner B's. The UI shows whose night it leans toward, openly.

#### 2.2 Blend Output

The user sees:

- A short blend summary ("You and Sam agree on cozy and West End; you stretch on budget and timing").
- A compatibility score (0–100) — visible but not gamified; this is informational, not a ranking.
- A list of resolved vetoes (e.g., "Gluten-free + wheelchair-accessible required").

#### 2.3 Solo Blend

In solo mode, the Blend is simply Partner A's profile with a "solo" flag. If the absent partner later completes the quiz, the Blend recomputes and the user is notified.

---

### 3. Core Experience Engine (amended v2.2: the Swipe Deck)

#### 3.1 Input

The user selects tonight's "story" (energy state, not cuisine) and optional parameters (time of arrival, party size if not just the two of them). Session filters can narrow the deck: cuisine categories and a "drinks" filter (bars and cocktail spots). Filters narrow the deck; they never override vetoes (Section 2.1).

#### 3.2 Output

A ranked deck of up to ~30 venue cards (restaurants and bars), ordered by blend fit. Deck order is deterministic per couple per night: both partners browsing the same evening see the same cards in the same order.

#### 3.3 Card Format

The deck is browsed one full-screen card at a time: swipe left = pass, swipe right = shortlist, tap = detail, undo reverses the last swipe. Every gesture has a visible button equivalent (Section 9 accessibility).

Card front, maximum five elements:

- Full-bleed venue photo (3 to 5 curated photos per venue).
- Venue name + one metadata line: cuisine or drink tag, neighborhood, price tier, distance.
- Two numbers only: aggregate rating (Google + Yelp, review-count weighted) and blend-fit percentage.
- One-line "why this place tonight" blurb, pre-authored per energy archetype with weather/season variants. Tone calibrated to the archetype (cozy = warm and intimate; lively = punchy).
- Reservations badge only when a live check verified seats ("Reservations available"); open venues without verified seats carry the walk-in-or-call guidance in the detail view.

Card detail (tap): photo gallery, full blurb, tonight's hours, availability per Section 4.1, one-tap menu (in-app browser), book / call / directions, dietary and accessibility icons, content flag.

Shortlisted cards collect into a compare view; picking a winner (or a dinner + drinks pair) creates the **plan**: the bookable, offline-cached artifact for the evening.

#### 3.4 Design Principles

Minimalist, glossy, low cognitive load. Testable rules, not taste:

- One decision per screen; the deck asks exactly one question at a time.
- Five elements max on a card front; two numbers max; everything else is progressive disclosure behind the tap.
- Photography is the interface: full-bleed images, text on a bottom scrim, no chrome while swiping.
- Dark-first palette (the app is used on evenings out), one accent color, two type scales per screen.
- Motion communicates state: spring physics, haptic tick on decisions, no decorative animation.
- Honest empty and thin states ("7 places open tonight that fit"), never fake-infinite feeds.

A screen that violates a rule needs a written reason.

#### 3.5 Deferred: Narrative Itineraries

The v2.1 core experience (3 to 5 narrative multi-act itineraries with title, opening line, three acts, and a logistics footer) is deferred, not deleted. The blend, scoring, and pre-authored blurbs that power the deck are the same components multi-act assembly composes on top of; the design essence is preserved in `docs/technical-design.md` Appendix A and returns alongside Phase 3 orchestration.

#### 3.6 Group Match (added v2.3, fully async as of v2.5)

Account holders can turn the deck into a group decision. The whole flow is asynchronous: nobody needs to be on the app at the same time.

- **Roster:** the host creates a match session with their partner or friends: 2 to 10 people, picked from a friends list or invited by share link. Friends are added by invite link only (no search, no contact upload).
- **Time box:** the host sets a close time (presets: 1 hour, 3 hours, this evening, 24 hours). Everyone swipes whenever they can before the deadline; a progress view shows participation ("4 of 6 have swiped"). The host can close early once enough votes are in.
- **One deck, everyone swipes:** the deck is generated at creation from the host's area + story and the invited participants' vetoes, with the lowest budget ceiling applied. Someone joining by link completes the quiz first if needed, and their vetoes remove venues from contention: the deck only ever shrinks, never reorders, and a venue that violates any participant's vetoes cannot win even if it collected right-swipes before they joined.
- **Results at close:** at the deadline (or early close) the session locks and results appear for everyone: venues ranked by right-swipe count with blend fit as the tie-break, unanimous picks highlighted at the top ("everyone said yes"), participation shown honestly. Non-swipers never count against a venue; their vetoes still protect them.
- **Lock it in:** the host converts a result into the plan for the whole roster, party size prefilled. Large parties default to the call action, since reservation platforms rarely take 8-tops.

There is no group chat in the prototype and none planned: the session link rides the group's existing text thread. The app's job is the decision, not the conversation.

---

### 4. Restaurant & Bar Discovery and Booking (amended v2.2)

- **Coverage:** curated restaurants and bars (25 alpha, 100 Phase 1), regardless of reservation platform. Full-metro automated coverage remains Phase 2.
- **Booking aggregation:** Links to OpenTable, Resy, Yelp, or phone number.
- **Menus:** every curated venue carries a working menu link, one tap from the card. Menu coverage is a curation requirement.
- **Cultural context:** Authenticity indicators, signature dishes, cuisine background.

#### 4.1 Availability States (simplified in v2.6)

Availability is exactly three user-facing states, each backed by something we verified. We are deliberately careful about what we tell the user: no guesses, no stale claims.

- **Closed:** the venue's posted schedule (from our daily refresh) says it isn't open tonight. Closed venues never appear in a tonight deck; anywhere else they surface (a saved plan, a future-dated shortlist), they're labeled plainly.
- **Open (no reservations):** open per posted schedule, but no verified seats: the venue doesn't take reservations, a live check found none left, or we couldn't check. The user is told they can try walking in or call the restaurant. If the venue has a reservation platform, its link remains available in the detail view, without an availability claim.
- **Open (reservations):** OpenTable or another integrated reservation tool checked for available seats on the user's behalf, within the last 30 minutes, at the user's party size, and found them. "Reservations available" + one-tap booking with date, time, and party size prefilled.

Availability never changes a venue's ranking; it informs and routes the action (book vs walk-in/call).

#### 4.2 State Determination Process (amended v2.6)

Two data sources, nothing else:

1. **Daily schedule refresh:** the nightly job pulls each venue's posted hours and closed days. This alone decides Closed vs Open.
2. **Reservation checks (when partner access exists):** demand-driven. Generating a deck or opening a venue's detail view queues a background seat check for surfaced venues whose latest check is older than 5 minutes. Screens always render instantly from cache and update in place; no screen ever waits on a partner API. "Reservations available" is only ever asserted from a check fresh within 30 minutes.

There is no manual availability entry and no user-feedback availability signal (deliberately, to keep the system simple and its claims verifiable). If a reservation check fails or partner access doesn't exist yet, open venues simply show the walk-in/call guidance. The outbound booking tap is the tracked activation event; booking completion is not verifiable in the prototype.

---

### 5. Map and Explore UX (Secondary View)

The **primary UX is the swipe deck** (Section 3). The map is a secondary, opt-in "Explore" mode for users who want to browse spatially rather than swipe. **In MVP as of v2.4:**

- One tap toggles the deck into a full-screen map.
- Pins show exactly the current deck's candidate pool: same vetoes, same filters, same story. The map never surfaces a venue the deck wouldn't.
- Tapping a pin opens that venue's card in a bottom carousel; swiping the carousel pans the map. Pass and shortlist work from the map card and count as normal swipes; tapping through opens the same detail sheet (menu, photos, availability, booking).
- Location permission is optional: without it the map centers on Hartford; with it, a position dot appears (position is used on-device only).

Deferred to Phase 2: route optimization (drive times, parking, walkability), contextual layers (weather, neighborhood character, event-density heatmap), and the split-screen web layout.

This resolves the prior tension between "narrative, not lists" and "split-screen map + list" — they are two distinct modes, not one conflicted screen.

---

### 6. Post-Experience Learning

- **Simple feedback:** "Would repeat?" + mood descriptors (not star ratings).
- **Pattern recognition:** System learns couple preferences over time.
- **Relationship signals:** Captures what works for specific dynamics/occasions.

---

### 7. Hartford-Specific Intelligence

- **Seasonal optimization:** Patio season flags (~May–September), snow-friendly options for New England winters, fall-foliage timing for scenic drives through the Farmington Valley and along the Connecticut River.
- **Local context:** Pre-show dining around The Bushnell; pre-game spots near PeoplesBank Arena (formerly the XL Center) for Hartford Wolf Pack and UConn basketball nights; Trinity Health Stadium proximity for Hartford Athletic match days; the West Hartford / Blue Back Square dining corridor as an alternative to downtown.
- **Weather adaptation:** Real-time weather integration affecting outdoor seating, snow-day routing, and drive-radius tolerance.

---

### 8. Dietary & Accessibility Handling (Venue-Level)

- **Dietary filters:** Vegan, vegetarian, gluten-free, halal, kosher options within cuisine searches.
- **Venue accessibility:** Wheelchair access, noise levels, lighting considerations, restroom accessibility.
- **Matching integration:** Preference learning includes dietary patterns.

---

### 9. App-Level Accessibility Standards

The application itself (separate from venue accessibility) targets WCAG 2.1 AA compliance:

- Screen reader support (VoiceOver, TalkBack) for full itinerary navigation.
- Keyboard navigation for web.
- Text resize up to 200% without loss of functionality.
- Color contrast ratio 4.5:1 minimum for body text, 3:1 for large text.
- Captions for any video content.
- Alt text on all venue imagery.
- No critical information conveyed by color alone.

Accessibility testing is a gating criterion before each phase release.

---

## Technical Requirements

### Data Sources

- Google Places API for restaurant listings
- OpenTable, Resy, Yelp APIs for reservation availability (v2.4: partner-access applications at build milestone 1; demand-driven polling once granted; manual/pattern tiers and prefilled booking deep links regardless)
- Event data for concerts, sports, shows (deferred: manual event flags in the prototype; Ticketmaster/AXS/venue APIs arrive with Phase 3)
- Real-time traffic/parking data (deferred: curated parking notes in the prototype)
- Weather API for Hartford-specific conditions (New England seasonal swings, nor'easters)

### Platform (amended v2.2)

- Android app (Expo / React Native) for the prototype, distributed as a sideloaded APK to alpha testers. JS-level changes ship over the air between installs; no store listing during alpha.
- Web surfaces limited to the curation admin and the invite-landing quiz page (a partner without the app must be able to answer an invite in the browser).
- iOS, public web client, and Play Store distribution deferred to the post-alpha launch-strategy conversation.

### Performance

- <2 second deck generation (measured at p95).
- Swiping holds 60fps with no visible image loading in normal use (upcoming cards prefetched).
- Explore map pin set updates within 500ms of a filter change (client-side data, no new fetch).
- Availability states per Section 4.2: schedule from the daily refresh; seat claims only from reservation checks fresh within 30 minutes.
- Offline capability for saved plans.

### Content Quality

- **Review aggregation:** Multiple sources (Google, Yelp, TripAdvisor) with authenticity weighting.
- **Editorial curation:** Partner with local Hartford writers — CT Bites, Hartford Courant food coverage, Hartford Magazine.
- **Community moderation:** Flag system for inappropriate content.

### Scalability Considerations

- Database architecture supports multiple cities (future expansion).
- Content management supports easy addition of venues, events, and cultural context.
- API rate limiting in place from day one.

---

## User Account System

- **Progressive registration:** Anonymous by default; account creation offered after three completed plans, and required only for group matching (Section 3.6).
- **Friends (v2.3):** added by invite link only; list, remove, and block. No contact upload, no username search, no discovery.
- **Session-based learning:** Preferences stored on-device for anonymous users.
- **Data portability:** Export preference data and history.
- **Privacy controls:** Granular settings for data usage and location sharing.
- **Guest mode:** Full functionality without account creation.

---

## Error Handling & Reliability

- **Booking fallbacks:** Multiple reservation options per venue, graceful degradation to phone numbers.
- **Cache strategy:** Pre-load popular venues/availability to handle API outages.
- **Alternative suggestions:** the shortlist and the deck are the user's alternatives when a choice doesn't work out; the system makes no speculative availability claims or automated swaps (Section 4.2).
- **Offline mode:** Previously saved plans accessible without internet.

---

## Data Governance

- **Storage minimization:** Only essential preference data is stored; location data deleted after session.
- **GDPR-style data handling** even for US users.
- **Analytics privacy:** Aggregate behavioral data only, no individual tracking.
- **Anonymous analytics (amended v2.3):** aggregate behavioral counters only; no device fingerprinting (an opaque profile UUID provides continuity). Account holders' PII is limited to auth email and display name; friends lists and session history are included in export and deletion.
- **Clear value exchange:** Account creation unlocks multi-device sync, deeper personalization, saved favorites — never required for core functionality.

---

## Content Strategy

### Tiered Recommendation Engine

- **Tier 1 — Comprehensive baseline:** All restaurants meeting basic operating criteria (open, reachable, has listing).
- **Tier 2 — Quality filter:** Rating thresholds and review sentiment analysis applied to Tier 1.
- **Tier 3 — Cultural authenticity layer:** Editorial input and community validation. **Phase 1's manually curated 100 venues live here.**
- **Tier 4 — Personal preference matching:** Learned patterns applied per couple.

### Phase 1 Curation Criteria (the "Top 100")

A venue qualifies for the Phase 1 manual list if it meets **all** of:

- 4.3+ aggregate rating with 200+ reviews across Google and Yelp.
- Active reservation system (OpenTable, Resy, or consistent phone reservations).
- Open for dinner service at least 5 nights/week (restaurants) or evening service at least 5 nights/week (bars).
- Editorial validation: appears in at least one of CT Bites, Hartford Courant food coverage, Hartford Magazine, or The Infatuation in the past 24 months.
- No more than one ownership/concept change in the past 18 months.

The set is balanced for:

- **Cuisine diversity:** at least 12 distinct cuisine categories represented.
- **Geographic diversity:** at least 8 neighborhoods/inner-ring towns represented.
- **Price range:** $ through $$$$ all represented, with $$ as the largest bucket.

National chain restaurants are excluded; local mini-chains (≤5 Connecticut locations) are eligible.

---

## Competitive Landscape

Where2Eat operates in a crowded but fragmented space. The wedge is the combination of narrative itinerary + couple compatibility + full-evening orchestration. No single competitor offers all three.

- **Reservation-first:** OpenTable, Resy — strong booking flows, weak discovery and zero couple logic.
- **Editorial discovery:** The Infatuation, Eater, Hartford Magazine, CT Bites — strong taste, weak booking and personalization.
- **General discovery:** Google Maps, TripAdvisor, Yelp — strong inventory, low signal-to-noise.
- **Tourism boards:** Visit Hartford, CT Office of Tourism — partner opportunities, not direct competitors.

Where2Eat does not try to out-inventory Google or out-curate Eater; it tries to be the only place a Hartford couple can answer "where are we going tonight?" in under five minutes with confidence.

---

## Launch Strategy

### Phase 0 — Closed Alpha (Months 1–2)

- James and spouse only, via sideloaded APK on both phones (over-the-air updates between installs).
- 25 manually curated venues (restaurants and bars) across Downtown, West End, and West Hartford.
- Goal: validate quiz, blend logic, the swipe-deck experience, and core engine.

### Phase 1 — Hartford Restaurant Core (Months 3–6)

- 5–10 beta couples recruited from James's network (APK distribution).
- 100 manually curated venues (per criteria above).
- Basic personality quiz and matching.
- Plans of a single venue or a dinner + drinks pair.

### Phase 2 — Full Hartford-Area Restaurant Coverage (Months 7–12)

- 50–100 active couples.
- Automated venue discovery and aggregation across full Hartford metro.
- Multi-platform booking integration.
- Explore Mode enhancements: route optimization, contextual layers (weather, event density).

### Phase 3 — Full Date Night Orchestration (Year 2)

- Sports events (Hartford Wolf Pack, Hartford Athletic, Hartford Yard Goats, UConn basketball/hockey at PeoplesBank Arena).
- Music and performing arts (The Bushnell, Infinity Hall, Webster Theater, Xfinity Theatre, PeoplesBank Arena concerts).
- Art and culture (Wadsworth Atheneum, Mark Twain House & Museum, Connecticut Science Center, Hartford Stage, TheaterWorks Hartford).

### Phase 4 — Second-City Expansion (Year 3+)

- Apply learnings to a second market.
- Refined recommendation engine.
- Scalable content operations.

### Team & Resources (Placeholder)

- 1 founder/PM (James)
- 1 full-stack developer (contracted through Phase 2)
- 1 designer (contracted through Phase 2)
- 1 local editorial contributor (part-time)

Budget and hiring plan are deferred until post-prototype.

---

## Success Metrics

### Product-Market Fit Indicators

- **Primary engagement:** 40% of first-time users either complete a booking *or save a plan* within 7 days.
- **Activation:** Average session converts to action (booking or saved plan) in <5 minutes.
- **Quality:** User completes a plan and rates it "would repeat" at least 60% of the time.

### Retention & Growth

- **Short-term:** 60% monthly active retention after 3 months.
- **Long-term:** Weekly active usage by core couples.
- **Expansion:** 25% of active users engage with a non-restaurant pillar in Phase 3+.
- **Match adoption (v2.3, proposed targets):** 30% of Phase 1 active users run or join at least one match session; 60% of created sessions end in a locked pick before expiry.

### Operational Metrics

- **Reliability:** 99% uptime for booking integrations.
- **Content Quality:** >80% of recommendations rated positively.

(Note: the <2 second performance target is defined in Technical Requirements and is not duplicated here.)

---

## Risk Mitigation

### Technical Risks

- **API dependencies:** Multiple fallback options per critical integration; pattern-inferred availability as last resort.
- **Data quality:** Manual curation plus automated quality checks.
- **Scaling:** Cloud-native architecture from day one.

### Product Risks

- **Competition:** Focus on the unique wedge (narrative + blend + orchestration).
- **User adoption:** Start with personal use case to validate core value.
- **Sustainability:** Prototype is unmonetized by design; sustainability planning deferred to post-validation.

### Market Risks

- **Seasonal variation:** Hartford-specific features handle New England weather impacts (winter storms, mud season, peak fall foliage, summer patio surge).
- **Local partnerships:** Partner with rather than compete against CT Bites, Hartford Courant, Visit Hartford.
- **Smaller market size:** Hartford metro is smaller than peer launch cities; success is measured in engagement depth, not raw user volume.

---

## Next Steps

1. Validate core assumptions with user interviews beyond the beta couple.
2. Technical architecture design for MVP.
3. Content partnerships with Hartford food writers and bloggers.
4. API partnership negotiations with OpenTable, Resy, Yelp.
5. MVP wireframes and design system.

---

*Where2Eat v2.6 — Hartford Prototype. Built to solve date night decision fatigue through a curated, swipeable deck of Hartford restaurants and bars, alone, as a couple, or as a group. Monetization deferred until after prototype validation.*

---

## Appendix: Sources for Hartford-Specific References

- Hartford neighborhoods: *Neighborhoods of Hartford, Connecticut*, Wikipedia — https://en.wikipedia.org/wiki/Neighborhoods_of_Hartford,_Connecticut
- PeoplesBank Arena (renamed from XL Center, June 2025), home of Hartford Wolf Pack and UConn basketball — https://www.peoplesbankarena.com/news/detail/oak-view-group-secures-new-naming-rights-partnership-with-peoplesbank-to-rename-xl-center
- Hartford Athletic (USL Championship) and Trinity Health Stadium — https://www.hartfordathletic.com/
- Wadsworth Atheneum Museum of Art, Connecticut Office of Tourism — https://ctvisit.com/
