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

This study compares four alternative POI datasets — Overture Maps, SafeGraph, Foursquare, and OpenStreetMap — against Google Places. Google Places is used as the reference, since it is widely seen as the most complete and accurate source.

The study covers **14 U.S. Metropolitan Statistical Areas**, split into three population tiers: high, mid, and low. We first collected Google Places POIs across each area to use as ground truth. Then we matched each alternative dataset against these POIs. This gives three scores for every dataset: completeness, positional accuracy, and spatial heterogeneity, across 15 categories.

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

To build the reference set, we placed sample points in a grid across each MSA, 5 km apart. At every point, we queried the Google Places API once for each of the 15 categories, within a 5 km radius (up to 20 POIs per query). We left out *Facilities* and *Housing*: Facilities has too few subtypes to compare well, and most alternative datasets don't have a matching category for Housing.
{: .note}

### Why Query by Category, Not Generally

By default, the Google Places API ranks results by popularity, not distance, and returns only a small number per request. We ran a pilot study in Isla Vista, CA (bounding box: lon −119.8694 to −119.85346, lat 34.40887 to 34.41727; 100 m radius) to test this. We compared one **general query** (no category filter) against **category-specific queries** (one query per category), to see if a general query alone is enough.

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

The result is clear: a general query misses the most in **Facilities, Health, and Nature** (100% missing), then **Business** (66.67%) and **Finance** (50%). Only popular categories like Food, Shop, and Entertainment come back well in a general query.

This is why we query **each of the 15 categories separately, at every sample point**, instead of using one general query. A general query would miss low-density, non-commercial categories like Nature, Government, and Facilities much more than common ones like Food or Shop.
{: .note}

## 1.2 &nbsp;Why Standardization Comes First {#s-standardization}

Here is the core problem this study had to solve: **the four alternative datasets don't just disagree on which POIs exist — they also write the same POI differently.** Before we can compare names, addresses, or locations, every record has to be put into the same format first.

This is a real problem, not a small one. The same business can look different across sources, because each provider has its own way of writing names. We first produce a **cleaned name** (accents removed, uppercase, punctuation removed). Then we produce a **primary string**: the cleaned name with generic words, legal suffixes, and stopwords removed, so only the core brand name is left.

| Raw name (source) | After cleaning | After primary-string extraction |
|---|---|---|
| `Café Roma` | `CAFE ROMA` | `ROMA` |
| `McDonald's` | `MCDONALD` | `MCDONALD` |
| `Trader Joe's (Downtown)` | `TRADER JOE` | `TRADER JOE` |
| `Joe's Pizza House LLC` | `JOE PIZZA HOUSE LLC` | `JOE` |
| `7-Eleven #24817` | `7 ELEVEN 24817` | `7 ELEVEN 24817` |
{: .table-wrap}

### The standardization pipeline

Names and addresses go through similar, but not identical, cleaning steps. Addresses skip two steps — removing possessives and extracting a primary string — because words like *STREET* or *AVE* matter in an address, but not in a business name.

**Applied to both names and addresses:**
1. **Unicode normalization** — Strip accents so they match their plain letter (`é` → `e`).
2. **Uppercase conversion** — Make everything uppercase.
3. **Parenthetical removal** — Delete anything inside `( )`.
4. **Non-ASCII stripping** — Remove emoji and trademark symbols.
5. **Punctuation replacement** — Replace punctuation with a space. Keep digits.
6. **Whitespace collapsing** — Turn repeated spaces into one space, and trim the ends.
{: .steps}

**Applied to names only, after the steps above:**

7. **Possessive/plural-S removal** — Drop a trailing `'s` or a standalone `S` (`MCDONALD'S` → `MCDONALD`).
8. **Primary-string extraction** — Remove generic food/venue words, legal suffixes, and stopwords, so only the core brand name is left. If this would leave just one word under 3 letters, we skip this step and keep the full cleaned name instead — this stops us from over-trimming short brand names.
{: .steps}

Category fields are matched to the Google Places Table A classification separately, since every alternative dataset uses its own category system.
{: .note}

## 1.3 &nbsp;The Two-Stage Matching Pipeline {#s-matching}

