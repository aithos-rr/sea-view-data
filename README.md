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
| `ports` | `ports.sqlite` | fused port gazetteer | ODbL 1.0 (derivative database) | no release yet |
| `coast-<region>` | `coast-<region>.pmtiles` | coastal vector tiles | ODbL 1.0 | no release yet |
| `basemap-<region>` | `basemap-<region>.pmtiles` | fallback basemap tiles | ODbL 1.0 | no release yet |

The table is filled in as the artefacts are published; every row is "no release
yet" until then.

---

## `ports` — the fused port gazetteer

**Status: no release yet.**

A single SQLite database of ports, marinas, harbours and their berthing
structures, fused from four sources with a deterministic rule set, used by Sea
View to name the place a yacht is moored or anchored at.

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

  Source: <https://msi.nga.mil/commercial-use>.
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

**Status: no release yet.**

PMTiles archives of the marine detail a general-purpose basemap does not carry:
breakwaters, piers, quays, pontoons, moorings, harbour and marina polygons,
seamarks. One archive per region.

**Origin and licence.** Built from OpenStreetMap data via Geofabrik extracts,
and published under **ODbL 1.0** ([`LICENSES/ODbL-1.0.txt`](LICENSES/ODbL-1.0.txt)).

> Contains information from OpenStreetMap, which is made available under the
> Open Database License (ODbL).

Attribution to display when these tiles are rendered: **© OpenStreetMap
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
