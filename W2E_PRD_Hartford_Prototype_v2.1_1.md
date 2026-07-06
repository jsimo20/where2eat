# Where2Eat Product Requirements Document

**Version 2.1 — Hartford Prototype**
**Date: May 11, 2026**

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
- **Preferred evening structure** — dinner only, dinner + drinks, dinner + live music, dinner + show, dinner + late-night spot.

#### 1.2 Onboarding Modes

The system supports three onboarding paths so couples can pair however suits them:

- **Pass-the-phone:** Both partners answer back-to-back on one device. Blend is computed immediately.
- **Async invite:** Partner A completes the quiz, then shares an invite link via SMS or email. Partner B takes the quiz independently on their own device. Blend is computed when both quizzes are submitted.
- **Solo mode:** A single user completes the quiz and plans for themselves and their partner. The system flags the itinerary as "solo-planned" and uses Partner A's quiz as the sole input.

If the async invite is unanswered after 48 hours, the system prompts Partner A to either proceed in solo mode or resend the invite.

#### 1.3 Account Stance

No account is required for the core experience. Preferences and history are stored locally on-device for anonymous users. After three completed itineraries, the system offers optional account creation to enable multi-device sync. This is additive, never a paywall.

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

### 3. Core Experience Engine

#### 3.1 Input

The user selects tonight's "story" (energy state, not cuisine) and optional parameters (time of arrival, party size if not just the two of them).

#### 3.2 Output

3–5 narrative itinerary options.

#### 3.3 Narrative Itinerary Format

Each itinerary is a structured short story with the following components:

- **Title** — 4–8 evocative words (e.g., "A Slow West End Sunday," "Pre-Wolf Pack Pasta and Pints").
- **Opening line** — one sentence setting the mood and rationale ("For a night that starts quiet and ends laughing").
- **Three acts**, each with:
  - Time block (e.g., 6:45–8:15 PM)
  - Venue name, neighborhood, distance from prior stop
  - One-line "why this place tonight" rationale
  - Primary action button (book, call, directions)
- **Logistics footer:**
  - Total driving/walking time
  - Parking notes (lots, meters, validation)
  - Dress code if relevant
  - Weather note if outdoor seating is involved
  - Reservation status across the whole arc ("2 of 3 stops confirmed available")

Target length: ~150 words per itinerary. Tone calibrated to the energy archetype (cozy = warm and intimate prose; lively = punchy and energetic).

---

### 4. Restaurant Discovery & Booking

- **Comprehensive coverage:** All restaurants regardless of reservation platform.
- **Booking aggregation:** Links to OpenTable, Resy, Yelp, or phone number.
- **Cultural context:** Authenticity indicators, signature dishes, cuisine background.

#### 4.1 Real-Time Availability Fallback UX

Availability is tiered:

- **Tier A — Live:** Confirmed via partner API in the last 5 minutes. Show "Available now" + one-tap booking.
- **Tier B — Stale cache:** Last confirmed 5–60 minutes ago. Show "Last confirmed available 23 min ago" + booking link.
- **Tier C — Pattern-inferred:** No live signal. Show historical pattern ("Usually has tables Tue/Wed at 7 PM") + one-tap phone call.
- **Tier D — Unavailable:** Surface a same-category alternative immediately, with a transparent swap note ("We swapped Bears Smokehouse for Black-Eyed Sally's — same lively BBQ vibe, available tonight").

---

### 5. Map and Explore UX (Secondary View)

The **primary UX is narrative-first** (Section 3). The map view is a secondary, opt-in "Explore" mode for users who want to browse spatially rather than receive curated options. Users toggle between Story Mode (default) and Explore Mode.

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
- Event data for concerts, sports, shows (Ticketmaster, AXS, venue APIs)
- Real-time traffic/parking data
- Weather API for Hartford-specific conditions (New England seasonal swings, nor'easters)

### Platform

- Web application (mobile-responsive) for prototype.
- Native iOS and Android apps deferred to post-prototype.

### Performance

- <2 second load time for itinerary generation (measured at p95).
- Real-time availability updates within tier definitions in Section 4.1.
- Offline capability for saved itineraries.

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

- **Progressive registration:** Anonymous by default; account creation offered after three completed experiences.
- **Session-based learning:** Preferences stored on-device for anonymous users.
- **Data portability:** Export preference data and history.
- **Privacy controls:** Granular settings for data usage and location sharing.
- **Guest mode:** Full functionality without account creation.

---

## Error Handling & Reliability

- **Booking fallbacks:** Multiple reservation options per venue, graceful degradation to phone numbers.
- **Cache strategy:** Pre-load popular venues/availability to handle API outages.
- **Alternative suggestions:** When a primary choice is unavailable, instant alternatives from the same narrative category (per Section 4.1).
- **Offline mode:** Previously saved itineraries accessible without internet.

---

## Data Governance

- **Storage minimization:** Only essential preference data is stored; location data deleted after session.
- **GDPR-style data handling** even for US users.
- **Analytics privacy:** Aggregate behavioral data only, no individual tracking.
- **Anonymous analytics:** Device fingerprinting for basic preference learning without accounts.
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
- Open for dinner service at least 5 nights/week.
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

- James and spouse only.
- 25 manually curated venues across Downtown, West End, and West Hartford.
- Goal: validate quiz, blend logic, narrative output format, and core engine.

### Phase 1 — Hartford Restaurant Core (Months 3–6)

- 5–10 beta couples recruited from James's network.
- 100 manually curated venues (per criteria above).
- Basic personality quiz and matching.
- Single-activity itineraries (dinner + one optional add-on).

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

- **Primary engagement:** 40% of first-time users either complete a booking *or save an itinerary* within 7 days.
- **Activation:** Average session converts to action (booking or saved itinerary) in <5 minutes.
- **Quality:** User completes a suggested itinerary and rates it "would repeat" at least 60% of the time.

### Retention & Growth

- **Short-term:** 60% monthly active retention after 3 months.
- **Long-term:** Weekly active usage by core couples.
- **Expansion:** 25% of active users engage with a non-restaurant pillar in Phase 3+.

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

*Where2Eat v2.1 — Hartford Prototype. Built to solve date night decision fatigue through curated, narrative-driven recommendations in the Hartford, Connecticut metro area. Monetization deferred until after prototype validation.*

---

## Appendix: Sources for Hartford-Specific References

- Hartford neighborhoods: *Neighborhoods of Hartford, Connecticut*, Wikipedia — https://en.wikipedia.org/wiki/Neighborhoods_of_Hartford,_Connecticut
- PeoplesBank Arena (renamed from XL Center, June 2025), home of Hartford Wolf Pack and UConn basketball — https://www.peoplesbankarena.com/news/detail/oak-view-group-secures-new-naming-rights-partnership-with-peoplesbank-to-rename-xl-center
- Hartford Athletic (USL Championship) and Trinity Health Stadium — https://www.hartfordathletic.com/
- Wadsworth Atheneum Museum of Art, Connecticut Office of Tourism — https://ctvisit.com/
