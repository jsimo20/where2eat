# Where2Eat: Customer Journeys and MVP Scenario Catalog

Companion to [W2E_PRD_Hartford_Prototype_v2.2.md](../W2E_PRD_Hartford_Prototype_v2.2.md).
Defines the journey stages, the probable scenarios the MVP must handle, spec gaps found in the PRD (with proposed resolutions), and the scope lines.

**Revision note (v2):** this version reflects three PM decisions, since folded into PRD v2.2: (1) Android app distributed by sideloaded APK for alpha testing, web deferred; (2) scope limited to restaurants and bars, multi-act narrative itineraries deferred; (3) core UX is a swipe deck (Tinder/Hinge-style cards) with photo-forward venue cards, fit scores, menu access, and cuisine/drinks filters, under a minimalist low-cognitive-load design goal.

**MVP definition used throughout:** Phase 0 closed alpha through Phase 1. Android app (Expo/React Native), sideloaded APK, 25 then 100 curated venues (restaurants and bars), swipe-deck UX. Launch/deployment strategy deliberately deferred until the alpha experience is satisfactory.

---

## 1. Journey map

Seven stages. Every scenario below hangs off one of these.

| # | Stage | What happens |
|---|-------|--------------|
| 0 | First open | Anonymous by default. Local device profile created. No paywall, no signup wall. |
| 1 | Onboarding | 5-question quiz plus a hard-requirements step, via one of three pairing modes (pass-the-phone, async invite, solo). |
| 2 | Blend | Vetoes unioned, soft preferences blended or rotated, summary + compatibility score shown. |
| 3 | Browse the deck | User picks tonight's story (energy state) and optional filters (cuisine, drinks). A ranked deck of venue cards comes back in under 2s. Swipe left = pass, right = shortlist, tap = detail (photos, menu, hours, booking). |
| 4 | Shortlist and decide | Review shortlisted cards side by side, pick the winner (or dinner + drinks pair). Booking actions per availability tier. Plan saved for offline. |
| 5 | The date | Saved plan works offline. tel:, maps, and menu links function without our backend. |
| 6 | Afterward | Next-open feedback (would repeat + mood chips). Swipes and feedback feed learning. After 3 completed plans, optional account offer. |

```mermaid
flowchart TD
    A[First open: anonymous device profile] --> B{Onboarding mode?}
    B -->|Pass the phone| C[Both quizzes on one device]
    B -->|Async invite| D[Partner A quiz, share invite link]
    B -->|Solo| E[One quiz, solo flag]
    D --> F{Partner B completes within 48h?}
    F -->|Yes| G[Blend computed]
    F -->|No| H[Nudge A: go solo or resend]
    H --> E
    H --> D
    C --> G
    E --> G2[Solo blend]
    G --> I[Tonight's story + optional cuisine or drinks filters]
    G2 --> I
    I --> J[Ranked deck of venue cards, under 2s]
    J --> K{Swipe}
    K -->|Left: pass| J
    K -->|Right: shortlist| J
    K -->|Tap: detail sheet| L[Photos, menu, hours, availability, book or call]
    L --> J
    J -->|Deck exhausted| M[End card: review shortlist or widen filters]
    M --> N[Shortlist review: pick the winner]
    L -->|Book now| N
    N --> O["Plan created: book / call / directions, offline capable"]
    O --> P[Date happens]
    P --> Q[Next open: would repeat + mood chips]
    Q --> R[Learning updates couple profile]
    R --> S{3 completed plans?}
    S -->|Yes| T[Optional account offer: sync + portability]
    S -->|No| I
    T --> I
```

---

## 2. Scenario catalog

Numbered so we can reference them in the technical design and test plan. "Required MVP behavior" is the acceptance bar. IDs are stable across doc revisions: S13..S26 were adapted from the itinerary-era design; S24 is deferred (its ID is not reused); S35..S40 are new with the swipe UX.

### Stage 1: Onboarding and pairing

