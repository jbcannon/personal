# Alabama/Georgia/Florida Biodiversity Adventures — Project Plan

## 📍 RESUME HERE
**One tool now: `adventure_planner.html`** (renamed from `alabama_ranker.html` — the old name stopped making sense once GA/FL and the map were folded in). Four tabs: ⚔️ Compare, 📋 Ranking, 🗺️ Map, 📊 Stats. Fully local, no server, no publishing — same as always.
- **Compare / Ranking** — unified AL+GA+FL ranker (146 active places), still in progress. Original 54 AL places keep their real Elo from the first 150-round session; the 92 new AL-supplement/GA/FL places are seeded with Claude's similarity-based estimate, so this is fine-tuning, not starting over. Progress bar tracks avg-times-seen (~8× good cut, ~15× solid). Export ranking (💾) is the only way Claude can see your progress, and now also exports the full head-to-head match log, not just the aggregate Elo.
- **Map** — all 152 places (including the 6 non-public ones, clearly flagged), satellite/street basemap toggle, real driving-time contour lines from home (despiked, labeled like elevation contours), visited/paddling/state filters, home-location icon. This absorbed the old standalone `adventure_map.html` (see note below).
- **Stats** *(new)* — dot-plot histogram of the Elo distribution (hover for details), Elo-vs-drive-time scatter with median quadrant lines ("better/worse than expected for the drive"), per-state and per-region breakdowns, paddling-availability split, and a consistency panel that flags any matchup where you picked differently the two times you saw it (powered by the new match-log history, which only started recording from this point forward).

Every place has a `paddling` field (researched, not just keyword-guessed) — 74 of 152 are paddling-relevant, shown as a 🛶 tag/filter.

**Cleanup done:** deleted the now-redundant standalone `adventure_map.html`, the stale Quarto-rendered `PROJECT_PLAN.html` export (+ its `PROJECT_PLAN_files/` folder), and the Phase 1/2 intermediate files that were fully superseded by the JSON databases (`alabama_biodiversity_places.md`, `alabama_places_database.md`, `southern_wonder.txt`). Kept `alabama_ranking_export.json` and `trip_summary.csv` even though they're stale snapshots (from Oct 12/14) — they're the only on-disk record of past ranking progress, since the live Elo only lives in the browser's localStorage; re-export from the Ranking tab when you want a fresh one.

**Fixed this round:** a real bug where the drive-time contours silently failed to render (isochrone data was nested in the wrong JSON shape), two rounds of mojibake corruption in the AL/FL source databases (em-dashes, then degree signs in "68°F"-style text) traced back to earlier background-agent edits, horizontal scrollbars on the brick wall and the Stats charts on narrow windows (both now scale to fit instead of scrolling), unreadable black axis labels on the dark theme, and a brick-wall click handler that was silently resetting a place's score (removed — bricks are now display-only). The brick wall itself was also redesigned to position each brick by its real Elo value along an axis (best on the left) rather than evenly by rank, so it visually reads as a live strip plot of the whole distribution.

---

Goal: build a ranked, richly-detailed, map-based trip-planning tool (for use with family) covering AL/GA/FL biodiversity hotspots — state/national parks, preserves, refuges, WMAs, caves, botanical gardens, private parks.

Home base for distance calcs: **Albany, GA**.

Status key: ✅ done · 🔄 in progress · ⏳ planned (blocked on earlier phase) · ⬜ planned

---

## Phase 1 — Extract AL places from "Southern Wonder" ✅
Scrape `Duncan, R. Scot - Southern Wonder...pdf` (already extracted to `southern_wonder.txt`) for every specific named, visitable biodiversity site in Alabama mentioned in the book.
- Output: `alabama_biodiversity_places.md` — 60 places catalogued, region-grouped, with a flagged list of 7 places NOT open to the public.

## Phase 2 — Build AL location database ✅
- Output: `alabama_places_database.json` (59 records) + `alabama_places_database.md`.
- Fields: name, location, lat/lon (estimates), flora/fauna, unique/special note, best time of year, driving distance from Albany GA, public access, source traceability.
- ✅ Follow-up enrichment done: `scenic_highlights` + `photos` fields added to all 59 records. **57 of 59** now have a real downloaded photo in `photos/` (~18 MB total, already resized/compressed, EXIF-orientation-corrected). Only 2 still unillustrated (Fred T. Stimpson WMA, Watercress Darter NWR — genuinely no verifiable place-specific photo found anywhere).
  - Per user call: unlicensed/all-rights-reserved photos are fine since this is personal-use-only, never published — `license` field on each photo notes the real source/terms so this is easy to audit if that ever changes.

