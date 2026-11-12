# sea-view-data

Open-data artefacts for [Sea View](https://api-production-ae77.up.railway.app): a
world port gazetteer and coastal vector tiles, derived from OpenStreetMap, the
NGA World Port Index, EMODnet Human Activities and improved-un-locodes.

## What this is

This repository is the **data channel** of Sea View. The application server is
elsewhere; here there is no code, no configuration and no secret. Its only
purpose is to publish, under an explicit licence and with an explicit
provenance, the files the server downloads at boot.

Two rules hold for everything in it:

1. **Binaries live only in Releases.** A gazetteer database is hundreds of
   megabytes and a tile archive up to two gigabytes; committing either would put
   it in the history for ever. Every artefact is a *release asset*, and the
   repository itself carries only this README, the licence texts and a
   `manifest.json` per release. See `.gitattributes`.
2. **No personal data, no vessel positions, no code.** The artefacts describe
   *places* — ports, marinas, quays, coastlines. No AIS track, no vessel
   identifier, no name of any person, and nothing derived from an individual
   hull's movements is published here, now or later.

## Releases

Each release is tagged `data-vYYYY.MM.DD` (with a `-2`, `-3`, … suffix when a
day needs more than one) and carries a `manifest.json` listing every artefact of
that release:

```json
{ "schema": 1, "tag": "data-v2026.09.14", "createdAt": "…",
  "artifacts": [{ "name": "ports", "file": "ports.sqlite", "sha256": "…",
                  "bytes": 123456789, "kind": "gazetteer", "license": "ODbL-1.0",
                  "sources": ["…"], "required": true }] }
```

The server fetches `manifest.json`, downloads the artefacts it needs, and
verifies each one against its `sha256` before using it. An artefact carried over
unchanged from an earlier release appears with an absolute `url` pointing at the
release that actually holds the bytes, so a large file is uploaded once.

## Artefacts

| Artefact | File | Kind | Licence | Status |
|---|---|---|---|---|
| `ports` | `ports.sqlite` | fused port gazetteer | ODbL 1.0 (derivative database) | `data-v2026.11.10` (OpenStreetMap: Europe only) |
| `coast-<region>` | `coast-<region>.pmtiles` | coastal vector tiles | ODbL 1.0 | `coast-europe`: `data-v2026.11.12`; other regions: no release yet |
| `basemap-<region>` | `basemap-<region>.pmtiles` | fallback basemap tiles | ODbL 1.0 | no release yet |

The table is filled in as the artefacts are published.

---

## `ports` — the fused port gazetteer

**Status: first release `data-v2026.11.10`** — `ports.sqlite`, 133,980,160
bytes, sha256
`b277d84ada0a363e3c8d5fe3e3cbbbfe9197735a3bfea5082b013222cd71297c`, built
2026-11-10.

A single SQLite database of ports, marinas, harbours and their berthing
structures, fused from four sources with a deterministic rule set, used by Sea
View to name the place a yacht is moored or anchored at.

**Coverage of this release.** The OpenStreetMap part is **Europe only**:
Geofabrik's `europe` region (`europe-latest.osm.pbf`, Last-Modified
2026-11-04; it includes Turkey and Cyprus). The World Port Index, the EMODnet
marinas and UN/LOCODE are **worldwide**, so outside Europe a port is a WPI or
EMODnet point, without an OSM outline or berths. Later releases add the OSM
extracts of the other continents.

**Contents.** 27,973 ports (20,946 named; 13,426 with an OSM polygon, 14,547
points) in 207 countries — marina 17,301, harbour 5,439, anchorage 2,276,
dock 1,575, port 1,382; 396,931 berths (piers, quays, pontoons, breakwaters),
142,269 of them attached to a port; 116,067 UN/LOCODEs for decoding AIS
destinations, 5,244 linked to a port; 55,316 name aliases. Tables: `ports`
(with an R*Tree), `aliases`, `berths` (with an R*Tree), `locodes`, `meta`
(licence, notices, counts, sources), `provenance` (every input file with its
sha256) and `exports` (`ports.geojson`: 20,945 named ports of rank ≤ 3 as
points, 3.9 MB). The licence notices travel inside the file (`meta` keys
`licence` and `notice.*`).