| ID | Scenario | Required MVP behavior |
|----|----------|----------------------|
| S1 | Pass-the-phone happy path | Both partners answer back-to-back on one device. Blend computed immediately, no network round trip needed beyond quiz submit. |
| S2 | Async invite happy path | A completes quiz, shares link (device share sheet). B opens on their own device, completes quiz. Blend computed on B's submit; B sees it immediately, A sees a "blend ready" banner on next open. B without the app installed hits a lightweight web quiz page (invite links must not require a sideloaded APK to answer). |
| S3 | Async invite stalls | 48h after send with no acceptance: invite marked expired, A prompted in-app on next open to go solo or resend (resend issues a new token, old one invalidated). |
| S4 | Invite token misuse | Token is single-use and expires at 48h. Second open of a used token shows a friendly "already paired" screen. Expired token offers to request a fresh invite. |
| S5 | Solo mode | One quiz, plans flagged "solo-planned". Full functionality otherwise. |
| S6 | Solo, partner joins later | Absent partner completes the quiz later (via invite from the solo user). Blend recomputes, solo flag drops, user notified on next open (PRD 2.3). |
| S7 | Returning anonymous user | Same device: profile and history load locally, no re-quiz. Straight to Stage 3. |
| S8 | Quiz retake / preference edit | Either partner can retake or edit answers at any time. Blend recomputes, rotation state preserved. |

### Stage 2: Blend computation

| ID | Scenario | Required MVP behavior |
|----|----------|----------------------|
| S9 | Aligned couple | Overlapping answers produce a clean blend, high compatibility score, short summary. |
| S10 | Divergent energy archetypes | Third-way lookup (5x5 archetype matrix) surfaces 1 or 2 blend zones, e.g. romantic + social = "intimate booth in a lively room". |
| S11 | Disjoint neighborhoods or structure | Rotation with visible primacy: "tonight leans toward Sam's pick" influences deck ranking, shown openly. |
| S12 | Heavy vetoes, thin candidate pool | Vetoes (dietary union, accessibility union, min budget cap) are hard filters and are NEVER relaxed, including by user-applied cuisine/drinks filters. If the deck falls under a handful of cards, offer soft relaxations (radius expand, widen neighborhoods) with honest messaging. If still empty, say so plainly and show the closest misses with what blocked them. |

### Stage 3: Browse the deck

