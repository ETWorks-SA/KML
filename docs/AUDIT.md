# SANParks KML Audit &amp; Build Notes

Source of truth for everything in `/parks`. Read this before assuming a gap
is a bug rather than a documented missing input.

## Source material

`source/My_Places.kmz` — the Google Earth "My Places" export you provided.
It is a personal library (13.6MB), not a SANParks-specific product: most of
it (1,033 mountain passes worldwide, global shipwrecks, birding spots,
Greater Kruger private reserves) is out of scope and was **not** carried
into `/parks`. Only the `SanParks` subfolder was used.

`source/wdpa/part{0,1,2}` — the South-Africa-filtered WDPA (World Database
on Protected Areas) shapefile export you downloaded from Protected Planet
and added to git, split into its 3 delivered parts. Used for **Boundary**
geometry only (see below); the full multi-language documentation bundle
that ships with a WDPA download was not kept, only the shapefile
components (`.shp/.shx/.dbf/.prj/.cpg`).

**XML validity bug found and not carried forward:** `doc.kml` uses
`xsi:schemaLocation` on one `<Document>` node without ever declaring the
`xsi` namespace prefix — `xmllint` rejects the file outright on that line.
That node (`CCT Boundary`) is an empty, unrelated stub (Cape Town city
boundary, not a national park) and was dropped rather than fixed in place.

## Official park list vs. source coverage

19 SANParks-managed national parks. Naming corrected to SANParks' official
names where the source file had errors:

| Source name (as found) | Official name used in `/parks` |
|---|---|
| Khalagadi Transfrontier Park | Kgalagadi Transfrontier Park |
| Namaquland National Park | Namaqua National Park |
| Tankwa National Park | Tankwa Karoo National Park |
| Garden Route (Tsitsikamma, Knysna, Wilderness) National Park | Garden Route National Park |
| Augrabies National Park | Augrabies Falls National Park |
| \|Ai-\|Ais/Richtersveld Transfrontier Park | Ai-Ais Richtersveld Transfrontier Park |

**Entirely absent from the source file — no data to restructure:**
Golden Gate Highlands National Park, Mapungubwe National Park, Marakele
National Park. Their KML files exist in `/parks` with every category
folder present but empty, so they show up as gaps in Earth's sidebar
rather than being silently missing from the repo.

## Coverage matrix (placemark counts per category, per park)

Verified: for every park with source data, the sum of the counts below
equals the total placemark count found under that park's source folder —
nothing was dropped or double-counted during restructuring.

| Park | Boundary | Main Roads | Gates | Camps | Picnic | Bird Hides | Water Holes | Dams | Waypoints | POI | Other* |
|---|--:|--:|--:|--:|--:|--:|--:|--:|--:|--:|--:|
| Addo Elephant | 0 | 0 | 3 | 6 | 1 | 0 | 7 | 3 | 0 | 8 | 1 |
| Agulhas | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 2 | 1 |
| Augrabies Falls | 0 | 0 | 1 | 1 | 0 | 0 | 0 | 0 | 0 | 0 | 1 |
| Bontebok | 0 | 0 | 1 | 2 | 1 | 0 | 0 | 0 | 0 | 1 | 6 |
| Camdeboo | 0 | 0 | 2 | 2 | 0 | 1 | 0 | 0 | 0 | 2 | 1 |
| Garden Route | 0 | 0 | 1 | 2 | 0 | 0 | 0 | 0 | 0 | 0 | 1 |
| Golden Gate Highlands | 0 | 0 | 1 | 3 | 0 | 0 | 0 | 0 | 0 | 1 | 0 |
| Karoo | 0 | 0 | 1 | 2 | 2 | 0 | 0 | 0 | 0 | 3 | 1 |
| Kgalagadi Transfrontier | 0 | 0 | 4 | 32 | 6 | 1 | 45 | 0 | 14 | 10 | 11 |
| Kruger | 0 | 40 | 19 | 48 | 12 | 11 | 80 | 50 | 0 | 40 | 13 |
| Mapungubwe | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| Marakele | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| Mokala | 0 | 16 | 1 | 6 | 2 | 0 | 7 | 0 | 0 | 4 | 2 |
| Mountain Zebra | 0 | 0 | 1 | 1 | 2 | 0 | 0 | 0 | 0 | 4 | 1 |
| Namaqua | 0 | 0 | 2 | 10 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
| Ai-Ais Richtersveld Transfrontier | 0 | 0 | 2 | 8 | 0 | 0 | 0 | 0 | 24 | 10 | 0 |
| Table Mountain | 0 | 0 | 2 | 3 | 0 | 0 | 0 | 0 | 0 | 16 | 1 |
| Tankwa Karoo | 0 | 49 | 1 | 16 | 0 | 0 | 3 | 2 | 0 | 4 | 2 |
| West Coast | 0 | 0 | 3 | 1 | 0 | 5 | 0 | 0 | 0 | 3 | 1 |

