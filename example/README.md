# GeoTileset with a tileset below the WGS 84 ellipsoid

Two copies of the same four-tile OGC 3D Tiles tileset, made by MB-System
mbmesh from swath bathymetry of the seafloor. Only the root transform
differs: `lifted/` is moved 2400 m up along the ellipsoid normal, so its
heights are positive.

- `deep.html`   the seafloor at about -2189 m (below the ellipsoid)
- `lifted.html` the seafloor at about +211 m

Both pages use the same GeoViewpoint, 60 m above the seafloor, looking
straight down, with a GeoOrigin at the seafloor.

## Run

Put a build of x3dom master (x3dom-full.js, x3dom.css) in this directory,
then:

    python3 -m http.server 8000

and open http://127.0.0.1:8000/deep.html and .../lifted.html. The overlay
shows what GeoTiles computed (selected tiles, viewport zoom, pitch).

## Expected and observed

Expected: both pages select tiles (the camera is 60 m from them).

Observed with x3dom 1.8.4-dev build 7527 (2026-10-15), on a larger
MB-System tileset of the same kind, in Chrome: no tile is selected
(#TILES 0), with viewport zoom 25.08 and pitch 180.

Not yet confirmed: that lifted.html selects tiles. The cause below comes
from reading GeoTiles.js and from the zoom and pitch values above;
please run both pages and compare.

## Cause

In GeoTiles.onBeforeCollectChildNodes, gd.z is the camera height above
the ellipsoid. With gd.z < 0:

- the point used for the look direction is `(0, 0, -gd.z * 0.01)`,
  which is behind the camera, so pitch comes out as 180 degrees;
- `elevation: Math.max(gd.z, 1)` clamps to 1 m, so zoom is 25.

Suggested fix: use `Math.abs(gd.z)` (or the distance to the tileset's
bounding volume) for both.