| ID | Scenario | Required MVP behavior |
|----|----------|----------------------|
| S13 | Standard deck generation | Story + optional filters in, ranked deck (up to ~30 cards) out, p95 under 2s. Deterministic per couple per night: both partners see the same deck order on the same evening. |
| S14 | Thin inventory night | Sunday/Monday/holiday in Hartford: fewer venues open. Small deck shown with an honest count ("7 places open tonight that fit"), plus the widen prompt. Never pad with venues that violate the blend. |
| S15 | Bad weather | Rain or nor'easter: patio-dependent venues down-ranked or badged, weather-variant card blurbs used, snow-day bias toward shorter drives. |
| S16 | Event night | Bushnell show / Wolf Pack / UConn game night (manual event flags in MVP): downtown cards carry a parking-pressure warning and pre-show timing note. |
| S17 | Repeat user novelty | Venues passed recently are down-ranked for a while; the venue from the last completed plan is excluded unless the archetype is "low-key & familiar" or feedback said "would repeat". Familiarity seekers get deliberate resurfacing of loved spots. |
| S18 | Party size above 2 | Passed through to the plan and booking action. No group-blend logic in MVP (blend stays the couple's). |
| S35 | Swipe mechanics | Left = pass, right = shortlist, tap = detail sheet. Undo button reverses the last swipe (fat thumbs are real). On-screen pass/save buttons mirror the gestures at all times: gesture-only UI fails screen-reader users and WCAG (PRD section 9 gate). Haptic tick on each decision. |
| S36 | Filters | Chip row: cuisine categories + a "drinks" chip (bars/cocktail spots). Filters are session-scoped, stack with the blend (filters narrow, never override vetoes), and visibly show active state. Clearing filters restores the full deck without regenerating. |
| S37 | Deck exhaustion | Swiped through everything: end card summarizes ("You passed on 18, shortlisted 3"), offers shortlist review, filter widening, or radius expansion. Never an infinite feed of junk; scarcity is honest here. |
| S39 | Menu access | One tap from card detail opens the venue's curated menu URL in an in-app browser. Venues without a stable menu URL fall back to website, then tel:. Menu coverage is a curation requirement for the top 100. |
| S40 | Photos | 3 to 5 photos per venue, full-bleed on cards. Next 3 cards' images prefetched so swiping never shows a spinner. Blurhash placeholder on slow connections. Source attribution rendered where licensing requires it. |

### Stage 4: Shortlist, decide, book

| ID | Scenario | Required MVP behavior |
|----|----------|----------------------|
| S38 | Shortlist review and decide | Shortlisted cards in a compact compare view (photo, fit, rating, price, distance, tier). Picking one (or a dinner + drinks pair) creates the plan. Shortlist persists for the session and syncs to the couple so either partner can look. |
| S19 | Tier A live availability | "Available now" badge + one-tap deep link to OpenTable/Resy. (Framework built in MVP; effectively dormant until a partner API exists.) |
| S20 | Tier B stale cache | "Last confirmed available 23 min ago" + booking link. Timestamp always honest. |
| S21 | Tier C pattern-inferred | "Usually has tables Tue/Wed at 7 PM" + one-tap call. This is the MVP workhorse tier. |
| S22 | Tier D unavailable | Card carries an "unlikely tonight" badge and is down-ranked, with the next-best same-category venue ranked above it. The transparent-swap note appears if the user shortlists it anyway. |
| S23 | Availability source outage | Tiers degrade (A drops to B drops to C) and the deck is never blocked by an availability failure. Phone number is the floor. |
| S24 | ~~Mixed availability across acts~~ | Deferred with multi-act itineraries. ID retired, not reused. |

### Stage 5: During the date

| ID | Scenario | Required MVP behavior |
|----|----------|----------------------|
| S25 | Plan offline | Saved plans cached on device (including card images already in the image cache). Full plan readable with no connection. tel:, maps, and menu links work natively or degrade gracefully. |
| S26 | Generation-time API outage | Venue data, fragments, and photo URIs are all in our own DB, so deck generation works even if every external API is down. Photos degrade to cached copies; weather note degrades to seasonal default. |

### Stage 6: Feedback, learning, accounts

| ID | Scenario | Required MVP behavior |
|----|----------|----------------------|
| S27 | Feedback prompt | Next open on or after the morning after the planned date: "Would repeat?" + mood descriptor chips. One tap to dismiss, never nags twice for the same plan. |
| S28 | Feedback skipped | Plan marked completed-unconfirmed. Still counts toward the account-offer counter. No learning update from feedback (swipe signals still count). |
| S29 | Learning applied | Explicit feedback is the strongest signal: would-repeat = yes boosts that venue and its attributes for this couple. Swipes are weak signals: rights nudge attribute affinities up, lefts nudge down, with small weights (a pass can mean "ate there yesterday", not "hate it"). Next session visibly reflects it ("because you loved X"). Deterministic rules, no ML in MVP. |
| S30 | Account offer at 3 completions | Optional, dismissible, additive (sync + portability pitch). Declining changes nothing (PRD: guest mode is full functionality). |
| S31 | Account created | Local profile, quizzes, couple link, history migrate to the account. Second device signs in and syncs. |
| S32 | Cleared storage / new device, no account | Cold start with honest messaging: "no account means this device starts fresh". This IS the sync value prop; never guilt-trip. |
| S33 | Data export / delete | Export preference data + history as JSON. Delete wipes server-side anonymous records and instructs on clearing local data. GDPR-style even for US users. |
| S34 | Content flag | Any venue description, photo, or fragment can be flagged; flags queue for admin review (PRD community moderation). |

---

## 3. Spec gaps and PRD deviations, with resolutions

Items 1 to 6 are ambiguities found in the PRD. Items 7 to 10 were deliberate deviations introduced by the v2 pivot, folded into PRD v2.2.

1. **Vetoes are not in the 5-question quiz.** The quiz (PRD 1.1) covers neighborhood, energy, radius, budget comfort, structure. But the blend's hard vetoes (PRD 2.1) need dietary restrictions, accessibility needs, and an absolute budget ceiling. Resolution: onboarding = 5 personality questions + a separate "hard requirements" step. Also keeps "budget comfort zone" (soft, blendable) distinct from "absolute ceiling" (veto), which the PRD itself distinguishes.
2. **Who sends the invite?** PRD says "shares an invite link via SMS or email". Resolution: the user sends it themselves via the device share sheet. No SMS provider, no email infrastructure, no partner PII collected. The 48h nudge is in-app on A's next open (we have no channel to reach an anonymous A anyway).
3. **"Completed plan" is undefined** but drives the account offer (3 completions) and the feedback prompt. Resolution: saved plan whose planned date has passed = presumed completed. Feedback response upgrades it to confirmed. Presumed counts toward the account-offer counter.
4. **Rotation advance trigger is undefined.** Resolution: rotation primacy advances only when a session produces a plan or an outbound booking action, not on browsing or swiping alone. Browsing shouldn't burn a partner's turn.
5. **Device fingerprinting (PRD Data Governance) conflicts with the privacy stance.** Resolution: drop it. The anonymous profile UUID already provides continuity without fingerprinting's privacy smell.
6. **Anonymous-but-paired tension.** PRD says anonymous preferences live on-device, but async invite and solo-partner-joins-later require both quizzes server-side to compute the blend. Resolution: minimal anonymous server record (opaque UUID + quiz answers + couple link, zero PII, deletable via S33). History and saved plans stay local-first with a server copy for couple sync.
7. **Platform (PRD says web-first).** v2 decision: Android app (Expo/React Native), sideloaded APK for alpha; the only web surfaces are the admin tool and the invite-landing quiz page (S2). iOS and public web deferred to the launch-strategy conversation.
8. **Core experience (PRD section 3 says 3 to 5 narrative itineraries).** v2 decision: swipe deck of individual venue cards. Pre-authored narrative fragments survive as card blurbs; multi-act assembly is deferred (see scope cuts). PRD section 3 was rewritten in v2.2.
9. **Evening structure question (quiz Q5).** With restaurants + bars only, the PRD's options (dinner + live music / + show / + late-night) can't be honored. Resolution: Q5 narrows to dinner only / dinner then drinks / drinks then dinner / just drinks. Original options return with Phase 3 orchestration.
10. **Scores on cards.** "Most accurate score" = two numbers max per card for cognitive load: the Google+Yelp count-weighted rating and a blend-fit percentage (from deck scoring). Couple compatibility score stays on the blend screen only.

---

## 4. MVP scope cuts (explicit)

In the PRD but deliberately deferred, so the build doesn't sprawl:

- **Multi-act narrative itineraries**: the founding concept, deferred by the v2 pivot to lock the swipe UI first. Fragments, blend, and scoring all survive and feed the deck; act-chaining logic and venue-to-venue distances are parked (essence preserved in the technical design appendix).
- **Explore/map mode**: Phase 2 per launch strategy. MVP is the deck only. No map SDK dependency at all in MVP.
- **Live partner availability (Tier A)**: requires OpenTable/Resy partnership that a prototype won't get. Tier framework ships in MVP; A activates when a partnership lands.
- **Event API integrations** (Ticketmaster/AXS): manual event-flag table in MVP, admin-entered for Bushnell/PeoplesBank/Trinity Health dates.
- **iOS, public web app, Play Store listing, push notifications, SMS**: all deferred to the launch-strategy conversation. Notifications are in-app banners on next open.
- **Couple swipe-matching** (both partners swipe independently, agreement surfaces as "it's a date"): the obvious Hinge-mechanic extension of the blend, and probably the killer feature. Deliberately post-MVP; the deck determinism (S13) and couple sync (S38) are designed so it can be added without rework.
- **ML-based learning**: deterministic scoring boosts only. Revisit when there's enough swipe + feedback data to matter.
- **Review sentiment analysis (Tier 2)**: Google + Yelp rating/review-count snapshots at curation time are enough for 100 hand-picked venues. TripAdvisor stays out.
