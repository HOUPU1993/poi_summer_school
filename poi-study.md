---
title: A National-Level POI Quality Assessment Across 4 Data Sources and 15 Categories
permalink: /poi-study/
---
<div class="page-wrap">
<div class="page-content wide" markdown="1">

<div class="page-eyebrow"><span class="dot"></span>Part II · The Study</div>

# A Multi-Dimensional Quality Assessment of Popular Alternative POI Datasets

<p class="page-dek">Evidence from 14 U.S. Metropolitan Statistical Areas. Houpu Li, 2026 — prepared for <em>Transactions in GIS</em>.</p>

## 1.1 &nbsp;Study Area and Data Collection {#s-intro}

This study evaluates four alternative POI datasets — Overture Maps, SafeGraph, Foursquare, and OpenStreetMap — against Google Places as the reference benchmark, across **14 U.S. Metropolitan Statistical Areas** spanning three population tiers (high, mid, and low). Google Places POIs were collected as ground truth via a systematic grid-sampling approach, then matched against each alternative dataset to compute completeness, positional accuracy, and spatial heterogeneity across 15 categories.

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

<iframe 
  src="../assets/html/msa_centers_satellite.html" 
  style="width:100%;height:600px;border:1px solid var(--line);border-radius:3px;margin:20px 0;" 
  loading="lazy">
</iframe>

Reference POIs were built from a grid of sampling points spaced 5 km apart across each MSA; each point queried the Google Places API separately for each of 15 categories within a 5 km radius (max 20 POIs per query). *Facilities* and *Housing* were excluded — the former for having too few subtypes to compare meaningfully, the latter because most alternative datasets lack a comparable category.
{: .note}

### Why Query by Category, Not Generally

The Google Places API returns results for a search circle ranked by popularity by default (distance-ranking is also selectable), capped at a small number of results per request. A pilot study in Isla Vista, CA (bounding box: lon −119.8694 to −119.85346, lat 34.40887 to 34.41727; 100 m radius) compared a single **general query** (no category filter) against **category-specific queries** (one separate query per category) to test whether a general query alone is sufficient.

![Completeness by population tier](../assets/poi_study/total_query_cat.png)
*<span class="fig-label">FIG. 2</span>Comparison Between Total Query and Category-specific Query.*
{: .figcap}

| Category | General Query (Total) | Category-Specific Query | Missing Rate |
|---|---:|---:|---:|
| Automotive | 8 | 10 | 20% |
| Business | 1 | 3 | 66.67% |
| Culture | N/A | 2 | 0% |
| Education | N/A | 5 | 0% |
| Entertainment | 31 | 34 | 8.82% |
| Facilities | N/A | 1 | 100% |
| Finance | 3 | 6 | 50% |
| Food | 46 | 53 | 13.21% |
| Government | 2 | 3 | 33.3% |
| Health | N/A | 11 | 100% |
| Lodging | 2 | 2 | 0% |
| Nature | 2 | 1 | 100% |
| Places of Worship | 8 | 9 | 11.1% |
| Services | 20 | 29 | 33.03% |
| Shop | 27 | 29 | 6.9% |
| Sports | 3 | 4 | 25% |
| Transportation | 24 | 27 | 11% |
{: .table-wrap}

The highest bias arises in **Facilities, Health, and Nature** (100% missing under a general query alone), followed by **Business** (66.67%) and **Finance** (50%). Only high-density, frequently-searched categories (Food, Shop, Entertainment) are adequately captured by a general query.

This gap is exactly why the national-scale collection queries **each of the 15 categories separately at every sampling point**, rather than relying on a single general query — a general query would systematically undercount low-density and non-commercial categories (Nature, Government, Health, Facilities) relative to high-density commercial ones (Food, Shop, Entertainment).
{: .note}

## 1.2 &nbsp;Why Standardization Comes First {#s-standardization}

The most consequential methodological fact underlying this entire study is easy to state and easy to underestimate: **the four alternative datasets don't just disagree about which POIs exist — they disagree about how to describe the POIs they agree on.** Before any distance threshold or similarity score means anything, every record has to be forced into the same shape.

This is not a hypothetical concern. Real POI names go through several concrete transformations before two sources can be fairly compared. The pipeline first produces a **cleaned name** (accent-stripped, uppercased, punctuation-normalized), then a **primary string** — the cleaned name with generic descriptor words, legal-entity suffixes, and stopwords removed, isolating the brand's core identifying token:

| Raw name (source) | After cleaning | After primary-string extraction |
|---|---|---|
| `Café Roma` | `CAFE ROMA` | `ROMA` |
| `McDonald's` | `MCDONALD` | `MCDONALD` |
| `Trader Joe's (Downtown)` | `TRADER JOE` | `TRADER JOE` |
| `Joe's Pizza House LLC` | `JOE PIZZA HOUSE LLC` | `JOE` |
| `7-Eleven #24817` | `7 ELEVEN 24817` | `7 ELEVEN 24817` |
{: .table-wrap}