Once every record is cleaned, we start matching. There are two ways matching can go wrong: missing a real match (hurts recall), or accepting a wrong match (hurts precision). So the pipeline runs in two passes: a cheap, loose first pass, then a stricter, learned second pass.

1. **Stage 1 — Rule-based candidate matching**<br>For each Google Places POI, we use a KD-tree spatial index to find up to 100 nearby candidates within 1,000 m. We reduce each candidate name to a primary string, then shortlist the top 5 by RapidFuzz's WRatio score. For each of those 5, we take the highest score across `WRatio`, `partial_ratio`, `token_sort_ratio`, and `token_set_ratio`. The single best-scoring candidate is accepted as a match only if that score is 80 or higher. We also score address similarity separately (same four RapidFuzz metrics, but on addresses without primary-string extraction), and record it along with the location distance. These become features for Stage 2.
2. **Stage 2 — ML-based match verification**<br>When we checked Stage 1's output by hand, about 10.3% of matches were wrong — and this rate varied a lot by MSA and by dataset. Naming style, address format, and how dense an area is all differ between metros, so one fixed rule can't work everywhere. To fix this, we train a **separate XGBoost classifier for every MSA × dataset combination** (56 classifiers in total), using three features: name similarity, location distance, and address similarity.
{: .steps}

### Understanding Levenshtein Distance

Levenshtein Distance is the **minimum number of edits** needed to turn one string into another. It's computed with dynamic programming. Each edit — insertion, deletion, or substitution — counts as one step:
- Insertions
- Deletions
- Substitutions

