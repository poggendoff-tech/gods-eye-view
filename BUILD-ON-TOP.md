# Building on God's Eye View — Houston Seller Farm sketch

This note is a sketch for a future overlay on **this fork** ([poggendoff-tech/gods-eye-view](https://github.com/poggendoff-tech/gods-eye-view)). It is not implemented. Do not open it as a feature pull request against upstream [bilawalsidhu/gods-eye-view](https://github.com/bilawalsidhu/gods-eye-view).

How to run the globe first is in [ACCESS.md](ACCESS.md).

## The line

God's Eye View models **events, assets, infrastructure, and systems**. From the upstream README:

> It does not build features for named-person search, face recognition, or tracking individuals, and pull requests that cross that line won't be merged. People are not a query type here.

A Houston Seller Farm layer stays on the geography side of that line:

- ZIP polygons that define a farm territory.
- Parcel or property footprints inside those ZIPs: lot outline, land use, acreage, year built, building area, and a site address when it is a place, not a person.
- Camera bookmarks that fly the Cesium globe to a ZIP or a parcel.

Out of scope, including as a "later" field on the same layer:

- Owner names, occupant names, phones, emails, or any other person identifier, even when a county extract includes them.
- Skip-tracing, people search, or joining a parcel to a person graph.
- Face recognition, plate recognition, or following an individual across cameras or time.
- Private-data scraping. Public boundary and parcel geometry only, with provenance written down the way `src/data/local_data/neighborhoods/SOURCE.md` does.

If a public feed mixes geometry with owner columns, drop the person columns **before** the file is bundled or served. The globe should never receive them.

## What the overlay is

**Houston Seller Farm** is a farm-zip view for REISMS: a named set of Houston-area ZIP codes drawn on the Cesium globe, with property polygons inside the active ZIP. An operator picks a farm ZIP, the camera frames that polygon, and parcel footprints draw on the keyless Esri basemap (or on Cesium ion photorealistic 3D once a token is added in POWER UP). Clicking a parcel opens a property card. The card does not have a person.

Suggested stable ids, matching the existing `local-*` layers:

| Layer id | Draws |
| --- | --- |
| `local-farm-zips` | ZIP (or ZCTA) polygons for the configured Houston farm set |
| `local-farm-parcels` | Parcel polygons clipped to those ZIPs |

The ZIP list is configuration for the farm, not a people list and not something to invent in git. Start from public boundary files (Census ZCTA / TIGER, or a City of Houston open-data ZIP layer) and a public parcel polygon feed (Harris County or City of Houston GIS). Record the download URL, license, date, and the column drop-list in a `SOURCE.md` next to the data.

## Where it would plug in

The closest pattern already in the tree is the infrastructure GeoJSON layers, not a new viewer.

1. **Data.** Put simplified GeoJSON or GeoJSONL under `src/data/local_data/`, with a `SOURCE.md`. Keep files small enough to ship: simplify rings, round coordinates, and clip parcels to the farm ZIPs. Same hygiene as `scripts/build-sf-neighborhoods.mjs` for the San Francisco neighborhoods.
2. **Layer factory.** `createLocalGeoJsonLayer` in `src/data/localGeojsonCore.js` already loads a URL, paints entities, and publishes overlay entries. `createInfrastructureLayers` in `src/data/infrastructure.js` shows the options that matter: `id`, `url`, `name`, `color`, `source`, and label budget. A fork-local `createFarmLayers(services)` can return the ZIP layer and the parcel layer the same way.
3. **Registration.** `src/app/constructCatalog.js` spreads `createInfrastructureLayers(...)` into the catalog. A farm factory would be added beside that call. Share-link order lives in `LAYER_STATE_REGISTRY` in `src/data/layerState.js` (one unused token per layer, `enabled-only`). The DATA LAYERS panel groups live in `PANEL_GROUPS` in `src/ui/layerPanel.js` — a **Places** group next to Infrastructure, not under Cameras.
4. **Selection.** Parcel click metadata should be property attributes only (APN or account number as a parcel id, land use, area, site address). Do not register a person, owner, or camera-track field on the entity.
5. **Startup.** Keyless `npm run dev` is enough to see polygons on Esri imagery. Cesium ion (POWER UP) is the optional upgrade when the farm needs photorealistic buildings under the parcels. See [ACCESS.md](ACCESS.md).

Do not add a server proxy that fetches owner rolls, listing portals, or people-search APIs. Static, reviewed geometry keeps the layer inside the local-first model in [SECURITY.md](SECURITY.md).

## Fork boundary

Build and review this on `poggendoff-tech/gods-eye-view`. Upstream's responsible-use rule is the constraint to keep, not a request to merge the farm product back. A ZIP-and-parcel overlay that stays free of person search can still be offered upstream later as a generic local-GeoJSON example; the Houston farm configuration itself belongs on this fork.
