---
title: A National-Level POI Quality Assessment Across 4 Data Sources and 15 Categories
permalink: /poi-study/
---
<div class="page-wrap">
<div class="page-content wide" markdown="1">

<div class="page-eyebrow"><span class="dot"></span>Part II · The Study</div>

# A Multi-Dimensional Quality Assessment of Popular Alternative POI Datasets

<p class="page-dek">Evidence from 14 U.S. Metropolitan Statistical Areas. Houpu Li, 2026 — prepared for <em>Transactions in GIS</em>.</p>

## 2.1 &nbsp;Introduction & Study Design {#s-intro}

Urban studies increasingly rely on POI data to characterize the spatial distribution of urban functions and amenities. Google Places is widely regarded as the most comprehensive and accurate source, but its API costs and access limits restrict its use in large-scale research — pushing researchers toward Overture Maps, SafeGraph, Foursquare, and OpenStreetMap. Because these four datasets differ substantially in how they are collected, they may carry systematic quality differences that bias empirical findings. Despite growing reliance on them, a nationwide, multi-dimensional quality assessment had not existed. This study provides one.

- **14** — U.S. Metropolitan Statistical Areas, across 3 population tiers
- **289,944** — unique Google Places reference POIs
- **15** — Google Places Table A categories evaluated
- **84,000** — manually validated matched pairs used to train verification classifiers
{: .card-grid}

### Study areas

| Tier | MSA | Population | Google Places POIs |
|---|---|---:|---:|
| High | New York–Newark–Jersey City, NY-NJ | 20,011,812 | 65,680 |
| High | Los Angeles–Long Beach–Anaheim, CA | 13,202,558 | 39,829 |
| High | Chicago–Naperville–Elgin, IL-IN | 9,607,711 | 56,961 |
| High | Dallas–Fort Worth–Arlington, TX | 7,543,340 | 55,860 |
| High | Houston–Pasadena–The Woodlands, TX | 7,048,954 | 50,599 |
| Mid | Bremerton–Silverdale–Port Orchard, WA | 273,072 | 4,261 |
| Mid | Sioux Falls, SD-MN | 272,414 | 7,012 |
| Mid | Santa Cruz–Watsonville, CA | 272,138 | 4,387 |
| Mid | Erie, PA | 271,903 | 4,990 |
| Mid | Norwich–New London–Willimantic, CT | 269,131 | 6,825 |
| Low | Hot Springs, AR | 99,694 | 2,413 |
| Low | Dubuque, IA | 98,687 | 2,584 |
| Low | Pinehurst–Southern Pines, NC | 98,618 | 2,839 |
| Low | Paducah, KY-IL | 98,477 | 4,226 |
{: .table-wrap}

Reference POIs were built from a grid of sampling points spaced 5 km apart across each MSA; each point queried the Google Places API separately for each of 15 categories within a 5 km radius (max 20 POIs per query). *Facilities* and *Housing* were excluded — the former for having too few subtypes to compare meaningfully, the latter because most alternative datasets lack a comparable category.
{: .note}

## 2.2 &nbsp;Why Standardization Comes First {#s-standardization}

The most consequential methodological fact underlying this entire study is easy to state and easy to underestimate: **the four alternative datasets don't just disagree about which POIs exist — they disagree about how to describe the POIs they agree on.** Before any distance threshold or similarity score means anything, every record has to be forced into the same shape.

This is not a hypothetical concern. The same real-world business shows up differently across sources because each provider has its own internal conventions for fields, encodings, and formatting:

| Source | Recorded name |
|---|---|
| Google Places | `7-Eleven` |
| Provider B | `7-ELEVEN #24817` |
| Provider C | `Seven Eleven` |
| Provider D | `7‑Eleven® (Café)` |
{: .table-wrap}

*↑ same store, four different strings — a naive string comparison sees four different POIs ↑*
{: .figcap}

Four categories of inconsistency recur across every dataset pair:

