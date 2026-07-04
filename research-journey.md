---
title: Research Journey
permalink: /research-journey/
---
<div class="page-wrap">
<div class="page-content wide" markdown="1">

<div class="page-eyebrow"><span class="dot"></span>Part I · Research Journey</div>

# From an open question to a testable one

<p class="page-dek">Before there was a paper or a platform, there were six weeks of reading, pilot queries, and a comparison table that kept getting longer before it got shorter. This section retraces that process.</p>

## 1.1 &nbsp;Why POI Data Quality Matters for Urban Research {#j-why}

Point-of-Interest data has quietly become load-bearing infrastructure for urban research. Over roughly fifteen years, its role has shifted from a supporting layer to a primary source of evidence — which also means that any quality problem in the underlying data now propagates directly into empirical findings and, downstream, into policy decisions.

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
> Google Places is widely regarded as the most complete and accurate POI source, but its restrictive access policy and per-request cost make it impractical for large-scale academic research. Researchers therefore fall back on alternative datasets — but almost always without knowing, in a systematic way, what they are trading off by doing so. As POI data moves from a supporting variable to the evidentiary backbone of urban studies — and as multi-source conflation becomes common practice — that blind spot becomes a methodological risk rather than a footnote.
{: .callout}

## 1.2 &nbsp;Mapping the Data Source Landscape {#j-landscape}

