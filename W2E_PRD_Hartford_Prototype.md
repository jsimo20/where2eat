# Where2Eat Product Requirements Document

**Version 2.3 - Hartford Prototype**
**Date: July 4, 2026** (v2.2: July 4, 2026; v2.1: May 11, 2026)

> **v2.2 revision.** Three decisions supersede parts of v2.1 for the prototype:
>
> 1. **Platform:** Android app (Expo / React Native), distributed as a sideloaded APK for alpha testing. Launch and deployment strategy will be revisited once the alpha experience is satisfactory.
> 2. **Scope:** restaurants and bars only. Multi-act narrative itineraries and full-evening orchestration are deferred, not abandoned (see Section 3.5 and Phase 3).
> 3. **Core UX:** a swipe deck of photo-forward venue cards (Tinder/Hinge-style) with fit scores, one-tap menus, cuisine/drinks filters, and a minimalist low-cognitive-load design standard.
>
> Amended sections: 1.1 (quiz Q5), 3 (core experience), 4 (coverage, menus), 5 (mode naming), Technical Requirements (platform, performance), Launch Strategy, Success Metrics. Companion design docs: `docs/customer-journeys.md`, `docs/technical-design.md`.

> **v2.3 revision.** Group swipe matching added to MVP: account holders can run a match session with their partner or friends (rosters of 2 to 10) where everyone swipes the same deck; unanimous right-swipes surface as a match, a ranked-overlap leaderboard is the fallback, and the host locks the pick into the plan. Friends are added by invite link only. Accounts remain optional for the core solo/couple experience and are required for matching. New Section 3.6; amended Sections 1.3, User Account System, Data Governance, Success Metrics.

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
- Availability badge only when it earns attention ("Available now", "Unlikely tonight").

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

#### 3.6 Group Match (added v2.3)

Account holders can turn the deck into a group decision:

- **Roster:** the host starts a match session with their partner or friends: 2 to 10 people, picked from a friends list or invited by share link. Friends are added by invite link only (no search, no contact upload).
- **Lobby:** invitees land in a lobby, completing the quiz first if they haven't (their vetoes are needed). The host starts the session, which locks the roster.
- **One deck, everyone swipes:** all participants swipe the same deck in the same order. Hard vetoes are unioned across the whole roster, the budget ceiling is the lowest anyone set, and the host frames the area and tonight's story.
- **Match:** when every participant swipes right on the same venue, it surfaces as a match ("It's a dinner"). Multiple matches can accumulate; swiping can continue.
- **Leaderboard fallback:** if nothing goes unanimous (likely in bigger groups), the host ends the session and gets a ranked overlap view: venues ordered by right-swipe count.
- **Lock it in:** the host converts a match or leaderboard pick into the plan for the whole roster, party size prefilled. Large parties default to the call action, since reservation platforms rarely take 8-tops.

There is no group chat in the prototype and none planned: the session link rides the group's existing text thread. The app's job is the decision, not the conversation.

---

### 4. Restaurant & Bar Discovery and Booking (amended v2.2)

- **Coverage:** curated restaurants and bars (25 alpha, 100 Phase 1), regardless of reservation platform. Full-metro automated coverage remains Phase 2.
- **Booking aggregation:** Links to OpenTable, Resy, Yelp, or phone number.
- **Menus:** every curated venue carries a working menu link, one tap from the card. Menu coverage is a curation requirement.
- **Cultural context:** Authenticity indicators, signature dishes, cuisine background.

#### 4.1 Real-Time Availability Fallback UX

Availability is tiered:

- **Tier A — Live:** Confirmed via partner API in the last 5 minutes. Show "Available now" + one-tap booking.
- **Tier B — Stale cache:** Last confirmed 5–60 minutes ago. Show "Last confirmed available 23 min ago" + booking link.
- **Tier C — Pattern-inferred:** No live signal. Show historical pattern ("Usually has tables Tue/Wed at 7 PM") + one-tap phone call.
- **Tier D — Unavailable:** In the deck, the card is badged "Unlikely tonight" and down-ranked, with the best same-category alternative ranked above it. If the user shortlists it anyway, a transparent swap suggestion appears ("We swapped Bears Smokehouse for Black-Eyed Sally's — same lively BBQ vibe, available tonight").

---

### 5. Map and Explore UX (Secondary View)

The **primary UX is the swipe deck** (Section 3, amended v2.2). The map view is a secondary, opt-in "Explore" mode for users who want to browse spatially rather than swipe curated cards. Users toggle between Deck Mode (default) and Explore Mode. Explore Mode remains deferred to Phase 2.

In Explore Mode:

- **Split-screen design:** Map + list synchronized.
- **Route optimization:** Drive times, parking, walkability.
- **Contextual layers:** Weather, neighborhood characteristics, event-density heatmap.

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
- OpenTable, Resy, Yelp APIs for reservation availability
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
- Real-time availability updates within tier definitions in Section 4.1.
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
- **Alternative suggestions:** When a primary choice is unavailable, instant alternatives from the same narrative category (per Section 4.1).
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
- Enhanced map/Explore Mode.

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
- **Match adoption (v2.3, proposed targets):** 30% of Phase 1 active users run or join at least one match session; 60% of started sessions end in a match or a locked leaderboard pick.

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

*Where2Eat v2.3 — Hartford Prototype. Built to solve date night decision fatigue through a curated, swipeable deck of Hartford restaurants and bars, alone, as a couple, or as a group. Monetization deferred until after prototype validation.*

---

## Appendix: Sources for Hartford-Specific References

- Hartford neighborhoods: *Neighborhoods of Hartford, Connecticut*, Wikipedia — https://en.wikipedia.org/wiki/Neighborhoods_of_Hartford,_Connecticut
- PeoplesBank Arena (renamed from XL Center, June 2025), home of Hartford Wolf Pack and UConn basketball — https://www.peoplesbankarena.com/news/detail/oak-view-group-secures-new-naming-rights-partnership-with-peoplesbank-to-rename-xl-center
- Hartford Athletic (USL Championship) and Trinity Health Stadium — https://www.hartfordathletic.com/
- Wadsworth Atheneum Museum of Art, Connecticut Office of Tourism — https://ctvisit.com/