*↑ the same processing pipeline handles accents, possessives, parentheticals, and generic/legal words — but leaves embedded digits untouched, which is exactly why fuzzy similarity scoring, not exact-string matching, does the final comparison ↑*
{: .figcap}

Four concrete transformations recur across every dataset pair:

- **é→e** — **Unicode normalization** — accented characters are decomposed and stripped (`Café` → `CAFE`)
- **'s→∅** — **Possessive & plural-S removal** — a trailing `'s` or standalone `S` token is dropped when cleaning names (`McDonald's` → `MCDONALD`); this step is not applied when cleaning addresses
- **(...)→∅** — **Parenthetical removal** — any content inside `( )` is dropped entirely, regardless of what it contains
- **generic words→∅** — **Generic & legal-entity word removal** — for names only, a fixed word list (food-business terms like *RESTAURANT/CAFE/PIZZA*, venue terms like *HOTEL/PLAZA/CENTER*, legal suffixes like *LLC/INC/CORP*, and stopwords like *THE/OF/AND*) is stripped to isolate the brand's core token
{: .card-grid}

> **A known limitation, by design**
>
> The cleaning pipeline does not specifically target embedded digits — a store or franchise number left in a name field (`7-Eleven #24817` → `7 ELEVEN 24817`) survives every cleaning step, since digits are treated as ordinary word characters. This residual noise is exactly why the matching stage relies on fuzzy similarity scoring rather than exact-string equality — a name with a trailing number can still score highly similar to the same name without one.
{: .callout}

> **The consequence**
>
> Without correcting for these conventions first, every downstream step degrades silently. A name-similarity score computed on `7-ELEVEN #24817` vs. `7-Eleven` will under-report similarity for reasons that have nothing to do with whether it's the same business — inflating false negatives and making a dataset look less complete than it is. This is exactly why standardization is not an optional cleanup step; it is the precondition for every completeness and accuracy number this study reports.
{: .callout}

### The standardization pipeline

Names and addresses are cleaned with an overlapping but not identical sequence of steps — addresses skip the possessive-removal and primary-string-extraction steps, since generic words like *STREET* or *AVE* are meaningful in an address but not in a business name.

**Applied to both names and addresses:**
1. **Unicode normalization** — NFKD decomposition followed by removal of combining characters, so accented characters compare equal to their unaccented form (`é` → `e`).
2. **Uppercase conversion** — all text is case-folded.
3. **Parenthetical removal** — any text inside `( )` is dropped entirely.
4. **Non-ASCII stripping** — remaining non-ASCII characters (emoji, trademark/registered-mark symbols) are removed via ASCII-only re-encoding.
5. **Punctuation replacement** — remaining non-word, non-whitespace characters are replaced with a space; digits are preserved.
6. **Whitespace collapsing** — repeated spaces are collapsed to one, and the result is trimmed.
{: .steps}

**Applied to names only, after the steps above:**

7. **Possessive/plural-S removal** — a trailing `'s` or a standalone `S` token is removed (`MCDONALD'S` → `MCDONALD`).
8. **Primary-string extraction** — tokens matching a fixed list of generic food/venue words, legal-entity suffixes, or stopwords are removed, leaving only the brand's core identifying token(s). If removing these words would leave a single token shorter than 3 characters, the extraction is skipped and the full cleaned name is kept instead, to avoid over-stripping short brand names.
{: .steps}

Category fields are separately cross-walked to the Google Places Table A classification, since each alternative dataset ships its own category taxonomy.
{: .note}

## 1.3 &nbsp;The Two-Stage Matching Pipeline {#s-matching}

Once records are standardized, matching balances two competing failure modes: missing true matches (hurting recall) and accepting false ones (hurting precision). The pipeline splits this into a cheap, generous first pass and a stricter, learned second pass.

1. **Stage 1 — Rule-based candidate matching**<br>For each Google Places POI, a KD-tree spatial index (built on coordinates reprojected to a metric CRS) finds up to 100 nearby candidates within a 1,000 m radius in the comparison dataset. Candidate names are reduced to primary strings using the same cleaning pipeline as above, and the top 5 candidates by RapidFuzz's WRatio score are shortlisted. Each of those 5 is then re-scored by taking the *highest* of its `WRatio`, `partial_ratio`, `token_sort_ratio`, and `token_set_ratio` values; the single best-scoring candidate among the 5 is accepted as a match only if that combined score is ≥ 80. Address similarity is scored separately (using the same four RapidFuzz metrics, but on addresses cleaned without primary-string extraction) and recorded for every accepted match, alongside its location distance, to be passed forward as features for Stage 2.
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

### Validating the Pipeline: Google Places × NYC DOHMH (Food)

To confirm that the two-stage pipeline actually captures true matches rather than merely producing plausible-looking ones, Google Places Food POIs in New York City were checked against the NYC Department of Health and Mental Hygiene (DOHMH) restaurant inspection dataset — an independent, government-maintained source with strong entity-level ground truth.