- **Aa** — **Case conventions** — all-caps store IDs vs. title case vs. mixed brand styling
- **é→e** — **Character encoding** — accented characters, unicode symbols, or smart quotes rendered inconsistently
- **St.** — **Address formatting** — abbreviated vs. spelled-out street types, unit numbers present or absent
- **#12** — **Field noise** — franchise/store numbers, trademark symbols, and punctuation baked into the name field
{: .card-grid}

> **The consequence**
>
> Without correcting for these conventions first, every downstream step degrades silently. A name-similarity score computed on `7-ELEVEN #24817` vs. `7-Eleven` will under-report similarity for reasons that have nothing to do with whether it's the same business — inflating false negatives and making a dataset look less complete than it is. This is exactly why standardization is not an optional cleanup step; it is the precondition for every completeness and accuracy number this study reports.
{: .callout}

### The standardization pipeline

All records — from Google Places and each of the four comparison datasets — pass through an identical four-step pipeline before matching begins:

1. **Field extraction** — Every record is reduced to five core fields regardless of source schema: a unique ID, name, address, category, and geographic coordinates (lat/lon).
2. **Special-character removal** — Punctuation, trademark symbols, and franchise/store numbers embedded in name fields are stripped.
3. **Unicode normalization** — Accented and non-ASCII characters are normalized to a consistent representation so that visually identical names compare equal.
4. **Uppercase conversion** — All text fields are case-folded, removing spurious mismatches from stylistic capitalization choices.
{: .steps}

Category fields are separately cross-walked to the Google Places Table A classification, since each alternative dataset ships its own category taxonomy.
{: .note}

## 2.3 &nbsp;The Two-Stage Matching Pipeline {#s-matching}

Once records are standardized, matching balances two competing failure modes: missing true matches (hurting recall) and accepting false ones (hurting precision). The pipeline splits this into a cheap, generous first pass and a stricter, learned second pass.

1. **Stage 1 — Rule-based candidate matching**<br>For each Google Places POI, a KD-tree spatial index finds up to 100 nearby candidates within a 1,000 m radius in the comparison dataset. Name similarity is scored with RapidFuzz's Levenshtein-based fuzzy matching, keeping the top 5 candidates scoring above 80. Consistency is cross-checked with `partial_ratio`, `token_sort_ratio`, and `token_set_ratio`; the highest-scoring candidate is selected as the matched pair. Address similarity and location distance are computed for every matched pair and passed forward.
2. **Stage 2 — ML-based match verification**<br>Manual review of Stage-1 output found a ≈10.3% mismatch rate, varying noticeably by MSA and dataset — evidence that naming conventions, address formats, and spatial density differ enough between metros that a single rule-based threshold can't fit all of them. To correct this, a **separate XGBoost classifier is trained for every MSA × dataset combination** (56 classifiers total), using three features: name similarity, location distance, and address similarity.
{: .steps}

- **84,000** — manually labeled candidate pairs, 75/25 train-test split per MSA-dataset combination
- **0.97** — median precision across 56 classifiers
- **0.98** — median recall & F1-score
- **0.96** — median accuracy
{: .card-grid}

> **Why 56 separate classifiers, not one**
>
> A single pooled model trained across all MSAs and datasets would average away real differences — a name-matching pattern that works for SafeGraph in New York doesn't necessarily transfer to OSM in Dubuque. Training per MSA-dataset combination lets the classifier absorb local naming conventions, address formats, and spatial density rather than fight against them.
{: .callout}

## 2.4 &nbsp;The Quality Assessment Framework {#s-framework}

Every alternative dataset is scored on three dimensions, each computed at three levels: pooled across all 14 MSAs, aggregated by population tier, and as a function of distance to the CBD.

