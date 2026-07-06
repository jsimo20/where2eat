# Third-Way Archetype Matrix

Content for the blend engine (D9 in technical-design.md references `THIRD_WAY[a][b]`; scenario S10). When partners pick different energy archetypes, the blend surfaces one of these ten zones instead of averaging or picking a winner. The matrix is symmetric: `THIRD_WAY[cozy][lively] == THIRD_WAY[lively][cozy]`.

Archetype keys: `cozy` (cozy & romantic), `lively` (lively & social), `curious` (cultural & curious), `familiar` (low-key & familiar), `adventurous` (adventurous & new).

Each zone defines: the display name shown in the blend summary, what it means, and the venue traits the scorer boosts. Blurbs for a blend-zone deck use the closer archetype's fragment, chosen per venue by trait fit.

| Pair | Zone name | What it means | Venue traits to boost |
|------|-----------|---------------|----------------------|
| cozy x lively | An intimate booth in a lively room | One of you wants buzz, one wants closeness. Places with energy in the room and shelter at the table. | noise_level moderate, booth/corner seating noted, warm lighting, bar-forward rooms with real tables |
| cozy x curious | Candlelight with a story | Romance carried by substance: a chef with a point of view, a historic room, a menu worth talking about. | authenticity_note present, editorial_sources rich, warm lighting, quieter rooms, chef-driven |
| cozy x familiar | Your table, the quiet one | The comfort of a known place, pointed at its calmest corner. Repeats are a feature here, not a bug. | previously loved venues (would_repeat), neighborhood standbys, quieter rooms, unfussy service |
| cozy x adventurous | New place, old soul | Somewhere neither of you has been, that still feels warm the moment you sit down. | recently added venues, warm lighting, small rooms, adventurous cuisine tags in an intimate format |
| lively x curious | The scene with substance | A crowd worth joining because something real is happening: chef's counters, food halls, rooms with culture attached. | food halls, open kitchens, high editorial interest, busy + character tags |
| lively x familiar | The regulars' night | Bustle you already know how to enjoy: taverns, brewpubs, the standby where the crowd is the comfort. | taverns/brewpubs, drink_tags beer, high review counts, repeat-loved venues |
| lively x adventurous | Loud and untried | The newest, most energetic thing going. Late kitchens, big flavors, no guarantees. | recently added, noise_level high acceptable, late hours, adventurous cuisine tags |
| curious x familiar | Comfort with a backstory | Institutions and heritage spots: old-school rooms where the tradition is the experience. | long-operating venues, authenticity_note strong, classic cuisine categories |
| curious x adventurous | The field trip | The most exploratory zone: cuisines new to you both, tasting formats, the drive that's worth it. | cuisine categories the couple hasn't tried (learning data), radius-stretch candidates, strong editorial |
| familiar x adventurous | A safe bet that's new to you | Familiar format, unfamiliar venue: the new pizza place, the standby's sister spot. Low risk, fresh page. | recently added venues within loved cuisine categories, mid noise, mid price |

Rules of use:

- Surface at most 2 zones per blend (PRD 2.1 says 1 to 2); if the couple's learning data strongly favors one, lead with it.
- Zone names appear in the blend summary exactly as written ("You pull cozy, Sam pulls lively: tonight looks like an intimate booth in a lively room").
- Zones never override vetoes or filters; they only shape scoring.
- Status: DRAFT, awaiting James's read. Names are the product voice; edit freely.
