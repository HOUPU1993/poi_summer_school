---
title: Research Background
permalink: /research-journey/
---
<div class="page-wrap">
<div class="page-content wide" markdown="1">

<div class="page-eyebrow"><span class="dot"></span>Part I · Research Background</div>

# Why POI Data Quality Matters for Urban Research

<p class="page-dek">15 years ago, most urban studies were survey-based. Between 2010 and 2026, POI data has shifted from a supporting layer to a primary source of evidence — which also means that any quality problem in the underlying data now propagates directly into empirical findings and, downstream, into policy decisions.</p>

## 1.1 &nbsp;Why POI Data Quality Matters for Urban Research {#j-why}

The table below summarizes how POI data has been applied across urban research fields over time, adapted from [Andris et al. (2022), *Points of Interest (POI): a commentary on the state of the art, challenges, and prospects for the future*](https://link.springer.com/article/10.1007/s43762-022-00047-w).

| Period | Research Application | What Changed |
|---|---|---|
| ≈2010–2017 | Urban functional zoning & land-use classification | POIs used mainly as *auxiliary* spatial data — density, category ratios, and nearest-neighbor indicators integrated with remote sensing or census data to detect functional zones and validate land-use maps. |
| ≈2010–2017 | Urban morphology & structure | POI distribution linked to street-network metrics (connectivity, centrality) to study compactness, mixed use, and spatial hierarchy. |
| ≈2018–2024 | Urban vibrancy & spatial equity | POIs combined with mobile-phone, social-media, and nighttime-light data to quantify vitality, accessibility, and livability via entropy and diversity indices. |
| ≈2018–2024 | Socioeconomic inequality & segregation | POIs used to detect disparities in access to retail, food, and education — food deserts, opportunity gaps, racialized service distributions. |
| ≈2018–2024 | Transportation & mobility | POIs as destination proxies in trip-generation and demand models; accessibility via clustering and travel-time buffers. |
| ≈2018–2024 | Multi-source integration | Fusion of heterogeneous POI datasets (Google, Overture, SafeGraph, OSM) to improve completeness, positional accuracy, and semantic consistency — **the direct predecessor of this study.** |
| ≈2025 → | Dynamic, real-time monitoring | High-frequency POI updates for near-real-time business turnover and post-pandemic recovery analysis. |
| ≈2025 → | Semantic & AI-driven understanding | Text semantics, embeddings, and LLM-based classification to capture urban meaning and function. |
{: .table-wrap}

> **Why this matters**
>
> Google Places is widely regarded as the most complete and accurate POI source, but its restrictive access policy and per-request cost make it impractical for large-scale academic research. Researchers therefore fall back on alternative datasets — but almost always without knowing, in a systematic way, what they are trading off by doing so. As POI data moves from a supporting variable to the evidentiary backbone of urban studies, that blind spot becomes a methodological risk rather than a footnote.
{: .callout}

## 1.2 &nbsp;Mapping the Data Source Landscape {#j-landscape}

The first concrete step was not modeling — it was cataloguing. Fourteen POI providers were reviewed for size, update cadence, and access terms.

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

> **Week 1 takeaway**
>
> **OSM**, **Foursquare**, **SafeGraph**, and **Google Places** were the datasets most frequently cited in recent literature. Weighing data size, category coverage, and update frequency, **Mapbox**, **Precisely Places**, and **Overture Maps** were added as further candidates for comparison — a shortlist that would later narrow again on cost and access grounds.
{: .callout}

## 1.3 &nbsp;What Prior Comparison Studies Found {#j-priorwork}

Four existing POI-comparison studies were reviewed in depth to understand what had already been measured, and how.

| Study | Datasets / Region | Key Findings |
|---|---|---|
| [Edmisten, 2024](https://surprisedatespot.com/blog/comparing-overture-osm-restaurants/) | OSM vs. Overture Maps · global restaurants | Overture returned more restaurants by raw count, but a 100-POI manual spot-check against Google Maps found OSM better aligned to reality (72 vs. 60 confirmed). Filtering to high-confidence records on both sides reversed the ranking — Overture edged ahead. |
| [Franzini et al., 2020](https://www.scitepress.org/Papers/2020/95643/95643.pdf) | Google Maps vs. OSM · Province of Pavia, Italy | Building-level completeness rose from 42%→65% (OSM) and 28%→91% (GM) between spring and summer 2018 updates. OSM completeness tracked the number and consistency of local contributors. |
| [Klinkhardt et al., 2023](https://journals.sagepub.com/doi/10.1177/03611981231169280) | OSM vs. real-world survey · 49 German urban areas | Completeness varied sharply by category and density — ≈73% for shopping, ≈22% for private business. A *saturation index* proved a valid intrinsic indicator for estimating missing POIs without ground truth. |
| [Hochmair et al., 2017](https://link.springer.com/chapter/10.1007/978-3-319-71470-7_15) | OSM, Foursquare, Google, Yelp, Facebook, Instagram, Twitter · 7 global cities | Social platforms had far higher POI counts but lower positional accuracy and heavy clustering (Nearest-Neighbor Index: Facebook 0.51 vs. OSM 0.73). A Cross-K analysis showed spatial segregation by function — no single source dominated on every axis. |
{: .table-wrap}

Two of the four used *relative* cross-dataset comparison, one ran an on-site survey for an *absolute* comparison, and one treated Google Places as a benchmark.

## 1.4 &nbsp;How Many POI Comparison Metrics Can We Get? {#j-metrics}

Synthesizing across the four prior studies above surfaced a working list of nine candidate metrics:

- **01** — **Spatial coverage & distribution** — Nearest-Neighbor Index for clustered / random / dispersed patterns
- **02** — **Completeness** — share of real-world POIs present; saturation index for growth-curve maturity
- **03** — **Accuracy** — via confidence fields (Overture), reality scores (Foursquare), or Google Places as proxy ground truth
- **04** — **Consistency** — Cross-K function; do sources cover the same functional areas?
- **05** — **Positional accuracy** — coordinate error vs. ground truth, in meters
- **06** — **Category structure** — clarity and comparability of classification hierarchies
- **07** — **Average density** — POIs per km²
- **08** — **Download cost** — API limits, pricing, retrieval time
- **09** — **Update frequency** — refresh interval and temporal validity
{: .card-grid}

This nine-metric wishlist was feasible to *define*, but not to *compute consistently at national scale* within one project timeline — which is what forced the next round of scoping.

## 1.5 &nbsp;Why Four Datasets, and Only Three Dimensions? {#j-narrow}

The final study focuses on **four alternative datasets — Overture, SafeGraph, Foursquare, and OSM — evaluated against Google Places as the reference benchmark**, on **three quality dimensions**: completeness, positional accuracy, and spatial heterogeneity. Getting there took six weeks of deliberately cutting scope.

1. **Week 1 — nine dimensions, no ground truth strategy**<br>Broad literature-derived metric list; open question of what counts as "ground truth" at a national scale versus a single-city scale.
2. **Week 2 — a two-scale design and a shorter metric list**<br>Introduced a **MSA scale** (Google Places as proxy ground truth, broad coverage) and a **city scale** (municipal open data as ground truth for category-specific validation, e.g., restaurants, schools). The nine metrics were cut to four core ones — **completeness**, **location accuracy**, **category bias**, and **spatial bias** (downtown vs. suburban vs. rural) — with cost and update frequency demoted to a one-paragraph note rather than a computed metric.
3. **Week 3 — a pilot study exposes category-level bias**<br>A small Google Places pilot in Isla Vista (see 1.7) showed that "missingness" itself varies enormously by category — this became the seed of what the final paper calls *category-specific heterogeneity*.
4. **Week 4 — matching becomes the bottleneck, not the metrics**<br>Attempting to actually compute completeness required first solving cross-dataset **entity matching** — the four remaining metrics were only as good as the matching pipeline underneath them. This is the point where methodology work (Levenshtein distance, RapidFuzz, ML-based verification) took over as the main technical focus.
5. **Weeks 5–6 — convergence to the final framework**<br>Category bias and spatial bias were folded together as two expressions of the same underlying phenomenon — a dataset's completeness or accuracy changing across space and category — and reframed as **spatial heterogeneity**. The final framework was set: **completeness**, **positional accuracy**, and **spatial heterogeneity** (measured as a function of both metro population tier and distance to the CBD).
{: .steps}

**Why these four datasets specifically?** Overture, SafeGraph, Foursquare, and OSM were the most consistently available, most frequently cited in prior literature (Section 1.3), and — critically — free or academically licensed, unlike Mapbox, HERE, TomTom, or Precisely. Google Places was kept as the *reference*, not a fifth candidate, because it is the benchmark everyone else is measured against, not an alternative to it.
{: .note}

> **Net effect**
>
> Fewer metrics, computed rigorously and at national scale (14 MSAs, 289,944 reference POIs), proved more useful than nine metrics computed loosely. The two dropped dimensions — cost and update frequency — were not abandoned; they survive as qualitative context in Part II rather than as quantitative outputs.
{: .callout}

## 1.6 &nbsp;How to Download the Data by Coding (Live Demo) {#j-download}

Each data source has a different access pattern. These are the actual queries used to pull sample data during development — useful as a starting point to demo live in class.

### Google Places API
Requires an API key and billing enabled on Google Cloud. The `searchNearby` endpoint returns up to 20 POIs per request, ranked by popularity by default (distance-ranking is also selectable). See the [Nearby Search documentation](https://developers.google.com/maps/documentation/places/web-service/nearby-search).

### Overture Maps
Public Parquet files on Azure Blob Storage / AWS S3 — no API key required. Example query (Synapse/Azure SQL serverless) for a bounding box around Goleta, CA:

```sql
SELECT *
FROM
    OPENROWSET(
        BULK 'https://overturemapswestus2.blob.core.windows.net/release/2025-10-22.0/theme=places/type=place/',
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
    AS [result]
WHERE
        TRY_CONVERT(FLOAT, JSON_VALUE(bbox, '$.xmin')) > -119.86940
    AND TRY_CONVERT(FLOAT, JSON_VALUE(bbox, '$.xmax')) < -119.85346
    AND TRY_CONVERT(FLOAT, JSON_VALUE(bbox, '$.ymin')) > 34.40887
    AND TRY_CONVERT(FLOAT, JSON_VALUE(bbox, '$.ymax')) < 34.41727
```

### Foursquare Open Source Places
Distributed via an Iceberg catalog; requires a free access token from [places.foursquare.com](https://places.foursquare.com/).

```python
from pyiceberg.catalog import load_catalog
from pyiceberg.expressions import And, GreaterThanOrEqual, LessThanOrEqual

catalog = load_catalog(
    "default",
    **{
        "warehouse": "places",
        "uri": "https://catalog.h3-hub.foursquare.com/iceberg",
        "token": token,
        "header.content-type": "application/vnd.api+json",
        "rest-metrics-reporting-enabled": "false",
    },
)

minLon, maxLon = -119.86940, -119.85346
minLat, maxLat =  34.40887,  34.41727

table = catalog.load_table('datasets.places_os')

expr = And(
    And(GreaterThanOrEqual("longitude", minLon), LessThanOrEqual("longitude", maxLon)),
    And(GreaterThanOrEqual("latitude",  minLat), LessThanOrEqual("latitude",  maxLat)),
)

places_os = table.scan(row_filter=expr, limit=5000).to_pandas()
places_os
```

### SafeGraph
Requires applying for a free **academic data license** through SafeGraph directly — not self-serve. Data ships as Parquet/CSV via a private S3 bucket once approved.

### OpenStreetMap
Free, no key required. Small areas: query the [Overpass API](https://wiki.openstreetmap.org/wiki/Overpass_API) directly. Large areas: download regional extracts from [Geofabrik](https://download.geofabrik.de/).
{: .note}

</div>
</div>