- **①** — **Completeness** — the share of Google Places POIs in category *c* that have a verified true match in dataset *d*: `|TP(d,c)| / |GP(c)|`
- **②** — **Positional accuracy** — the median Euclidean distance, in meters, between matched POI pairs (median chosen over mean to resist long-tail outliers from bad source coordinates)
- **③** — **Spatial heterogeneity** — a weighted-least-squares regression of completeness or location bias against distance to the city center, weighted by the number of reference POIs per 2 km bin
{: .card-grid}

## 2.5 &nbsp;Results {#s-results}

### Overall completeness

Pooled across all 14 MSAs, *SafeGraph*{: .badge-safegraph} achieves the highest true match rate at 71.8%, followed by *Overture*{: .badge-overture} (69.5%), *Foursquare*{: .badge-foursquare} (64.4%), and *OSM*{: .badge-osm} at roughly half the others (31.8%) — consistent with OSM's volunteer-driven collection model. False-positive rates stay low (8.2–13.3%) across the board, indicating high-quality candidate generation at Stage 1.

![Overall completeness bar chart by dataset](../assets/poi_study/study_fig1_overall_completeness.png)
*<span class="fig-label">FIG. 1</span>Overall completeness (true match rate) and false-positive rate, pooled across 14 MSAs.*
{: .figcap}

### Completeness declines with metropolitan scale

<div class="fig-grid" markdown="1">

<div markdown="1">

![Completeness by population tier](../assets/poi_study/study_fig2_completeness_by_tier.png)
*<span class="fig-label">FIG. 2</span>Completeness by MSA population tier.*
{: .figcap}

</div>
<div markdown="1">

![Completeness vs distance to CBD](../assets/poi_study/study_fig3_completeness_vs_distance.png)
*<span class="fig-label">FIG. 3</span>Completeness vs. distance to the CBD, by tier (WLS fit).*
{: .figcap}

</div>

</div>

| Dataset | High-Tier | Mid-Tier | Low-Tier | Relative change |
|---|---:|---:|---:|---:|
| SafeGraph | 72.7% | 64.8% | 63.4% | −12.8% |
| Overture | 70.7% | 61.8% | 58.4% | −17.4% |
| Foursquare | 65.4% | 58.6% | 53.4% | −18.3% |
| OSM | 32.0% | 33.4% | 21.4% | −33.1% |
{: .table-wrap}

True match rates fall for every dataset as metropolitan size decreases and as distance from the CBD increases. OSM's relative decline (−33.1%) is roughly twice that of Overture or Foursquare — its completeness is the most sensitive to both metro scale and peripheral location. Overture is notably the *only* dataset to retain a statistically significant distance-decay gradient across all three population tiers (table below).

| Tier | Dataset | β (pp / 10 km) | R² | Sig. |
|---|---|---:|---:|---|
| High | Overture | −0.69 | 0.76 | *** |
| High | SafeGraph | −0.40 | 0.56 | *** |
| High | Foursquare | −0.99 | 0.80 | *** |
| High | OSM | −1.33 | 0.77 | *** |
| Mid | Overture | −0.71 | 0.23 | ** |
| Mid | SafeGraph | −0.25 | 0.05 | n.s. |
| Mid | Foursquare | −1.61 | 0.42 | *** |
| Mid | OSM | −2.14 | 0.33 | *** |
| Low | Overture | −1.02 | 0.28 | *** |
| Low | SafeGraph | −0.51 | 0.08 | n.s. |
| Low | Foursquare | −0.57 | 0.06 | n.s. |
| Low | OSM | −1.96 | 0.11 | n.s. |
{: .table-wrap}

### Positional accuracy

*SafeGraph*{: .badge-safegraph} has both the best median error (4.8 m) and the narrowest spread. *Overture*{: .badge-overture} (12.9 m) and *OSM*{: .badge-osm} (12.7 m) post nearly identical medians, but Overture's distribution is far more compact. *Foursquare*{: .badge-foursquare} has the worst median (16.7 m) and the widest spread, with its upper whisker exceeding 100 m.

<div class="fig-grid" markdown="1">

<div markdown="1">

