# South African National Parks KML

One KML file per SANParks-managed national park, restructured from a source
Google Earth export into a consistent category schema: Boundary, Main
Roads, Gates, Camps, Picnic Spots, Bird Hides, Water Holes, Dams,
Waypoints, POI.

See [`docs/AUDIT.md`](docs/AUDIT.md) for the full coverage matrix, what was
fixed, what was kept as-is, and what's still missing (boundaries for every
park, three parks with no data at all, and a couple of icons pending
upload).

```
source/My_Places.kmz   - original source export, kept for provenance
source/icons/          - full raw icon set + brand color, kept for provenance
source/wdpa/           - WDPA boundary shapefiles, kept for provenance
icons/*.png            - curated category icons used by every park KML
parks/<slug>.kml       - one KML per national park (now includes Boundary)
docs/AUDIT.md          - audit findings, coverage matrix, known gaps
```