> **Process**
>
> For example, turning **kitten** into **sitting** takes 3 edits:
>
> ```text
> A = kitten
> B = sitting
> ```
>
> **Step 1:** Build a matrix with size (len(A)+1) × (len(B)+1).
>
> ```text
>       ''  s  i  t  t  i  n  g
>    -----------------------------
> '' |  0  1  2  3  4  5  6  7
> k  |  1
> i  |  2
> t  |  3
> t  |  4
> e  |  5
> n  |  6
> ```
>
> **Step 3:** Fill in the matrix, one letter pair at a time:
>
> ```text
> k → s  => +1 (substitution)
> i → i  => 0 (no change)
> t → t  => 0 (no change)
> t → t  => 0 (no change)
> e → i  => +1 (substitution)
> n → n  => 0 (no change)
> insert g at the end  => +1 (insertion)
> ```
>
> **Step 4:** The final Levenshtein distance is **3**:
>
> ```text
> k → s  => +1 (substitution)
> e → i  => +1 (substitution)
> insert g at the end  => +1 (insertion)
> ```
>
> More details: [Levenshtein Distance](https://yuminlee2.medium.com/levenshtein-distance-1080038a4d9)
>
> ![Levenshtein distance matrix](../assets/poi_study/levenstain distance intro.png)
{: .note}

### Weighted Rapid Fuzzy Matching

`RapidFuzz` is a fast Python library for string matching. It's a faster, more advanced version of `Fuzzywuzzy`, and it's built to handle large datasets. Here are its main scoring functions:

#### Scorer

- **Pure Levenshtein Distance Scorer:** `Levenshtein.distance`
- **Levenshtein-Based Scorers:** `fuzz.ratio` (same as `Levenshtein.distance`), `fuzz.QRatio` (same, just faster), `fuzz.partial_ratio`, `fuzz.token_sort_ratio`, `fuzz.token_set_ratio`
- **Enhanced Scorer:** **`fuzz.WRatio`**

| Function | Uses Levenshtein Internally? | Additional Processing | Typical Use Case |
|---|---|---|---|
| `ratio` | Yes (100%) | No | Basic Levenshtein similarity |
| `QRatio` | Yes | No | Faster version of `ratio` |
| `partial_ratio` | Yes | Substring extraction | Strong for partial matches; handles long–short gaps |
| `token_sort_ratio` | Yes | Token sorting | Same words but different order |
| `token_set_ratio` | Yes | Token set operations | One string is a superset/subset of the other |
| **`WRatio`** | Yes | Combines `ratio`, `partial_ratio`, `token_sort_ratio`, `token_set_ratio` + length-based scaling | Most robust; handles noise, reordering, and length differences |
{: .table-wrap}

```python
fuzz.ratio("this is a test", "this is a test!")
# 96.55172413793103

fuzz.QRatio("this is a test", "this is a test!")
# 96.55172413793103

fuzz.partial_ratio("this is a test", "this is a a test")
# 100.0

fuzz.token_sort_ratio("this is a test", "is this a test")
# 100.0

fuzz.token_set_ratio("this is a test", "is is a test")
# 100.0

fuzz.token_set_ratio("this is a test", "this is not a test")
# 92.3076923076923

fuzz.WRatio("this is a test", "is this always a TEST???")
# 100
```

#### Further resources

GitHub repo: [RapidFuzz](https://github.com/rapidfuzz/RapidFuzz)
Full documentation: [RapidFuzz 3.14.3 Documentation](https://rapidfuzz.github.io/RapidFuzz/Usage/distance/index.html)
{: .note}

- **84,000** — manually labeled candidate pairs, 75/25 train-test split per MSA-dataset combination
- **0.97** — median precision across 56 classifiers
- **0.98** — median recall & F1-score
- **0.96** — median accuracy
{: .card-grid}

### Validating the Pipeline: Google Places × NYC DOHMH (Food)

To check that the pipeline finds real matches — not just matches that look plausible — we ran one extra test. We checked Google Places Food POIs in New York City against the NYC Department of Health and Mental Hygiene (DOHMH) restaurant inspection dataset. This is an independent, government-run source, so it makes a strong ground truth.

- **530** — manually confirmed true POI pairs between Google Places and NYC DOHMH (ground truth)
- **519** — candidate pairs generated by Stage 1 (rule-based matching)
{: .card-grid}

| Metric | Count | Rate |
|---|---:|---:|
| True matches found by Stage 1 | 507 | 95.6% |
| False positives from Stage 1 | 12 | 2.2% (of 530) |
{: .table-wrap}

We drew a 200-sample subset from the 519 Stage-1 candidates to check by hand. This subset included 4 of the 12 known false positives, which we relabeled as `False`. We split this labeled subset 75/25 into train and test, and trained an XGBoost classifier — the same setup used in the full 56-classifier pipeline above.

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
> Stage 2 found 506 of the 507 true matches, and cut the false-positive rate from 2.2% down to 0.1%. Final result for Google Places Food POIs in New York City: **506 true matches (95.4%)**, **1 false positive (0.1%)**, **24 missed matches (4.5%)**.
{: .callout}

## 1.4 &nbsp;The Quality Assessment Framework {#s-framework}

We score every alternative dataset on three measures. Each one is computed three ways: pooled across all 14 MSAs, by population tier, and as a function of distance to the CBD.

- **①** — **Completeness** — the share of Google Places POIs in category *c* that have a verified true match in dataset *d*
- **②** — **Positional accuracy** — the median distance, in meters, between matched POI pairs
- **③** — **Spatial heterogeneity** — how completeness or location bias changes with distance to the city center, using a weighted regression
{: .card-grid}

### Formal definitions

**① Completeness**

$$
\text{Completeness}(d, c) = \frac{|TP(d, c)|}{|GP(c)|}
$$

$GP(c)$ is the set of Google Places POIs in category $c$. $TP(d, c)$ is the set of verified true matches for dataset $d$ in category $c$.

**② Positional accuracy**

$$
\text{LocAcc}(d, c) = \underset{i \in TP(d,c)}{\text{median}} \left\lVert \mathbf{p}_i^{GP} - \mathbf{p}_i^{d} \right\rVert_2
$$

This is the median distance between the coordinates of each matched pair. We use the median, not the mean, so a few bad coordinates don't skew the result.

**③ Spatial heterogeneity**

$$
Y_b(d, c) = \alpha + \beta_{d,c} \cdot \text{dist\_center}_b + \epsilon_b, \qquad \epsilon_b \sim \mathcal{N}\left(0, \frac{\sigma^2}{n_{b,c}}\right)
$$

$Y_b$ is either completeness or median location bias, inside a 2 km distance bin $b$. $\text{dist\_center}_b$ is that bin's distance to the city center. $n_{b,c}$ is the number of reference POIs in that bin, and it acts as the weight in the regression. The coefficient $\beta_{d,c}$ (per 10 km) tells us how fast, and in which direction, the score changes with distance.
{: .note}

## 1.5 &nbsp;Results {#s-results}

### Overall completeness

Across all 14 MSAs, *SafeGraph*{: .badge-safegraph} has the highest true match rate, at 71.8%. *Overture*{: .badge-overture} is close behind at 69.5%, then *Foursquare*{: .badge-foursquare} at 64.4%. *OSM*{: .badge-osm} is roughly half the others, at 31.8% — this fits its volunteer-driven collection model. False-positive rates stay low across the board (8.2–13.3%), which tells us Stage 1 generates good candidates.

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

Every dataset's true match rate drops as the city gets smaller, and as we move further from the CBD. OSM drops the most — its relative decline (−33.1%) is about twice that of Overture or Foursquare. So OSM's completeness is the most sensitive to both city size and distance from downtown. Overture stands out here: it's the only dataset with a statistically significant distance-decay trend at all three population tiers (see table below).

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

*SafeGraph*{: .badge-safegraph} has both the smallest median error (4.8 m) and the tightest spread. *Overture*{: .badge-overture} (12.9 m) and *OSM*{: .badge-osm} (12.7 m) land at almost the same median, but Overture's results are much more consistent. *Foursquare*{: .badge-foursquare} has the worst median (16.7 m) and the widest spread — its upper whisker goes past 100 m.

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
> Completeness reacts strongly to space — it drops for every dataset as cities get smaller or as we move away from downtown. Positional accuracy doesn't behave this way. Overture and SafeGraph show **no clear spatial trend** at any population tier. Only Foursquare and OSM get worse with distance. This suggests completeness depends on how much ground a dataset covers, while positional accuracy depends more on how the source collects its data in the first place.
{: .callout}

### Category-specific patterns

Looking across all three tiers, three completeness groups appear. **Food, Worship, Education, and Shop** are covered well everywhere (Overture and SafeGraph both around 80%, with little variation between tiers). A middle group — Finance, Automotive, Health, Government, Sport, Services, Culture, Entertainment, Business — sits between 50% and 80%. **Transportation and Nature** are the weakest group; Nature stays under 10% for every dataset. OSM lags behind in almost every category, worst of all in Services (15.0%) and Business (6.3%).

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

Completeness drops fastest with distance for **Transportation, Entertainment, Finance, Sport, Nature, and Culture**. It drops at a medium pace for Government, Health, and Education. It barely drops at all for Shop, Business, Services, Worship, Food, and Automotive. OSM usually drops the fastest within each group (for example, Sport: −2.87 pp per 10 km) — except in Transportation, where Foursquare and Overture drop even faster than OSM.

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

- **①** — **SafeGraph and Overture lead**, but by different amounts depending on the situation. Foursquare and OSM each fail in their own way: Foursquare gives up coordinate accuracy for completeness; OSM gives up completeness for coordinate accuracy.
- **②** — **Completeness reacts to space; accuracy mostly doesn't.** Match rates drop with smaller city size and greater distance from the CBD, for every dataset. Location accuracy stays mostly flat, except for Foursquare and OSM.
- **③** — **OSM's weak spots depend on category.** It drops fast for Sport, Culture, and Nature, but drops much less for Transportation.
- **④** — **No dataset wins everywhere.** SafeGraph leads in Government, Services, Business, and Transportation. The three commercial datasets all do well in Food, Worship, Education, and Shop.
{: .card-grid}

In practice, this means the right dataset depends on the category, not a single "best" choice. If your work depends on exact coordinates — accessibility, routing, proximity — SafeGraph is the safest default, thanks to its low error and tight spread. If you need broad coverage across cities of different sizes, SafeGraph and Overture are the most reliable, though both still lose ground in smaller and outlying areas. Be careful with OSM in low-population MSAs, and for leisure, culture, and nature categories. And since completeness — not positional accuracy — carries the bias between city centers and edges, studies that span that gradient should correct for coverage, not coordinate quality.

</div>
</div>