\* "Other" = Viewpoints/Lookouts/Pans/Trails placemarks that existed in the
source but aren't one of your 10 requested categories. Kept rather than
deleted since it's real surveyed data; feel free to say drop it.

**Boundary is now populated for all 19 parks from WDPA — see below.** The
`My_Places.kmz` source itself still has zero boundary polygons for any
actual national park (only for unrelated Greater Kruger private reserves,
excluded as out of scope); the boundary column in the table above reflects
only the KMZ, not the WDPA layer added afterward.

## Boundaries (from WDPA)

All 19 parks now have a real boundary polygon, sourced from the WDPA
extract you added to git, not invented. Method used, so it can be checked:

- WDPA shapefiles use the standard Esri winding convention: outer
  boundaries are digitized **clockwise**, holes **counter-clockwise**.
  Each polygon "part" was classified by the sign of its shoelace-formula
  signed area (negative = outer, positive = hole) — no simplification, no
  smoothing, full original vertex count kept (~71,000 coordinate points
  total across all 19 boundary layers).
- Every hole was assigned to the specific outer section that spatially
  contains it (point-in-polygon test), so a private concession or enclave
  excluded from, say, Kruger's northern section doesn't get wrongly
  subtracted from a different section.
- Verified for every park: `outer_rings + holes == total shapefile parts`
  (nothing silently dropped) and zero holes were left unassigned
  (no orphans — every hole landed inside some outer ring).
