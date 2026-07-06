---
title: POIBench Tool
permalink: /poibench/
---
<div class="page-wrap">
<div class="page-content wide" markdown="1">

<div class="page-eyebrow"><span class="dot"></span>Part III · The Platform</div>

# POIBench: An Interactive Benchmarking Platform

<p class="page-dek">How we designed and built an interactive benchmarking tool that lets researchers evaluate any city and any specific category of interest, backed by an LLM-based recommendation system — how to use it, and what's next.</p>

## 1.1 &nbsp;Why We Built This {#b-why}

Part II answers a national question: across the U.S., which POI dataset is best overall? But that answer doesn't always help one researcher with one specific question. Say you're studying restaurants in Goleta, CA. The national average for Overture doesn't tell you how Overture performs in Goleta, for restaurants specifically.

POIBench closes that gap. It's the same benchmarking method from Part II, packaged into a tool. Instead of running it once, at national scale, on our terms, anyone can run it on their own city, for their own categories, on demand.

- **1** — Configure a study area and select comparable datasets and categories
- **2** — Retrieve Google Places POIs and run cross-dataset matching automatically
- **3** — Compute completeness and location accuracy, overall and by distance to city center
- **4** — Explore multi-source spatial distributions and matches on an interactive map
- **5** — Get an agent-based dataset recommendation tailored to a described research scenario, and export the report
{: .card-grid}

A static demo is live at [poibench.streamlit.app](https://poibench.streamlit.app/). POIBench is planned as an open-source web application.
{: .note}

## 1.2 &nbsp;How It's Built: Seven Modules {#b-modules}

![POIBench system architecture diagram, seven modules](../assets/poibench/bench_fig1_architecture.png)
*<span class="fig-label">FIG. 1</span>The system architecture of POIBench — from benchmark configuration to dataset recommendation.*
{: .figcap}

1. **Benchmark Configuration**<br>Pick a study area (type a city name, and it pulls the U.S. Census TIGER boundary — or draw your own bounding box). Choose which datasets to compare, pick Table A categories, and set a sampling radius for the Google Places grid query.
2. **Progress Monitor**<br>The full pipeline — retrieval, standardization, matching, scoring — takes real compute time. This module shows live status as each step runs.
3. **POI Standardization & Matching**<br>Same pipeline as Part II: standardize → find candidates with a KD-tree → score with RapidFuzz → verify with XGBoost. Packaged here so it runs on-demand, on any area you pick.
4. **Quality Assessment & Dashboard**<br>Completeness, location accuracy, and spatial heterogeneity, each as an interactive chart, with category-level breakdowns (see 1.3 below).
5. **Interactive Map Explorer**<br>Switch between all POIs, matched POIs, and unmatched POIs. Filter by category and match confidence. Click any POI to see its matches across datasets, plus a Street View link to check the ground truth yourself.
6. **Agent-Based Dataset Recommendation**<br>Describe your research scenario in plain language. An LLM agent reads the completeness, accuracy, category, and heterogeneity scores, and ranks datasets for you, with reasoning. You can also set the weights yourself, if you want more control.
7. **Report Export**<br>Download a PDF summary, CSV tables, matched-entity data, or a shareable HTML report link.
{: .steps}

## 1.3 &nbsp;How to Use It: A Walkthrough {#b-walkthrough}

Here's what running a benchmark actually looks like, using the published demo: food-related POIs across the Los Angeles MSA. The outputs are the same three measures from Part II — completeness, positional accuracy, spatial heterogeneity — just computed live, for the area you pick, instead of pooled across the whole country.

**Step 1 — Check overall completeness and accuracy.** These two charts give you the big picture first: which dataset covers the most ground, and which one places POIs most accurately.

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

**Step 2 — Drill into your category.** The overall number can hide a lot. A dataset might be great for restaurants but weak for parks. These breakdowns let you check your specific category, not just the average.

![POIBench completeness by category demo LA MSA](../assets/poibench/bench_fig2b_completeness_by_category.png)
*<span class="fig-label">FIG. 2b</span>Completeness by POI category and dataset, LA MSA demo.*
{: .figcap}

![POIBench location accuracy by category demo LA MSA](../assets/poibench/bench_fig3b_locacc_by_category.png)
*<span class="fig-label">FIG. 3b</span>Location accuracy by POI category and dataset, LA MSA demo.*
{: .figcap}

**Step 3 — Check for spatial bias.** If your study area spans downtown and the suburbs, this matters a lot. These charts show whether a dataset gets worse the further you go from the city center.

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

**Step 4 — Ask the agent.** Once you've seen the numbers, describe your project to the recommendation agent (Module 6). It reads all four scores together and tells you which dataset fits your specific case — not just which one wins on average.
{: .note}

## 1.4 &nbsp;What's Next {#b-next}

POIBench has three limits right now:

- **📷** — Datasets are static snapshots, not live feeds — not a good fit for time-sensitive studies
- **🌐** — Standardization and matching are tuned for English-language names
- **🇺🇸** — Only four alternative datasets are supported, and Google Places is the only reference — this makes the tool U.S.-centric for now
{: .card-grid}

Here's what we're planning to fix that: temporal change detection, so benchmarks can track how a dataset changes over time; multilingual standardization and matching, so this works outside English-speaking regions; support for reference benchmarks other than Google Places; and cross-dataset conflation, so we can combine the strengths of multiple sources into one, more complete POI database.

</div>
</div>