- **530** — manually confirmed true POI pairs between Google Places and NYC DOHMH (ground truth)
- **519** — candidate pairs generated by Stage 1 (rule-based matching)
{: .card-grid}

| Metric | Count | Rate |
|---|---:|---:|
| True matches found by Stage 1 | 507 | 95.6% |
| False positives from Stage 1 | 12 | 2.2% (of 530) |
{: .table-wrap}

A 200-sample subset was drawn from the 519 Stage-1 candidates for manual verification, containing 4 of the 12 known false positives, which were relabeled as `False`. This labeled subset was split 75/25 into train/test to train an XGBoost classifier — the same architecture used across the full 56-classifier national pipeline above.

| Class | Precision | Recall | F1-score | Support |
|---|---:|---:|---:|---:|
| 0 (false match) | 1.00 | 1.00 | 1.00 | 1 |
| 1 (true match) | 1.00 | 1.00 | 1.00 | 49 |
| **Accuracy** | | | **1.00** | 50 |
| Macro avg | 1.00 | 1.00 | 1.00 | 50 |
| Weighted avg | 1.00 | 1.00 | 1.00 | 50 |
{: .table-wrap}

AUC = **1.0** on the held-out test set.

> **Before → after Stage 2**
>
> Stage 2 correctly identified 506 of the 507 true matches, and cut the false-positive rate from 2.2% down to 0.1%. Final result for Google Places Food POIs in New York City: **506 true matches (95.4%)**, **1 false positive (0.1%)**, **24 missed matches (4.5%)**.
{: .callout}

## 1.4 &nbsp;The Quality Assessment Framework {#s-framework}

Every alternative dataset is scored on three dimensions, each computed at three levels: pooled across all 14 MSAs, aggregated by population tier, and as a function of distance to the CBD.

- **①** — **Completeness** — the share of Google Places POIs in category *c* that have a verified true match in dataset *d*
- **②** — **Positional accuracy** — the median Euclidean distance, in meters, between matched POI pairs
- **③** — **Spatial heterogeneity** — a weighted-least-squares regression of completeness or location bias against distance to the city center
{: .card-grid}

### Formal definitions

**① Completeness**

$$
\text{Completeness}(d, c) = \frac{|TP(d, c)|}{|GP(c)|}
$$

where $GP(c)$ is the set of Google Places POIs in category $c$, and $TP(d, c)$ is the set of verified true matches for dataset $d$ in category $c$.

**② Positional accuracy**

$$
\text{LocAcc}(d, c) = \underset{i \in TP(d,c)}{\text{median}} \left\lVert \mathbf{p}_i^{GP} - \mathbf{p}_i^{d} \right\rVert_2
$$

the median Euclidean distance between the projected coordinates of each matched pair — median rather than mean, to resist long-tail outliers from bad source coordinates.

**③ Spatial heterogeneity**

$$
Y_b(d, c) = \alpha + \beta_{d,c} \cdot \text{dist\_center}_b + \epsilon_b, \qquad \epsilon_b \sim \mathcal{N}\left(0, \frac{\sigma^2}{n_{b,c}}\right)
$$

where $Y_b$ is either completeness or median location bias in a 2 km distance bin $b$, $\text{dist\_center}_b$ is that bin's midpoint distance to the city center, and $n_{b,c}$ — the number of reference POIs in that bin — serves as the WLS regression weight. The coefficient $\beta_{d,c}$ (per 10 km) captures the strength and direction of the spatial decay gradient.
{: .note}

## 1.5 &nbsp;Results {#s-results}

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

## 1.6 &nbsp;Discussion & Implications {#s-discussion}

- **①** — **SafeGraph & Overture lead**, but by different margins depending on context. Foursquare and OSM show distinct failure modes: Foursquare trades completeness for noisy coordinates; OSM trades positional accuracy for large completeness gaps.
- **②** — **Completeness has a spatial signature that accuracy doesn't.** Match rates decay with metro size and CBD distance across the board; location accuracy is largely flat except for Foursquare and OSM.
- **③** — **OSM's weakness is category-dependent, not uniform** — steep decay for Sport/Culture/Nature, but comparatively mild for Transportation.
- **④** — **No dataset dominates every category.** SafeGraph leads Government/Services/Business/Transportation; the three commercial datasets converge on Food/Worship/Education/Shop.
{: .card-grid}

Practically, this means dataset choice should be **category-specific rather than global**. For coordinate-sensitive work (accessibility, routing, proximity measures), SafeGraph's low error and narrow spread make it the safer default. For broad spatial coverage across metro sizes, SafeGraph and Overture are most reliable, though both still lose ground in smaller and peripheral areas. OSM warrants particular caution in low-population MSAs and for leisure, cultural, and natural-amenity categories. Because completeness — not positional accuracy — carries the systematic urban-core/periphery bias, studies spanning that gradient should correct for coverage rather than coordinate quality.

</div>
</div>
