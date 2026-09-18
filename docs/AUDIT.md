# SANParks KML Audit &amp; Build Notes

Source of truth for everything in `/parks`. Read this before assuming a gap
is a bug rather than a documented missing input.

## Source material

`source/My_Places.kmz` — the Google Earth "My Places" export you provided.
It is a personal library (13.6MB), not a SANParks-specific product: most of
it (1,033 mountain passes worldwide, global shipwrecks, birding spots,
Greater Kruger private reserves) is out of scope and was **not** carried
into `/parks`. Only the `SanParks` subfolder was used.

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
| Golden Gate Highlands | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 | 0 |
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

**Boundary is 0 for every single park.** No boundary polygon exists in the
source data for any actual national park (only for unrelated Greater
Kruger private reserves, which were excluded as out of scope). This is the
one category that needs a new data source before it can be filled in —
see "Known gaps" below.

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

## Known gaps (need input from you, not more processing)

1. **Boundaries for all 19 parks.** Zero source data. Needs an authoritative
   file (SANParks GIS export, a GPS boundary track, or similar) uploaded
   directly to this session/repo. This session has no network access to
   pull public sources (OSM, Protected Planet/WDPA, Wikipedia) — outbound
   requests are blocked by environment policy.
2. **Golden Gate Highlands, Mapungubwe, Marakele** — no source data at all.
   Same as above: need an upload, not a link.
3. **Thin parks** (Agulhas, Bontebok, Augrabies Falls, Garden Route,
   Camdeboo, Karoo, Mountain Zebra, Namaqua, Table Mountain, West Coast) —
   have partial category coverage. Filling the empty categories needs more
   source data; it was not invented.
4. **Dedicated Dam and Waypoint icons** — not in the icon set added to git
   yet. Still using Google's generic placeholder circle for those two
   categories.

## Repo layout

```
source/My_Places.kmz   - your original export, kept as provenance/reference
icons/*.png            - shared category icons referenced by every park KML
parks/<slug>.kml       - one standalone KML per national park
docs/AUDIT.md          - this file
```