**Verify.**

```sh
curl -sLO https://github.com/aithos-rr/sea-view-data/releases/download/data-v2026.11.10/ports.sqlite
echo "b277d84ada0a363e3c8d5fe3e3cbbbfe9197735a3bfea5082b013222cd71297c  ports.sqlite" | sha256sum -c
```

**Origin and licence.** The result is an **ODbL Derivative Database**: it is
produced from OpenStreetMap data and is published under the same licence.

> Contains information from OpenStreetMap, which is made available under the
> Open Database License (ODbL).

Licence text: [`LICENSES/ODbL-1.0.txt`](LICENSES/ODbL-1.0.txt), also at
<https://opendatacommons.org/licenses/odbl/1-0/>.

**Sources and attribution.**

- **OpenStreetMap**, via [Geofabrik](https://download.geofabrik.de/) extracts —
  © OpenStreetMap contributors, ODbL 1.0
  (<https://www.openstreetmap.org/copyright>). Harbour, marina, pier, quay,
  breakwater and mooring features.
- **World Port Index**, National Geospatial-Intelligence Agency (NGA) — public
  domain (17 U.S.C. §105). Naming the agency triggers a prescribed disclaimer,
  reproduced here verbatim:

  > World Port Index — National Geospatial-Intelligence Agency (NGA), public
  > domain. This reproduction, partial or complete, of any National
  > Geospatial-Intelligence Agency (NGA), National Imagery and Mapping Agency
  > (NIMA) or Defense Mapping Agency (DMA) product, information or data is not
  > approved, authorized or endorsed by the Secretary of Defense, the Director
  > of National Intelligence, the Director, NGA or any other element of the US
  > Government. The US Government and NGA accept no liability for the accuracy
  > or quality of this reproduction or the use of any NGA, NIMA or DMA products,
  > information or data.

  Source: <https://msi.nga.mil/commercial-use>. In `data-v2026.11.10` the
  input is the Internet Archive's byte-exact capture of NGA's official
  Pub. 150 CSV as NGA served it on 2026-09-29 (sha256
  `315644f1e77966291633145fd351c2762b37702d115926b9ed074b39e8e21667`):
  NGA's download answered HTTP 503 on 2026-11-05 and again on 2026-11-10.
- **EMODnet Human Activities** marinas (berth counts, curated names) —
  Data from EMODnet Human Activities (European Commission, DG MARE), CC BY 4.0.
  Licence text: [`LICENSES/CC-BY-4.0.txt`](LICENSES/CC-BY-4.0.txt).
  Source: <https://ows.emodnet-humanactivities.eu/wfs>.
- **improved-un-locodes** — <https://github.com/cristan/improved-un-locodes>.
  Geocoded UN/LOCODEs and their aliases, combining PDDL, Nominatim (ODbL) and
  Wikidata (CC0) inputs; treated as ODbL, which is what the fused database is
  published under.

---

## `coast-*` — coastal vector tiles

**Status: first archive `coast-europe` in `data-v2026.11.12`**: `coast-europe.pmtiles`,
287,534,736 bytes, sha256
`a3088f3a75a0c2a21659ff62fcdf0594046300957d13d715ad4924b4c28dcce0`, built
2026-11-10. The other regions have no release yet.

These are PMTiles archives of the marine detail that a general-purpose basemap
does not carry: breakwaters, piers, quays, pontoons, moorings, harbour and
marina polygons, and seamarks. There is one archive per Geofabrik region.
They are **optional**: the manifest marks them `required: false`. The Sea View
server starts without them and does not download them by default. The map
reads them with HTTP Range requests straight from the release asset.

| Region | File | OpenStreetMap data | Bytes | sha256 | Release |
|---|---|---|---|---|---|
| Europe | `coast-europe.pmtiles` | Geofabrik `europe`, 2026-11-04 (OSM up to 2026-11-03T21:24:23Z) | 287,534,736 | `a3088f3a75a0c2a21659ff62fcdf0594046300957d13d715ad4924b4c28dcce0` | `data-v2026.11.12` |

**Coverage of `coast-europe`.** Geofabrik's `europe` region
(`europe-latest.osm.pbf`, Last-Modified 2026-11-04, 35,372,035,423 bytes)
runs from the Azores to 46.75° E. It therefore includes western Russia,
Turkey, Cyprus and Georgia. It does not include the Canary Islands, which
Geofabrik files under Africa.

**Contents.** Vector tiles (MVT, gzip), zoom 10 to 15, with 1,506,691 tiles
and 1,014,700 features in four layers. Each OpenStreetMap element is one
feature, in the most specific layer it matches:

- `structure`: 440,705 features (piers, groynes, breakwaters, floating structures, quays, lighthouses, pontoons)
- `seamark`: 517,723 features (buoys, beacons, lights, moorings, notices and the rest of `seamark:type`)
- `harbour`: 48,599 features (slipways, marinas, harbours, docks)
- `mooring`: 7,673 features (`mooring=*`)

The attributes are `kind`, `name`, `seamark_type`, `floating` and `osm_id`.
The archives were built with
[Planetiler](https://github.com/onthegomap/planetiler) 0.10.2. Nothing was
reduced for size: the Europe archive is well under 1 GB.

**Verify.**

```sh
curl -sLO https://github.com/aithos-rr/sea-view-data/releases/download/data-v2026.11.12/coast-europe.pmtiles
echo "a3088f3a75a0c2a21659ff62fcdf0594046300957d13d715ad4924b4c28dcce0  coast-europe.pmtiles" | sha256sum -c
```

**Origin and licence.** The archives are built from OpenStreetMap data, using
Geofabrik extracts. They are a Derivative Database of OpenStreetMap and are
published under **ODbL 1.0**
([`LICENSES/ODbL-1.0.txt`](LICENSES/ODbL-1.0.txt), also at
<https://opendatacommons.org/licenses/odbl/1-0/>).

> Contains information from OpenStreetMap, which is made available under the
> Open Database License (ODbL).

When these tiles are rendered, display this attribution: **© OpenStreetMap
contributors** (<https://www.openstreetmap.org/copyright>).

---

## `basemap-*` — fallback basemap tiles

**Status: no release yet.**

An optional, self-hosted basemap extract, kept so the map still draws if the
upstream tile service is unavailable. One archive per region.

**Origin and licence.** Derived from the OpenMapTiles-schema tiles published by
[OpenFreeMap](https://openfreemap.org/), themselves built from OpenStreetMap
data, and published under **ODbL 1.0**
([`LICENSES/ODbL-1.0.txt`](LICENSES/ODbL-1.0.txt)).

> Contains information from OpenStreetMap, which is made available under the
> Open Database License (ODbL).

Attribution to display when these tiles are rendered: **© OpenMapTiles Data from
OpenStreetMap** (<https://openmaptiles.org/>,
<https://www.openstreetmap.org/copyright>).

---

## Licences in this repository

| File | Applies to |
|---|---|
| [`LICENSES/ODbL-1.0.txt`](LICENSES/ODbL-1.0.txt) | `ports`, `coast-*`, `basemap-*` and every future artefact derived from OpenStreetMap |
| [`LICENSES/CC-BY-4.0.txt`](LICENSES/CC-BY-4.0.txt) | the EMODnet Human Activities input to `ports` |

The World Port Index input is a work of the US Government and carries no
copyright; its disclaimer is reproduced above and must travel with any
reproduction that names the agency.

## Contact

Data controller: Riccardo Romano — <chiri.bsnss@gmail.com>.