## Phase 3 — Excitement ranking tool ✅ (ready — ranking underway)
Redesigned from a straight drag-sort of 59 items (too overwhelming) to a pairwise/triplet comparison game with Elo-style scoring — compare 3 places at a time, tap most-to-least exciting, repeat; ranking refines as you go and never requires finishing. A Ranking tab shows the live leaderboard with drag-to-fine-tune as a manual override.
- **Pivoted to fully local**: no claude.ai artifact/online dependency. Tool lives at `alabama_ranker.html` in the project folder, reads photos from the local `photos/` folder, and saves progress to the browser's localStorage. Open by double-clicking the file.
- Fixed along the way: an EXIF-orientation bug in the photo compression pipeline (rotated/stretched thumbnails, e.g. Coosa WMA) — re-downloaded and re-processed all photos with correct orientation handling; and an HTML-escaping bug in the placeholder-icon fallback markup that could corrupt card rendering.
- Cards show full scenic/special-note description, flora & fauna, and auto-detected activity tags (🥾 Hiking, 💦 Waterfalls, 🕳️ Caving, 🦅 Birding, etc. — inferred from keywords, not a separate data field). 3-column layout on wide windows so all 3 options are visible without scrolling; slim header.
- **Simulated convergence** (same triplet/Elo logic, synthetic true ranking + comparison noise): ~150 rounds → ~75-80% match with true top 25%; ~300 rounds → ~85%+; diminishing returns past that. Tool has a live progress bar toward these two milestones.
- **💾 Export ranking** button (Ranking tab) downloads `alabama_ranking_export.json` — this is the ONLY way to get ranking data out, since the tool is fully local (localStorage, no online db Claude can read).
- Note: this first pass ranks the 59 book-sourced AL places. The AL-supplement places added in Phase 4 (below) get folded into ranking/prioritization later (Phase 7), since they don't exist yet at this point in the sequence.

## Phase 4 — AL supplemental search + Georgia list ✅
**Added per user note:** don't fully trust the book author to have found every great AL spot. So this phase does TWO web-research passes:
1. **AL supplement** — search the web (independent of the book) for notable Alabama biodiversity/scenic destinations NOT already in `alabama_places_database.json`, using the same kinds of places the user ranked highly in Phase 3 (hiking, rare species, distinctive ecosystems, dramatic scenery) as the template for what's worth including. Append new records to the AL database (same schema), clearly tagged with a `source` of "web supplement" vs. "Southern Wonder book" so provenance stays visible.
2. **Georgia list** — no book reference for GA, so build an equivalent list from web research from scratch, same schema, same "what the user likes" template.

## Phase 5 — Florida list ✅
Same web-research approach as the GA half of Phase 4, for Florida.

### Results (all 3 ran in parallel, per user call):
- **AL supplement**: `alabama_places_database.json` grew from 59 → **73 records** (14 new, tagged `source_section: "Web supplement (not in Southern Wonder)"`). Standouts: Russell Cave NM, Majestic Caverns, Rickwood Caverns (blind cave fish), Stephens Gap Cave (143-ft skylight pit + waterfall), Noccalula Falls, Horse Pens 40, Historic Blakeley SP (Mobile-Tensaw Delta), Wetumpka Impact Crater. 12/14 have photos.
- **Georgia**: new `georgia_places_database.json`, **38 records**, 5 regions (Blue Ridge, Ridge & Valley/Cumberland Plateau, Piedmont, Coastal Plain/Okefenokee, Coastal GA & Barrier Islands). 38/38 have photos.
- **Florida**: new `florida_places_database.json`, **41 records**, 6 regions (Panhandle, North FL Springs Country, Central FL, Gulf Coast, Atlantic Coast, South FL/Everglades) — leaned into FL's own version of the AL taste profile (karst caves, first-magnitude springs, wildlife spectacle). 40/41 have photos.
- `photos/` folder now has **149 files, ~56 MB** total.
- All three verified: valid JSON, zero duplicate ids, photos spot-checked as real images (not error pages) on disk.

