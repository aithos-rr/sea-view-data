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
| `ports` | `ports.sqlite` | fused port gazetteer | ODbL 1.0 (derivative database) | `data-v2026.11.15` (OpenStreetMap: the seven Geofabrik continental regions and Antarctica) |
| `coast-<region>` | `coast-<region>.pmtiles` | coastal vector tiles | ODbL 1.0 | `coast-europe`: `data-v2026.11.12`; `coast-africa`, `coast-asia`, `coast-australia-oceania`, `coast-central-america`, `coast-north-america`, `coast-south-america`, `coast-antarctica`: `data-v2026.11.15` |
| `basemap-<region>` | `basemap-<region>.pmtiles` | fallback basemap tiles | ODbL 1.0 | no release yet |

The table is filled in as the artefacts are published.

---

## `ports` — the fused port gazetteer

**Status: `data-v2026.11.15`.** `ports.sqlite` is 282,505,216 bytes, sha256
`cfb93f85f2aa743662cc54ddc8a92e8ab5ea198988731d641c9dfa4da7e1cfd2`, built
2026-11-16. It replaces the Europe-only first release, `data-v2026.11.10`
(133,980,160 bytes, sha256 `b277d84a…`).

A single SQLite database of ports, marinas, harbours and their berthing
structures, fused from four sources with a deterministic rule set, used by Sea
View to name the place a yacht is moored or anchored at.

**Coverage of this release.** The OpenStreetMap part covers **eight
Geofabrik regions**: the seven continental regions (africa, asia,
australia-oceania, central-america, europe, north-america and south-america)
and antarctica. Europe is the same extract as in `data-v2026.11.10`
(Last-Modified 2026-11-04). The other six continents are Geofabrik's files of
2026-11-11, and Antarctica is Geofabrik's file of 2026-11-14. The World Port
Index, the EMODnet marinas and UN/LOCODE are worldwide.

The database records where its OpenStreetMap data reaches, in the table
`osm_coverage` and the `meta` key `osmCoverage`. These are Geofabrik's
clipping polygons, simplified and widened by 0.06° so they never cover less
than the originals. Outside them a port is a WPI, EMODnet or UN/LOCODE point,
without an OSM outline or berths. In this release that applies to one port:
the WPI harbour of Johnston Atoll, which no Geofabrik extract holds.

**Contents.** 53,326 ports (34,965 named; 30,390 with an OSM polygon, 22,936
points) in 237 countries: marina 30,306, harbour 6,915, dock 6,485, port
5,321, anchorage 4,299. 857,582 berths (piers, quays, pontoons, breakwaters),
268,907 of them attached to a port. 116,067 UN/LOCODEs for decoding AIS
destinations, 5,692 of them linked to a port. 72,025 name aliases.

Tables: `ports` (with an R*Tree), `aliases`, `berths` (with an R*Tree),
`locodes`, `meta` (licence, notices, counts, sources), `provenance` (every
input file with its sha256), `osm_coverage` (the OpenStreetMap coverage
polygons) and `exports` (`ports.geojson`: 34,964 named ports of rank ≤ 3 as
points, 6.5 MB). The licence notices travel inside the file (`meta` keys
`licence` and `notice.*`).

**Verify.**