- Parks with naturally disjoint land (e.g. Addo Elephant's 42 separate
  parcels — Zuurberg, Woody Cape, Kabouga, marine sections, etc.; Garden
  Route's 17 sections spanning Tsitsikamma/Wilderness/Knysna; Table
  Mountain's 28 sections) are rendered as a `MultiGeometry` of that many
  `Polygon`s, each with its own holes — not force-merged into one shape.
- Boundary line/fill uses the SANParks green (`#0d6129`) from `Colour.txt`.

**Naming note:** WDPA hasn't been updated for two park mergers/renames —
its record is filed under the pre-merger name, but the polygon is the
correct location for that park:

| WDPA record name | Matched to |
|---|---|
| Kalahari Gemsbok National Park | Kgalagadi Transfrontier Park |
| Richtersveld National Park | Ai-Ais Richtersveld Transfrontier Park |
| Mapungupwe National Park (WDPA's own typo) | Mapungubwe National Park |

Also worth knowing before you check this: **WDPA boundaries are
generalized outlines for global reporting, not survey-grade cadastral fence
lines.** Good for "where is the park," not for anything needing
meter-level precision — that would need SANParks' own cadastral GIS layer,
which nothing supplied so far contains.

**Three WDPA records were NOT matched to any of the 19 and are excluded
from `/parks` — your call needed:**

| WDPA name | Why excluded | What I'd want confirmed |
|---|---|---|
| Vaalbos National Park | (Likely) de-proclaimed in 2009 following land restitution claims — no longer a national park | Confirm it should stay excluded, or say if it needs including for historical reasons |
| Groenkloof National Park | SANParks-managed reserve near Pretoria; (guessing) usually referred to as a nature reserve rather than counted among the flagship national-park list | Say if you want it added as a 20th park file |
| Meerkat National Park | (guessing) Possibly the Northern Cape reserve associated with the SKA radio-telescope buffer zone — low confidence, not independently verified | Say if you want it added; I'd want to confirm what this actually is before building a file for it |

I did not silently drop or silently add any of these — the shapefile data
for all three is present in `source/wdpa/` if you want them built later.

## Design decisions made during restructuring

- **Classification logic:** placemarks were sorted into categories primarily
  by the folder they were already manually filed under in your source file
  (e.g. a placemark under "Entrance Gates" → Gates). Placemarks sitting
  loose at the park level with no subfolder (e.g. Bontebok, which had no
  subfolders at all) were classified by their icon style instead (gate icon
  → Gates, camp icon → Camps, etc.).
- **Camps** keeps its original subgrouping (Rest Camps, Bushveld Camps,
  Camping Sites, Tented Camps, Trail Camps, Private Camps and Lodges) nested
  inside the Camps folder where the source had it — that distinction is
  useful for trip planning and dropping it would lose information.
- **Main Roads** keeps each named road (e.g. Kruger's H1-1, H1-2...) as its
  own subfolder with its real `LineString` geometry, not just point markers.
- **One "Boundary in name only" placemark found:** in the Bontebok source
  data there is a single point placemark literally named "Boundary". It is
  **not** a boundary polygon — it's a point (likely a boundary beacon
  location). It was kept as a real placemark but do not mistake it for
  actual park-outline geometry.
- **No coordinates were invented.** Every geometry in `/parks/*.kml` is
  copied byte-for-byte from your source file's `<coordinates>`. Nothing was
  estimated from memory.
- **Icons:** categories with a real custom icon (`Gates`, `Camps`,
  `Picnic Spots`, `POI`, `Water Holes`, `Bird Hides`, and the bonus "Other"
  bucket) use that icon, copied to `/icons`. `Bird Hides` was upgraded from
  a placeholder to the real `birds-2.png` icon you added to git directly
  (staged as `source/icons/`, promoted to `/icons/birdhide.png`).
  `Dams` and `Waypoints` still have **no** dedicated icon anywhere in
  anything supplied so far, and currently fall back to Google Earth's
  standard hosted placeholder icons (colored circles) so the files stay
  fully renderable.
- **Boundary styling** (line + fill color) now uses the SANParks green
  `#0d6129` you provided in `Colour.txt`, applied wherever a boundary
  polygon eventually gets added. The polygons themselves are still empty —
  color alone doesn't create geometry.
- The full raw icon set you added to git (`knp4x4`, `knpairport`,
  `knpcamping`, `knpcaravan`, `knpsec`, `knpspoor`, `knptrails`, `knptent`,
  `knpnoentry`, etc.) is kept at `source/icons/` for provenance even though
  most aren't wired into a category yet — say if you want any of them
  mapped to a specific folder (e.g. `knpairport` for airstrips as a POI
  subtype, `knpcaravan`/`knpcamping` as Camps subtypes).

## Manually-sourced data

For categories with no digital source at all, you can retrieve
coordinates yourself from the SANParks website (or elsewhere) and hand
them over as a plain list — I'll place and style them consistently rather
than you having to write KML by hand. These live in `source/manual_data/
<slug>.json` per park (kept separate from the KMZ/WDPA provenance so it's
clear what came from where) and get merged in at build time.

**Golden Gate Highlands — first entries added** (`source/manual_data/
golden-gate-highlands.json`): West Gate (Gates), Glen Reenen Rest Camp
(Camps → Rest Camps), Golden Gate Hotel and Chalets + Highlands Mountain
Retreat (Camps → Private Camps and Lodges), Basotho Cultural Village (POI).

**Data quality flag — needs your check, not mine:** the coordinates you
supplied for Glen Reenen Rest Camp and Basotho Cultural Village share the
exact same longitude, 28.744250°E, matching to 0.1 arc-second (~3m). Two
different named landmarks matching that precisely is not something that
happens by chance — it reads like a copy-paste artifact in the source
list. Both points independently fall inside the park's WDPA boundary
polygon, so it's not an obviously wrong result, but that's a weak check
(the park spans a wide area) — it doesn't confirm the longitude is right
for Basotho specifically. I used the coordinate as supplied and flagged it
here and in the JSON source rather than silently trusting or silently
"fixing" it. Please re-check that one value against the SANParks site.

## Known gaps (need input from you, not more processing)

1. ~~Boundaries for all 19 parks~~ — **done**, from WDPA. See above.
2. **Golden Gate Highlands** now has a boundary plus a Gate, 3 Camps and 1
   POI (manually retrieved by the user from the SANParks website — see
   "Manually-sourced data" below). Still missing Picnic/Bird Hides/Water
   Holes/Dams/Waypoints. **Mapungubwe, Marakele** still have only a
   boundary — no other categories have any source yet.
3. **Thin parks** (Agulhas, Bontebok, Augrabies Falls, Garden Route,
   Camdeboo, Karoo, Mountain Zebra, Namaqua, Table Mountain, West Coast) —
   have partial category coverage beyond their (now real) boundary. Filling
   the empty categories needs more source data; it was not invented.
4. **Dedicated Dam and Waypoint icons** — not in the icon set added to git
   yet. Still using Google's generic placeholder circle for those two
   categories.
5. **Vaalbos / Groenkloof / Meerkat** — see the table above, need your
   decision on whether any of these should get a park file.

## Repo layout

```
source/My_Places.kmz   - your original export, kept as provenance/reference
icons/*.png            - shared category icons referenced by every park KML
parks/<slug>.kml       - one standalone KML per national park
docs/AUDIT.md          - this file
```