**Ranking signal used to target Phases 4-5** (from the AL export, 150 rounds, top 14 of 54 = true top 25%):
- Loves: caves/caving (Cathedral Caverns #1, Dismals Canyon #2), dramatic canyons & overlooks (Walls of Jericho, Little River Canyon, Buck's Pocket, Monte Sano), waterfalls (Bankhead/Sipsey "Land of a Thousand Waterfalls," DeSoto Falls), rare/singular phenomena (bioluminescent dismalites, fossil trackway, endemic Pygmy Sculpin, Cahaba lily bloom), wildlife-refuge spectacle (Sandhill Crane, Wheeler NWR).
- Sank to the bottom: obscure WMAs with no named standout feature, thin "brief mention" entries, non-biodiversity historic sites (Moundville).
- Caveat: thin source material partly confounds "less exciting" with "less described" — worth writing GA/FL entries with real descriptive substance rather than assuming obscure = bad.

## Phase 6 — Master database with full enrichment ✅
All 152 places (73 AL + 38 GA + 41 FL) now have: official_link, adventurer_review_link, notable_taxa, hike_rare_different_note, overnight_town, camping {available, name, notes}, must_see_trails, nearby_linkages, and (added per user request) `paddling` {available, note} — a properly-researched field, not just a keyword guess, since river/lake/spring places don't always say "kayak" explicitly. 74/152 are paddling-relevant.
- AL needed 2 follow-up passes: the enrichment agent that spawned its own nested sub-agents got stuck 3x (resolved by relaunching without nested agents); the first successful pass also ran low on shared WebSearch quota and needed a link-gap-filling follow-up (official_link: 53→70/73, adventurer_review_link: 8→56/73 — remaining nulls are genuinely obscure/closed-access places with no findable page).
- elo_seed (real score for the 54 originally-ranked AL places, Claude's estimate for the other 92) is baked into both this database and the ranker tool.

## Phase 7 — Map + checklist web app ✅ (superseded by Phase 8 merge)
Originally built as a standalone `adventure_map.html` (Leaflet, OpenStreetMap tiles, 🗺️ Map / 📋 Checklist tabs, all 152 places). Later merged into the main ranker as its Map tab (Phase 8) — the standalone file still exists on disk but is no longer the maintained version.

## Phase 8 — Unified tool + Stats tab ✅
Merged the standalone map into the ranker as a third (then fourth) tab, and renamed `alabama_ranker.html` → **`adventure_planner.html`** since "alabama" stopped describing a tool covering AL+GA+FL with a map and stats. Also added:
- Satellite/street basemap toggle (Esri tiles — CARTO's free tier turned out to require an API key despite returning HTTP 200, so verified Esri visually before committing to it), a home-location marker, and real driving-time contour lines from Albany GA (OSRM table API, despiked for routing-noise spikes, rendered as thin labeled contour lines rather than filled bands).
- A `best_time_of_year` research pass tied to each place's actual signature draw (migration timing, bloom windows, cave-tour schedules) rather than generic season guesses, with non-seasonal places explicitly marked "Anytime."
- The brick-wall progress visualization went through several redesigns: single row of vertical bars, embedded "seen" tick marks, a hover "peek" effect linking a Compare tile to its brick, and finally repositioning every brick along a true Elo axis (not evenly by rank) so the wall reads as a live strip plot of the score distribution — collision-avoided so bricks never overlap, and scaled to the container width so it never needs a scrollbar.
- A **Stats tab**: Elo dot-plot histogram (hover for details), an Elo-vs-drive-time scatter with median quadrant lines, per-state/region breakdowns, and a consistency panel built on a new persisted match-log (every round's 3 head-to-head outcomes, not just the rolled-up Elo) so flip-flopped matchups can be surfaced later.
- Two rounds of mojibake cleanup in the AL/FL source JSON (corrupted em-dashes, then corrupted degree signs), traced to encoding mistakes made by earlier background research agents rather than to the splicing step.

---

## Notes / running decisions
- Order is mostly sequential (each phase feeds the next), so phases will be executed one at a time rather than in parallel.
- Logistics (driving distance, trip-looping, shopping towns) are deliberately kept OUT of the Phase 3 excitement ranking, and folded in during Phase 6.
- The book ("Southern Wonder") is treated as a strong starting point, not a ceiling — Phase 4 explicitly supplements it with independent web research so good AL spots the author missed/omitted still make the list.