```sh
curl -sLO https://github.com/aithos-rr/sea-view-data/releases/download/data-v2026.11.15/ports.sqlite
echo "cfb93f85f2aa743662cc54ddc8a92e8ab5ea198988731d641c9dfa4da7e1cfd2  ports.sqlite" | sha256sum -c
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
  breakwater and mooring features. The coverage polygons are Geofabrik's
  extract clipping polygons, an OpenStreetMap produced work under the ODbL
  1.0. Extract clipping polygons by Geofabrik GmbH (download.geofabrik.de).
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

  Source: <https://msi.nga.mil/commercial-use>. In `data-v2026.11.10` and
  `data-v2026.11.15` the input is the Internet Archive's byte-exact capture of
  NGA's official Pub. 150 CSV, as NGA served it on 2026-09-29 (sha256
  `315644f1e77966291633145fd351c2762b37702d115926b9ed074b39e8e21667`). NGA's
  download answered HTTP 503 on 2026-11-05, 2026-11-10, 2026-11-13 and
  2026-11-15.
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

**Status: eight regions.** `coast-europe` is in `data-v2026.11.12`. The other
seven (Africa, Asia, Australia–Oceania, Central America, North America, South
America and Antarctica) are in `data-v2026.11.15`, whose `manifest.json` also
lists `coast-europe` by `url`.

These are PMTiles archives of the marine detail that a general-purpose basemap
does not carry: breakwaters, piers, quays, pontoons, moorings, harbour and
marina polygons, and seamarks. There is one archive per Geofabrik region.
They are **optional**: the manifest marks them `required: false`. The Sea View
server starts without them and downloads them in the background, then serves
them to its map from its own origin. Release asset hosts do not allow a
browser to read them across origins.

| Region | File | OpenStreetMap data | Bytes | sha256 | Release |
|---|---|---|---|---|---|
| Europe | `coast-europe.pmtiles` | Geofabrik `europe`, 2026-11-04 (OSM up to 2026-11-03T21:24:23Z) | 287,534,736 | `a3088f3a75a0c2a21659ff62fcdf0594046300957d13d715ad4924b4c28dcce0` | `data-v2026.11.12` |
| Africa | `coast-africa.pmtiles` | Geofabrik `africa`, 2026-11-11 (OSM up to 2026-11-11T21:22:47Z) | 13,666,616 | `18ccf43bc50a04f0f30115ea97e1487563bc270fea165eb50a34d43f54e7978a` | `data-v2026.11.15` |
| Asia | `coast-asia.pmtiles` | Geofabrik `asia`, 2026-11-11 (OSM up to 2026-11-11T21:22:47Z) | 72,734,858 | `d43dbcfffee4f29c3b4148ed89bb7fecb93c7d7dc757bfb5fa09eeae373f212e` | `data-v2026.11.15` |
| Australia–Oceania | `coast-australia-oceania.pmtiles` | Geofabrik `australia-oceania`, 2026-11-11 (OSM up to 2026-11-11T21:22:47Z) | 29,337,150 | `f6651200191e04ef2a22fb761fd971837ec1a193a3359b0c40cae26546979498` | `data-v2026.11.15` |
| Central America | `coast-central-america.pmtiles` | Geofabrik `central-america`, 2026-11-11 (OSM up to 2026-11-11T21:22:47Z) | 8,552,042 | `20f320b5a32736901d88cc71218e238d156f8ca463d0d449ef2f2fd893716dc1` | `data-v2026.11.15` |
| North America | `coast-north-america.pmtiles` | Geofabrik `north-america`, 2026-11-11 (OSM up to 2026-11-11T21:22:47Z) | 105,613,672 | `2f5c026af7cbbdec44bd64f512180c020bd9ec609817e19f957bfe0589c18146` | `data-v2026.11.15` |
| South America | `coast-south-america.pmtiles` | Geofabrik `south-america`, 2026-11-11 (OSM up to 2026-11-11T21:22:47Z) | 31,361,058 | `6c45cd76a2a0f3dd9f82ff0a4d736bc571fea9864f4431c4519372d9f174a0c8` | `data-v2026.11.15` |
| Antarctica | `coast-antarctica.pmtiles` | Geofabrik `antarctica`, 2026-11-14 (OSM up to 2026-11-14T21:20:41Z) | 8,307,864 | `6719be6e7360263514ac455ec873f969ade333c589e1557db701a0db640e7592` | `data-v2026.11.15` |

**Coverage of `coast-europe`.** Geofabrik's `europe` region
(`europe-latest.osm.pbf`, Last-Modified 2026-11-04, 35,372,035,423 bytes)
runs from the Azores to 46.75° E. It therefore includes western Russia,
Turkey, Cyprus and Georgia. It does not include the Canary Islands, which
Geofabrik files under Africa.

**Coverage of the other seven.** Each archive covers one Geofabrik region.
Geofabrik's clipping polygons overlap (Mexico and Central America, for
example), so a feature near a border can be in two archives. The asia,
australia-oceania and north-america regions cross the 180th meridian, so their
bounds span every longitude. Antarctica's bounds are its region's box, south of
60° S.

**Contents.** Vector tiles (MVT, gzip), zoom 10 to 15, with 1,506,691 tiles
and 1,014,700 features in four layers. Each OpenStreetMap element is one
feature, in the most specific layer it matches:

- `structure`: 440,705 features (piers, groynes, breakwaters, floating structures, quays, lighthouses, pontoons)
- `seamark`: 517,723 features (buoys, beacons, lights, moorings, notices and the rest of `seamark:type`)
- `harbour`: 48,599 features (slipways, marinas, harbours, docks)
- `mooring`: 7,673 features (`mooring=*`)

The seven archives of `data-v2026.11.15` hold 728,875 features:

- Africa: 21,478
- Asia: 192,431
- Australia–Oceania: 73,855
- Central America: 21,155
- North America: 375,325
- South America: 43,952
- Antarctica: 679

They have the same four layers and attributes, and the same zoom range, 10 to 15.

The attributes are `kind`, `name`, `seamark_type`, `floating` and `osm_id`.
The archives were built with
[Planetiler](https://github.com/onthegomap/planetiler) 0.10.2. Nothing was
reduced for size: the largest archive, Europe's, is well under 1 GB.

**Verify.**

```sh
B=https://github.com/aithos-rr/sea-view-data/releases/download
curl -sLO "$B/data-v2026.11.12/coast-europe.pmtiles"
for r in africa asia australia-oceania central-america north-america south-america antarctica; do
  curl -sLO "$B/data-v2026.11.15/coast-$r.pmtiles"
done
sha256sum -c <<'EOF'
a3088f3a75a0c2a21659ff62fcdf0594046300957d13d715ad4924b4c28dcce0  coast-europe.pmtiles
18ccf43bc50a04f0f30115ea97e1487563bc270fea165eb50a34d43f54e7978a  coast-africa.pmtiles
d43dbcfffee4f29c3b4148ed89bb7fecb93c7d7dc757bfb5fa09eeae373f212e  coast-asia.pmtiles
f6651200191e04ef2a22fb761fd971837ec1a193a3359b0c40cae26546979498  coast-australia-oceania.pmtiles
20f320b5a32736901d88cc71218e238d156f8ca463d0d449ef2f2fd893716dc1  coast-central-america.pmtiles
2f5c026af7cbbdec44bd64f512180c020bd9ec609817e19f957bfe0589c18146  coast-north-america.pmtiles
6c45cd76a2a0f3dd9f82ff0a4d736bc571fea9864f4431c4519372d9f174a0c8  coast-south-america.pmtiles
6719be6e7360263514ac455ec873f969ade333c589e1557db701a0db640e7592  coast-antarctica.pmtiles
EOF
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
