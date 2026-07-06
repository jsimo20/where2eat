# Quiz Copy (final draft for build)

The onboarding quiz: 5 personality questions + the hard-requirements step (journeys gap 1). Copy below is build-ready draft; James approves before milestone 2.

Microcopy at the top of the quiz: **"Five taps, then the non-negotiables. Takes about two minutes."**

## Q1. Where do you like your nights? (pick up to 3)

- Downtown Hartford
- West End
- Asylum Hill
- Frog Hollow / Parkville
- South End (Franklin Ave)
- West Hartford (Blue Back / the Center / Park Rd)
- Anywhere that's worth it

(Data note: maps to `neighborhood` multi-select; "Anywhere" clears the soft preference.)

## Q2. What's your kind of night out? (pick one)

- **Cozy & romantic**: low light, corner table, no rush
- **Lively & social**: a room with a pulse
- **Cultural & curious**: food with a story
- **Low-key & familiar**: the reliable spot, no decoding
- **Adventurous & new**: somewhere neither of you has been

## Q3. How far will you go for dinner? (pick one)

- 5 to 10 minutes
- 15 to 20 minutes
- 30+ minutes
- Anywhere, for the right place

## Q4. Where's your comfort zone on price? (pick one)

- $
- $$
- $$$
- $$$$
- Depends on the occasion

(Data note: soft preference. The hard ceiling is asked separately below; "Depends" = widest comfort zone.)

## Q5. How does the evening usually go? (pick one)

- Dinner only
- Dinner, then drinks
- Drinks, then dinner
- Just drinks

## Hard requirements step

Header: **"The non-negotiables."**
Sub: **"These are never broken, never bent, no matter what you or anyone in your group picks later."**

**Dietary needs (check all that apply):**
- Vegetarian options required
- Vegan options required
- Gluten-free options required
- Halal required
- Kosher required
- None

**Accessibility needs (check all that apply):**
- Wheelchair-accessible entrance and seating
- Accessible restroom
- Quieter rooms (sound-sensitive)
- None

**Absolute price ceiling (pick one):**
- Keep it under $$
- Keep it under $$$
- No ceiling

(Data notes: dietary and accessibility selections filter on the venue flags with those exact names; only options we can actually filter on are offered. Ceiling maps to `budget_ceiling`; in couples and groups the lowest ceiling wins. Copy must never present these as preferences.)

## Blend summary templates (shown after both quizzes)

- Aligned: "You and {partner} agree on {archetype} and {neighborhood}. Easy."
- Third-way: "You pull {archetypeA}, {partner} pulls {archetypeB}: tonight looks like {zone name}."
- Vetoes line: "Always honored: {veto list}."
- Rotation line (when applicable): "Tonight leans {name}'s way. Next time flips."

Status: DRAFT for James's approval. Question order fixed (personality first, requirements last, so the fun part builds momentum before the form-like part).
