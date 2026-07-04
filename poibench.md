---
title: POIBench Tool
permalink: /poibench/
---
<div class="page-wrap">
<div class="page-content wide" markdown="1">

<div class="page-eyebrow"><span class="dot"></span>Part III · The Platform</div>

# POIBench: An Interactive Benchmarking Platform

<p class="page-dek">Houpu Li &amp; Wenfei Xu, UCSB — SIGSPATIAL '26 Demo Paper. The same methodology as the study in Part II, turned into a tool any researcher can point at their own study area.</p>

## 3.1 &nbsp;Platform Overview {#b-overview}

POIBench is the first online interactive platform for cross-dataset POI quality benchmarking with LLM-assisted dataset recommendations. It exists to solve a problem the study in Part II ran into directly: national-scale benchmarking gives general guidance, but a researcher planning a study of, say, Goleta, CA, needs to know how these datasets behave *in their own study area*, for *their own categories of interest* — not just the national average.

Given a user-defined study area and target categories, POIBench lets a researcher:

- **1** — Configure a study area and select comparable datasets and categories
- **2** — Retrieve Google Places POIs and run cross-dataset matching automatically
- **3** — Compute completeness and location accuracy, overall and by distance to city center
- **4** — Explore multi-source spatial distributions and matches on an interactive map
- **5** — Get an agent-based dataset recommendation tailored to a described research scenario, and export the report
{: .card-grid}

A static demo is available at [poibench.streamlit.app](https://poibench.streamlit.app/). POIBench is planned as an open-source web application.
{: .note}

## 3.2 &nbsp;System Architecture: Seven Modules {#b-modules}

![POIBench system architecture diagram, seven modules](../assets/poibench/bench_fig1_architecture.png)
*<span class="fig-label">FIG. 1</span>The system architecture of POIBench — from benchmark configuration to dataset recommendation.*
{: .figcap}

1. **Benchmark Configuration**<br>Define a study area (city name → U.S. Census TIGER boundary, or a bounding box), choose comparable datasets, pick Table A categories, and set a sampling radius for the Google Places grid query.
2. **Progress Monitor**<br>A real-time status view across retrieval, standardization, matching, and quality assessment — since the full pipeline is computationally intensive.
3. **POI Standardization & Matching**<br>The same standardization → KD-tree candidate generation → RapidFuzz scoring → XGBoost verification pipeline described in Part II, packaged for on-demand use on any study area.
4. **Quality Assessment & Dashboard**<br>Completeness, location accuracy, and spatial heterogeneity, each rendered as an interactive chart with category-level breakdowns (see 3.3 below).
5. **Interactive Map Explorer**<br>Toggle between all / matched / unmatched POIs, filter by category and match confidence, and click any POI to see its cross-dataset matches with a Street View link for ground-truth verification.
6. **Agent-Based Dataset Recommendation**<br>Describe a research scenario in plain language; an LLM agent synthesizes completeness, positional accuracy, category performance, and spatial heterogeneity into a ranked, weighted recommendation with rationale. Weights on each quality dimension can also be set manually.
7. **Report Export**<br>PDF summary, CSV metric tables, matched-entity datasets, or a shareable HTML report link.
{: .steps}

## 3.3 &nbsp;Quality Dashboard — Los Angeles MSA Demo {#b-dashboard}

The published demo benchmarks food-related POIs in the Los Angeles MSA. The outputs mirror the three-dimension framework from Part II, but computed live for a user-chosen area instead of pooled nationally.

<div class="fig-grid" markdown="1">

<div markdown="1">

![POIBench completeness overall demo LA MSA](../assets/poibench/bench_fig2a_completeness_overall.png)
*<span class="fig-label">FIG. 2a</span>Overall completeness by dataset, LA MSA demo.*
{: .figcap}

</div>
<div markdown="1">

![POIBench location accuracy overall demo LA MSA](../assets/poibench/bench_fig3a_locacc_overall.png)
*<span class="fig-label">FIG. 3a</span>Overall location accuracy by dataset, LA MSA demo.*
{: .figcap}

</div>

</div>

![POIBench completeness by category demo LA MSA](../assets/poibench/bench_fig2b_completeness_by_category.png)
*<span class="fig-label">FIG. 2b</span>Completeness by POI category and dataset, LA MSA demo.*
{: .figcap}

![POIBench location accuracy by category demo LA MSA](../assets/poibench/bench_fig3b_locacc_by_category.png)
*<span class="fig-label">FIG. 3b</span>Location accuracy by POI category and dataset, LA MSA demo.*
{: .figcap}

<div class="fig-grid" markdown="1">

<div markdown="1">

![POIBench match rate vs distance demo LA MSA](../assets/poibench/bench_fig4a_matchrate_vs_distance.png)
*<span class="fig-label">FIG. 4a</span>Match rate vs. distance to city center (WLS), LA MSA demo.*
{: .figcap}

</div>
<div markdown="1">

![POIBench location difference vs distance demo LA MSA](../assets/poibench/bench_fig4b_locdiff_vs_distance.png)
*<span class="fig-label">FIG. 4b</span>Median location difference vs. distance to city center (WLS), LA MSA demo.*
{: .figcap}

</div>

</div>

## 3.4 &nbsp;Limitations & Roadmap {#b-limits}

- **📷** — Datasets are pre-loaded as static snapshots — not suited to time-sensitive studies
- **🌐** — Standardization and matching are optimized for English-language names
- **🇺🇸** — Limited to four comparable datasets, with Google Places as the sole reference — a U.S.-centric design
{: .card-grid}

Planned work includes temporal change detection for longitudinal benchmarking, multilingual standardization and matching, support for user-defined reference benchmarks beyond Google Places, and cross-dataset conflation — combining complementary strengths across sources into a single, more complete POI database.


</div>

</div>