![Overall location accuracy box plot](../assets/poi_study/study_fig4_overall_location_accuracy.png)
*<span class="fig-label">FIG. 4</span>Overall location bias by dataset (pooled).*
{: .figcap}

</div>
<div markdown="1">

![Location accuracy by tier](../assets/poi_study/study_fig5_location_accuracy_by_tier.png)
*<span class="fig-label">FIG. 5</span>Location bias by MSA population tier.*
{: .figcap}

</div>

</div>

![Location accuracy vs distance to CBD](../assets/poi_study/study_fig6_location_accuracy_vs_distance.png)
*<span class="fig-label">FIG. 6</span>Location bias vs. distance to the CBD, by tier (WLS fit).*
{: .figcap}

> **The key asymmetry**
>
> Completeness is highly sensitive to spatial context — it declines systematically with both smaller metro size and greater distance from the CBD, for every dataset. Positional accuracy is not: Overture and SafeGraph show **no consistent spatial gradient** at any population tier, while only Foursquare and OSM degrade meaningfully with distance. This suggests completeness is governed by spatial coverage effort, while positional accuracy is governed by source-level collection practice.
{: .callout}

### Category-specific patterns

Aggregating across the three tiers reveals three completeness bands. **Food, Worship, Education, and Shop** are consistently well covered (Overture/SafeGraph ≈80%, low cross-tier variance). A middle band — Finance, Automotive, Health, Government, Sport, Services, Culture, Entertainment, Business — ranges 50–80%. **Transportation and Nature** form a low-completeness band; Nature sits below 10% for every dataset. OSM trails across almost every category, most severely in Services (15.0%) and Business (6.3%).

![Match rate by category, high-population MSAs](../assets/poi_study/study_fig7a_category_completeness_high.png)
*<span class="fig-label">FIG. 7a</span>Match rate by category and dataset — high-population MSAs.*
{: .figcap}

![Match rate by category, mid-population MSAs](../assets/poi_study/study_fig7b_category_completeness_mid.png)
*<span class="fig-label">FIG. 7b</span>Match rate by category and dataset — mid-population MSAs.*
{: .figcap}

![Match rate by category, low-population MSAs](../assets/poi_study/study_fig7c_category_completeness_low.png)
*<span class="fig-label">FIG. 7c</span>Match rate by category and dataset — low-population MSAs.*
{: .figcap}

| Category | Overture | SafeGraph | Foursquare | OSM |
|---|---:|---:|---:|---:|
| Food | 88.6 | 87.7 | 85.6 | 47.5 |
| Worship | 83.7 | 85.4 | 72.6 | 46.2 |
| Education | 80.9 | 84.5 | 73.8 | 50.4 |
| Shop | 79.3 | 79.3 | 72.9 | 39.4 |
| Finance | 72.6 | 75.9 | 69.9 | 28.2 |
| Automotive | 71.9 | 76.2 | 69.0 | 32.1 |
| Health | 71.6 | 78.0 | 64.2 | 23.8 |
| Government | 66.8 | 82.9 | 65.4 | 32.7 |
| Sport | 69.0 | 66.4 | 56.8 | 27.0 |
| Services | 64.5 | 72.5 | 57.5 | 15.0 |
| Culture | 65.2 | 70.6 | 59.7 | 36.0 |
| Entertainment | 60.9 | 54.8 | 55.3 | 37.4 |
| Business | 54.5 | 63.6 | 50.5 | 6.3 |
| Transportation | 34.8 | 54.3 | 38.9 | 24.1 |
| Nature | 9.0 | 7.4 | 9.2 | 3.7 |
{: .table-wrap}

*Mean true match rate (%) by category, averaged across tiers*
{: .figcap}

![Match rate vs distance to CBD, by category, all datasets](../assets/poi_study/study_fig8_category_distance_grid.png)
*<span class="fig-label">FIG. 8</span>Completeness vs. distance to the CBD, all 15 categories pooled across MSAs (WLS estimates).*
{: .figcap}