The first concrete step was not modeling — it was cataloguing. Fourteen POI providers were reviewed for size, update cadence, and access terms, drawing on provider documentation and the [SafeGraph POI data guide](https://www.safegraph.com/guides/points-of-interest-poi-data-guide).

| Resource | Reported Size | Notes | Rating |
|---|---|---|---|
| OpenStreetMap | ~2 TB total storage (no explicit POI count) | Volunteer-maintained, updated weekly, open *Map Features* taxonomy. | ★★★★ |
| Overture Maps | 71.5M Place POIs | Aggregates Foursquare, Microsoft, and others; Foursquare alone claims 200M+, of which Overture ingests ~6M. Also the largest known building-footprint layer (2.5B+). | ★★★★★ |
| Foursquare Open Source Places | 200M+ globally | 100% coverage of the top-100 U.S. retail/QSR chains; ships a `reality_score_mean` confidence field; free/pro/premium tiers. | ★★★★★ |
| SafeGraph | 40M+ across ~200 countries | Monthly refresh cadence; publishes its own coverage map. | ★★★★★ |
| Factual Places API | 130M (52 countries) | Acquired by Foursquare in 2019; no longer available standalone. | ★ |
| Google Places API | 200M+ globally | No native bounding-box search; unsigned requests capped at 25,000/day; the de-facto reference standard. | ★★★★★ |
| Mapbox API | 330M+ globally | Multiple search modes (search-box, address, category). | ★★★★★ |
| TomTom Places API | 100M+ / 180 countries (2020) | 100+ categories split into 600+ subcategories; dedicated EV-charging search; some Overture collaboration. | ★★★ |
| GeoNames | 13M global / 2.2M U.S. | Lightweight, open, sparse relative to commercial sources. | ★★ |
| Who's On First | 21M (219 countries) | Open gazetteer-style POI categories. | ★★ |
| Geocode Earth | — | Aggregates OpenAddresses, OSM, GeoNames, WOF, U.S. Census; has confidence field but no place-type field. | ★★ |
| OpenCorporates | — | Company registry, not a general POI dataset. | ★★ |
| HERE Places | 120M / ~200 countries, 400+ categories (2019) | Mature commercial catalogue. | ★★★★ |
| Precisely Places | 281.3M globally | Rich geometry-based search (travel distance/time, boundary, area). | ★★★★★ |
| Geoapify | — | Travel-type search around a location; 400+ place types. | ★★★ |
{: .table-wrap}

> **Week 1 takeaway**
>
> **OSM**, **Foursquare**, **SafeGraph**, and **Google Places** were the datasets most frequently cited in recent literature. Weighing data size, category coverage, and update frequency, **Mapbox**, **Precisely Places**, and **Overture Maps** were added as further candidates for comparison — a shortlist that would later narrow again on cost and access grounds.
{: .callout}

## 1.3 &nbsp;What Prior Comparison Studies Found {#j-priorwork}

Four existing POI-comparison studies were reviewed in depth to understand what had already been measured, and how.

| Study | Datasets / Region | Key Findings |
|---|---|---|
| Edmisten, 2024 | OSM vs. Overture Maps · global restaurants | Overture returned more restaurants by raw count, but a 100-POI manual spot-check against Google Maps found OSM better aligned to reality (72 vs. 60 confirmed). Filtering to high-confidence records on both sides reversed the ranking — Overture edged ahead. |
| Franzini et al., 2020 | Google Maps vs. OSM · Province of Pavia, Italy | Building-level completeness rose from 42%→65% (OSM) and 28%→91% (GM) between spring and summer 2018 updates. OSM completeness tracked the number and consistency of local contributors. |
| Klinkhardt et al., 2023 | OSM vs. real-world survey · 49 German urban areas | Completeness varied sharply by category and density — ≈73% for shopping, ≈22% for private business. A *saturation index* proved a valid intrinsic indicator for estimating missing POIs without ground truth. |
| Hochmair et al., 2017 | OSM, Foursquare, Google, Yelp, Facebook, Instagram, Twitter · 7 global cities | Social platforms had far higher POI counts but lower positional accuracy and heavy clustering (Nearest-Neighbor Index: Facebook 0.51 vs. OSM 0.73). A Cross-K analysis showed spatial segregation by function — no single source dominated on every axis. |
{: .table-wrap}

Two of the four used *relative* cross-dataset comparison, one ran an on-site survey for an *absolute* comparison, and one treated Google Places as a benchmark. Synthesizing across them surfaced a working list of candidate metrics:

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

## 1.4 &nbsp;From Nine Metrics to Three: How the Question Narrowed {#j-evolution}

The initial research question was broad by design: *how do different POI datasets perform when cross-validated against one another?* A nine-metric wishlist — spatial coverage, completeness, accuracy, consistency, positional accuracy, category structure, density, cost, and update frequency — was feasible to define, but not to compute consistently at national scale within one project timeline. Over the following weeks the scope was deliberately cut down.

1. **Week 1 — nine dimensions, no ground truth strategy**<br>Broad literature-derived metric list; open question of what counts as "ground truth" at a national scale versus a single-city scale.
2. **Week 2 — a two-scale design and a shorter metric list**<br>Introduced a **MSA scale** (Google Places as proxy ground truth, broad coverage) and a **city scale** (municipal open data as ground truth for category-specific validation, e.g., restaurants, schools). The nine metrics were cut to four core ones — **completeness**, **location accuracy**, **category bias**, and **spatial bias** (downtown vs. suburban vs. rural) — with cost and update frequency demoted to a one-paragraph note rather than a computed metric.
3. **Week 3 — a pilot study exposes category-level bias**<br>A small Google Places pilot in Isla Vista (see 1.5) showed that "missingness" itself varies enormously by category — this became the seed of what the final paper calls *category-specific heterogeneity*.
4. **Week 4 — matching becomes the bottleneck, not the metrics**<br>Attempting to actually compute completeness required first solving cross-dataset **entity matching** — the four remaining metrics were only as good as the matching pipeline underneath them. This is the point where methodology work (Levenshtein distance, RapidFuzz, ML-based verification) took over as the main technical focus.
5. **Weeks 5–6 — convergence to the final framework**<br>Category bias and spatial bias were folded together as two expressions of the same underlying phenomenon — a dataset's completeness or accuracy changing across space and category — and reframed as **spatial heterogeneity**. The final paper's three-part framework was set: **completeness**, **positional accuracy**, and **spatial heterogeneity** (measured as a function of both metro population tier and distance to the CBD).
{: .steps}

> **Net effect**
>
> Fewer metrics, computed rigorously and at national scale (14 MSAs, 289,944 reference POIs), proved more useful than nine metrics computed loosely. The two dropped dimensions — cost and update frequency — were not abandoned; they survive in the paper as qualitative context rather than quantitative outputs.
{: .callout}

## 1.5 &nbsp;Methodology Exploration {#j-method-explore}

### A pilot bias study: Google Places in Isla Vista

Before committing to a national design, a small pilot queried the Google Places *Nearby Search* endpoint over a 100 m radius around Isla Vista, CA, across all Table A categories, comparing a general query against category-specific queries.

| Category | Missing when queried generally |
|---|---|
| Facilities | 100% |
| Health | 100% |
| Nature | 100% |
| Business | 66.7% |
| Finance | 50% |
| Government | 33.3% |
| Services | 33.0% |
| Sports | 25% |
| Automotive | 20% |
| Food | 13.2% |
| Places (worship) | 11.1% |
| Transportation | 11% |
| Entertainment | 8.8% |
| Shop | 6.9% |
| Culture / Education / Lodging | 0% |
{: .table-wrap}

This single-region pilot is what first suggested that "how complete is a dataset" is the wrong-shaped question — the right-shaped one is "how complete is a dataset, *for a given category*." That reframing survives into the final paper's category-specific analysis.
{: .note}

### Reviewing the POI-matching literature

Before writing any matching code, four papers on automatic POI matching were reviewed (Piech et al. 2020; Deng et al. 2019; Novack et al. 2018; Almeida et al. 2018), converging on a common shape: a cheap spatial/textual candidate-generation stage followed by a stricter verification stage — the same two-stage structure later implemented in the paper and in POIBench.

### Understanding Levenshtein distance

Name matching across datasets ultimately reduces to string similarity. Levenshtein distance — the minimum number of single-character insertions, deletions, or substitutions needed to turn one string into another — is the base primitive. Turning `kitten` into `sitting` takes exactly three edits:

| Step | Edit | Operation |
|---|---|---|
| 1 | `k` → `s` | substitution |
| 2 | `e` → `i` | substitution |
| 3 | insert `g` at the end | insertion |
{: .table-wrap}

### Choosing a fuzzy-matching library: RapidFuzz

[RapidFuzz](https://github.com/rapidfuzz/RapidFuzz) was selected over FuzzyWuzzy for its speed on large datasets and its family of Levenshtein-derived scorers, each suited to a different failure mode in POI names:

| Function | Extra processing | Best suited for |
|---|---|---|
| `ratio` | none | Basic Levenshtein similarity |
| `partial_ratio` | substring extraction | Partial matches; long vs. short name pairs |
| `token_sort_ratio` | token sorting | Same words, different order |
| `token_set_ratio` | token set operations | One name is a superset/subset of the other |
| `WRatio` | combines all of the above + length scaling | Most robust default; handles noise and reordering |
{: .table-wrap}

```python
fuzz.token_sort_ratio("this is a test", "is this a test")
>>> 100.0

fuzz.WRatio("this is a test", "is this always a TEST???")
>>> 100
```

This exploration directly produced the Stage-1 candidate-matching design used in both the paper and POIBench: name similarity via `rapidfuzz`, cross-checked with `partial_ratio`, `token_sort_ratio`, and `token_set_ratio` — see [Part II](../poi-study/).
{: .note}

</div>
</div>
