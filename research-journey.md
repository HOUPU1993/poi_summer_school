---
title: Research Background
permalink: /research-journey/
---
<div class="page-wrap">
<div class="page-content wide" markdown="1">

<div class="page-eyebrow"><span class="dot"></span>Part I · Research Background</div>

# Why POI Data Quality Matters for Urban Research

<p class="page-dek">15 years ago, most urban studies were survey-based. Between 2010 and 2026, POI data has shifted from a supporting layer to a primary source of evidence — which also means that any quality problem in the underlying data now propagates directly into empirical findings and, downstream, into policy decisions.</p>

## 1.1 &nbsp;How POI Data Has Been Used in Urban Research {#j-why}

The table below summarizes how POI data has been applied across urban research fields over time, adapted from [Psyllidis, A., Gao, S., Hu, Y. et al. (2022), *Points of Interest (POI): a commentary on the state of the art, challenges, and prospects for the future*](https://link.springer.com/article/10.1007/s43762-022-00047-w).

| Period | Research Application |
|---|---|
| ≈2010–2017 | Urban functional zoning & land-use classification |
| ≈2010–2017 | Urban morphology & structure |
| ≈2018–2024 | Urban vibrancy & spatial equity |
| ≈2018–2024 | Socioeconomic inequality & segregation |
| ≈2018–2024 | Transportation & mobility |
| ≈2018–2024 | Multi-source integration |
| ≈2025 → | Dynamic, real-time monitoring |
| ≈2025 → | Semantic & AI-driven understanding |
{: .table-wrap}

## 1.2 &nbsp;Mapping the Data Source Landscape {#j-landscape}

As a preliminary step, fourteen popular POI providers from U.S were reviewed for size, update period, and access terms; the results are listed below.

| Resource | Reported Size | Notes | Rating |
|---|---|---|---|
| [OpenStreetMap](https://wiki.openstreetmap.org/wiki/Overpass_API) | ~2 TB total storage (no explicit POI count) | Volunteer-maintained, updated weekly, open *Map Features* taxonomy. | ★★★★ |
| [Overture Maps](https://overturemaps.org/) | 71.5M Place POIs | Aggregates Foursquare, Microsoft, and others; Foursquare alone claims 200M+, of which Overture ingests ~6M. Also the largest known building-footprint layer (2.5B+). | ★★★★★ |
| [Foursquare Open Source Places](https://places.foursquare.com/) | 200M+ globally | 100% coverage of the top-100 U.S. retail/QSR chains; ships a `reality_score_mean` confidence field; free/pro/premium tiers. | ★★★★★ |
| [SafeGraph](https://www.safegraph.com/guides/places-api) | 40M+ across ~200 countries | Monthly refresh cadence; publishes its own coverage map. | ★★★★★ |
| Factual Places API | 130M (52 countries) | Acquired by Foursquare in 2019; no longer available standalone. | ★ |
| [Google Places API](https://developers.google.com/maps/documentation/places/web-service/overview) | 200M+ globally | No native bounding-box search; unsigned requests capped at 25,000/day; the de-facto reference standard. | ★★★★★ |
| [Mapbox API](https://docs.mapbox.com/api/search/search-box/) | 330M+ globally | Multiple search modes (search-box, address, category). | ★★★★★ |
| [TomTom Places API](https://www.tomtom.com/products/places-apis/) | 100M+ / 180 countries (2020) | 100+ categories split into 600+ subcategories; dedicated EV-charging search; some Overture collaboration. | ★★★ |
| [GeoNames](https://www.geonames.org/) | 13M global / 2.2M U.S. | Lightweight, open, sparse relative to commercial sources. | ★★ |
| [Who's On First](https://whosonfirst.org/) | 21M (219 countries) | Open gazetteer-style POI categories. | ★★ |
| [Geocode Earth](https://geocode.earth/docs/) | — | Aggregates OpenAddresses, OSM, GeoNames, WOF, U.S. Census; has confidence field but no place-type field. | ★★ |
| [OpenCorporates](https://opencorporates.com/) | — | Company registry, not a general POI dataset. | ★★ |
| [HERE Places](https://www.here.com/docs/) | 120M / ~200 countries, 400+ categories (2019) | Mature commercial catalogue. | ★★★★ |
| [Precisely Places](https://developer.precisely.com/apis) | 281.3M globally | Rich geometry-based search (travel distance/time, boundary, area). | ★★★★★ |
| [Geoapify](https://apidocs.geoapify.com/) | — | Travel-type search around a location; 400+ place types. | ★★★ |
{: .table-wrap}

> **Google Places** is widely regarded as the most complete and accurate POI source, but its restrictive access policy and per-request cost make it impractical for large-scale academic research. Researchers therefore turn to alternative datasets — but rarely with a systematic, national-scale understanding of what trade-offs that choice entails.
>
> Further review confirmed that **OSM**, **Foursquare**, **SafeGraph**, and **Overture Maps** were the alternative datasets most frequently cited in recent literature, largely due to their data size, category coverage, update frequency, and ease of access.
{: .callout}

### Estimated Cost Comparison

Based on the [Nearby Search Pro pricing](https://developers.google.com/maps/billing-and-pricing/pricing#places-pricing): each request returns up to 20 POIs, with a maximum of 3 pages per search circle.

| Monthly Requests | Price per 1,000 Requests |
|---|---:|
| 0 – 5,000 | Free |
| 5,001 – 100,000 | $32.00 |
| 100,001 – 500,000 | $25.60 |
| 500,001 – 1,000,000 | $19.20 |
| 1,000,001 – 5,000,000 | $9.60 |
| 5,000,000+ | $2.40 |
{: .table-wrap}

![Estimated Cost Comparison](../assets/poi_study/estimated_cost_comparison.png)
*<span class="fig-label">FIG. 11</span>Estimated Cost Comparison Across Multuple Datasets.*
{: .figcap}


**Note:** **Mapbox** overlaps substantially with the sources above. **Precisely Places**, **TomTom Places API**, and **HERE Places**, despite wide commercial adoption — largely driven by supply-chain and history owners information — appear far less frequently in the academic urban-studies literatures.
{: .note}

![POI Data Sources Relationships](../assets/poi_study/poi_data_sources_relationships.png)
*<span class="fig-label">FIG. 1</span>How the nine POI data sources collect and share data — arrows show which sources feed into which aggregators.*
{: .figcap}

## 1.3 &nbsp;What Prior Comparison Studies Found {#j-priorwork}

Three existing POI-comparison studies were reviewed in depth to understand what had already been measured, and how.

| Study | Datasets / Region | Key Findings |
|---|---|---|
| [Edmisten, 2024](https://surprisedatespot.com/blog/comparing-overture-osm-restaurants/) | OSM vs. Overture Maps · global restaurants | Overture returned more restaurants by raw count, but a 100-POI manual spot-check against Google Maps found OSM better aligned to reality (72 vs. 60 confirmed). Filtering to high-confidence records on both sides reversed the ranking — Overture edged ahead. |
| [Franzini et al., 2020](https://www.scitepress.org/Papers/2020/95643/95643.pdf) | Google Maps vs. OSM · Province of Pavia, Italy | Building-level completeness rose from 42%→65% (OSM) and 28%→91% (GM) between spring and summer 2018 updates. OSM completeness tracked the number and consistency of local contributors. |
| [Klinkhardt et al., 2023](https://journals.sagepub.com/doi/10.1177/03611981231169280) | OSM vs. real-world survey · 49 German urban areas | Completeness varied sharply by category and density — ≈73% for shopping, ≈22% for private business. A *saturation index* proved a valid intrinsic indicator for estimating missing POIs without ground truth. |
{: .table-wrap}

> Two of them used *relative* cross-dataset comparison, one ran an on-site survey for an *absolute* comparison, and one treated Google Places as a benchmark.
{: .callout}

## 1.4 &nbsp;How Many POI Comparison Metrics Can We Get? {#j-metrics}

Synthesizing across the four prior studies above surfaced a working list of nine candidate metrics:

- **01** — **Spatial distribution pattern** — are a dataset's POIs clustered, randomly distributed, or dispersed across the study area?
- **02** — **Completeness** — how much of the real-world POI population does a dataset actually capture?
- **03** — **Accuracy** — are a POI's attributes (existence, name, category) correct?
- **04** — **Consistency** — do different datasets cover the same functional areas of a city, or different ones?
- **05** — **Positional accuracy** — how far off is a POI's recorded coordinate from its true location?
- **06** — **Category structure** — how comparable are datasets' classification hierarchies?
- **07** — **Average density** — how many POIs does a dataset report per km²?
- **08** — **Download cost** — how expensive or restrictive is it to access the data?
- **09** — **Update frequency** — how often is the data refreshed?
{: .card-grid}

This nine-metric wishlist was feasible to *define*, but not to *compute consistently at national scale* within one project timeline — which is what forced the next round of scoping.

## 1.5 &nbsp;Why Four Datasets, and Only Three Dimensions? {#j-narrow}

The final study focuses on **four alternative datasets — Overture, SafeGraph, Foursquare, and OSM — evaluated against Google Places as the reference benchmark**, on **three quality dimensions**: completeness, positional accuracy, and spatial heterogeneity.

## 1.6 &nbsp;How to Download the Data by Coding (Live Demo) {#j-download}

Each data source has a different access pattern. These are the actual queries used to pull sample data during development.

### Google Places API
Requires an API key and billing enabled on Google Cloud. The `searchNearby` endpoint returns up to 20 POIs per request, ranked by popularity by default (distance-ranking is also selectable). See the [Nearby Search documentation](https://developers.google.com/maps/documentation/places/web-service/nearby-search).

```python
import requests

API_KEY = "YOUR_API_KEY"

url = "https://places.googleapis.com/v1/places:searchNearby"
headers = {
    "Content-Type": "application/json",
    "X-Goog-Api-Key": API_KEY,
    "X-Goog-FieldMask": "places.displayName,places.location,places.types"
}
body = {
    "locationRestriction": {
        "circle": {
            "center": {"latitude": 34.4133, "longitude": -119.8610},
            "radius": 500.0
        }
    }
}

response = requests.post(url, headers=headers, json=body)
places = response.json()["places"]
for p in places:
    print(p["displayName"]["text"], p["location"])
```

### Overture Maps
Public Parquet files on Azure Blob Storage / AWS S3 — no API key required. Details [here](https://docs.overturemaps.org/), Example query (Synapse/Azure SQL serverless) for a bounding box around Goleta, CA:, when you finish the configuration in Azure platform, you can query data by the later codes:
```sql
SELECT *
FROM
    OPENROWSET(
        BULK 'https://overturemapswestus2.blob.core.windows.net/release/2026-04-15.0/theme=places/type=place/',
        FORMAT = 'PARQUET'
    )
WITH
    (
        id VARCHAR(MAX),
        names VARCHAR(MAX),
        categories VARCHAR(MAX),
        confidence VARCHAR(MAX),
        addresses VARCHAR(MAX),
        operating_status VARCHAR(MAX),
        [version] VARCHAR(MAX),
        sources VARCHAR(MAX),
        bbox VARCHAR(200),
        geometry VARBINARY(MAX)
    )
    AS
        [result]
WHERE
        TRY_CONVERT(FLOAT, JSON_VALUE(bbox, '$.xmin')) > -89.182473
    AND TRY_CONVERT(FLOAT, JSON_VALUE(bbox, '$.xmax')) < -88.187199
    AND TRY_CONVERT(FLOAT, JSON_VALUE(bbox, '$.ymin')) > 36.771569
    AND TRY_CONVERT(FLOAT, JSON_VALUE(bbox, '$.ymax')) < 37.425322
```

### Foursquare Open Source Places
Distributed via an Iceberg catalog; requires a free access token from [places.foursquare.com](https://places.foursquare.com/).

```python
from pyiceberg.catalog import load_catalog

catalog = load_catalog(
    "default",
    **{
        "warehouse": "places",
        "uri": "https://catalog.h3-hub.foursquare.com/iceberg",
        "token": "YOUR_TOKEN",
        "header.content-type": "application/vnd.api+json",
    },
)

table = catalog.load_table("datasets.places_os")
df = table.scan(limit=20).to_pandas()
print(df[["name", "latitude", "longitude"]])
```

### SafeGraph
Requires applying for a free **academic data license** through SafeGraph directly — not self-serve. Data ships as Parquet/CSV via a private S3 bucket once approved. Details [here](https://www.safegraph.com/products/places/)

### OpenStreetMap
Query from the [Pyrosm](https://pyrosm.readthedocs.io/en/stable/) directly.

```python
import osmnx as ox

ox.settings.cache_folder = "/Users/houpuli/Downloads/osm_cache"
# Query by city
tags = {"amenity": True, "shop": True}
gdf = ox.features_from_place("Santa Barbara, California, USA", tags)
```

</div>
</div>