Distance-decay in completeness is steepest for **Transportation, Entertainment, Finance, Sport, Nature, and Culture**, moderate for Government/Health/Education, and flattest for Shop/Business/Services/Worship/Food/Automotive. OSM consistently shows the steepest within-group decay (e.g. Sport: −2.87 pp/10 km), except for Transportation, where Foursquare and Overture decay faster than OSM.

![Location bias by category, high-population MSAs](../assets/poi_study/study_fig9a_category_locbias_high.png)
*<span class="fig-label">FIG. 9a</span>Location bias by category and dataset — high-population MSAs.*
{: .figcap}

![Location bias by category, mid-population MSAs](../assets/poi_study/study_fig9b_category_locbias_mid.png)
*<span class="fig-label">FIG. 9b</span>Location bias by category and dataset — mid-population MSAs.*
{: .figcap}

![Location bias by category, low-population MSAs](../assets/poi_study/study_fig9c_category_locbias_low.png)
*<span class="fig-label">FIG. 9c</span>Location bias by category and dataset — low-population MSAs.*
{: .figcap}

| Category | Overture | SafeGraph | Foursquare | OSM |
|---|---:|---:|---:|---:|
| Food | 11.0 | 5.1 | 10.9 | 6.7 |
| Culture | 13.0 | 4.2 | 18.7 | 15.2 |
| Services | 15.3 | 5.1 | 18.5 | 17.5 |
| Shop | 15.6 | 5.9 | 16.4 | 12.8 |
| Automotive | 15.9 | 6.8 | 16.3 | 9.0 |
| Health | 16.4 | 5.0 | 14.4 | 13.4 |
| Worship | 16.5 | 3.6 | 18.1 | 17.2 |
| Government | 16.8 | 3.5 | 17.6 | 15.8 |
| Finance | 16.9 | 4.7 | 11.2 | 6.7 |
| Education | 17.3 | 3.4 | 30.8 | 53.5 |
| Entertainment | 14.2 | 13.0 | 59.3 | 52.1 |
| Transportation | 38.3 | 7.2 | 61.8 | 21.4 |
| Nature | 168.1 | 274.5 | 342.8 | 264.3 |
{: .table-wrap}

*Mean median location bias (m) by category*
{: .figcap}

![Location bias vs distance to CBD by category](../assets/poi_study/study_fig10_category_locbias_distance_grid.png)
*<span class="fig-label">FIG. 10</span>Location bias vs. distance to the CBD, all 15 categories pooled (WLS estimates).*
{: .figcap}

## 2.6 &nbsp;Discussion & Implications {#s-discussion}

- **①** — **SafeGraph & Overture lead**, but by different margins depending on context. Foursquare and OSM show distinct failure modes: Foursquare trades completeness for noisy coordinates; OSM trades positional accuracy for large completeness gaps.
- **②** — **Completeness has a spatial signature that accuracy doesn't.** Match rates decay with metro size and CBD distance across the board; location accuracy is largely flat except for Foursquare and OSM.
- **③** — **OSM's weakness is category-dependent, not uniform** — steep decay for Sport/Culture/Nature, but comparatively mild for Transportation.
- **④** — **No dataset dominates every category.** SafeGraph leads Government/Services/Business/Transportation; the three commercial datasets converge on Food/Worship/Education/Shop.
{: .card-grid}

Practically, this means dataset choice should be **category-specific rather than global**. For coordinate-sensitive work (accessibility, routing, proximity measures), SafeGraph's low error and narrow spread make it the safer default. For broad spatial coverage across metro sizes, SafeGraph and Overture are most reliable, though both still lose ground in smaller and peripheral areas. OSM warrants particular caution in low-population MSAs and for leisure, cultural, and natural-amenity categories. Because completeness — not positional accuracy — carries the systematic urban-core/periphery bias, studies spanning that gradient should correct for coverage rather than coordinate quality.


</div>

</div>
