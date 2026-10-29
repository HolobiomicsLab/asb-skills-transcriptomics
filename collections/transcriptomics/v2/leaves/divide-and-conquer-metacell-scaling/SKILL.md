---
name: divide-and-conquer-metacell-scaling
description: Use when planning metacell construction on large scRNA-seq atlases (hundreds
  of thousands of cells) and you need to know which method/variant is computationally
  tractable at a given cell count and graining level.
when_to_use_negative:
- Dataset is small (e.g. ~10k cells) — all methods run tractably and the divide-and-conquer/projection
  variants add overhead without benefit
- Goal is metacell quality assessment (purity, compactness, separation, INV) rather
  than computational-cost scaling
- The atlas cannot be stratified into size subsets, so no scaling curve can be fit
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: MC2 (tanaylab/metacells)
  role: Metacell construction method; run standard and divide-and-conquer variants
    at graining level 75 on each MOCA subset
  repo: https://github.com/tanaylab/metacells
- name: SuperCell
  role: Metacell construction method; run standard and projection-based variants at
    graining level 75
  repo: https://github.com/GfellerLab/SuperCell
- name: SEACells
  role: Metacell construction method; run only on subset sizes tractable for the method
  repo: https://github.com/dpeerlab/SEACells
- name: MetacellAnalysisToolkit
  role: Integrated command-line pipeline (MATK) to run SEACells, SuperCell, or MetaCell2
    with a joined R package for QC and visualization
  repo: https://github.com/GfellerLab/MetacellAnalysisToolkit
- name: Seurat
  role: Single-cell preprocessing (normalization, variable features, scaling, PCA,
    UMAP) prior to metacell construction
- name: ggplot2
  role: Plot CPU time and peak memory scaling curves as fig5c_scaling_curves.png
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s44320-024-00045-6
    title: Building and analyzing metacells in single-cell genomics data
  source_content_hash: sha256:bf92730a6405dce8cbb4e3eb8e0512df4095027750f01db31d2d33c50cfaf867
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/tanaylab/metacells
  tools:
  - MC2 (tanaylab/metacells)
  - MetacellAnalysisToolkit
  - SEACells
  - Seurat
  - SuperCell
  - ggplot2
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# divide-and-conquer-metacell-scaling

## Summary

Benchmark how CPU time and peak memory of metacell construction (MC2, SuperCell, SEACells) scale with dataset size, using divide-and-conquer MC2 and projection-based SuperCell variants to reach very large datasets (up to 900k MOCA cells) at graining level 75.

## When to use

Use when planning metacell construction on large scRNA-seq atlases (hundreds of thousands of cells) and you need to know which method/variant is computationally tractable at a given cell count and graining level.

## When NOT to use

- Dataset is small (e.g. ~10k cells) — all methods run tractably and the divide-and-conquer/projection variants add overhead without benefit
- Goal is metacell quality assessment (purity, compactness, separation, INV) rather than computational-cost scaling
- The atlas cannot be stratified into size subsets, so no scaling curve can be fit

## Inputs

- MOCA mouse organogenesis scRNA-seq count matrices (GEO accession GSE114953, Cao et al 2019)
- Size-stratified cell subsets from 10,000 to 900,000 cells
- Graining level parameter (75) for metacell construction

## Outputs

- scaling_benchmark.csv with elapsed CPU time and peak memory per method, variant, and dataset size
- fig5c_scaling_curves.png showing CPU time and peak memory versus number of cells for each method and variant (ggplot2)

## How to apply

Download the MOCA scRNA-seq count matrices (GEO GSE114953, Cao et al 2019) and build size-stratified subsets spanning 10k to 900k cells. For each subset, construct metacells at graining level 75 with MC2 (tanaylab/metacells), SuperCell (GfellerLab/SuperCell), and SEACells where tractable, adding the divide-and-conquer MC2 variant on the largest subsets and the projection-based SuperCell variant, plus a per-sample parallel MOCA run. Instrument every construction run to record elapsed CPU time and peak memory per method, variant, and dataset size. Assemble measurements into scaling_benchmark.csv and plot CPU time and peak memory versus cell count per method/variant with ggplot2 (fig5c_scaling_curves.png). The rationale is that divide-and-conquer MC2 and projection SuperCell extend feasibility to the largest datasets, whereas standard MC2, SuperCell, and SEACells become time- and memory-consuming.

## Related tools

- **MC2 (tanaylab/metacells)** (Metacell construction method; run standard and divide-and-conquer variants at graining level 75 on each MOCA subset) — https://github.com/tanaylab/metacells
- **SuperCell** (Metacell construction method; run standard and projection-based variants at graining level 75) — https://github.com/GfellerLab/SuperCell
- **SEACells** (Metacell construction method; run only on subset sizes tractable for the method) — https://github.com/dpeerlab/SEACells
- **MetacellAnalysisToolkit** (Integrated command-line pipeline (MATK) to run SEACells, SuperCell, or MetaCell2 with a joined R package for QC and visualization) — https://github.com/GfellerLab/MetacellAnalysisToolkit
- **Seurat** (Single-cell preprocessing (normalization, variable features, scaling, PCA, UMAP) prior to metacell construction)
- **ggplot2** (Plot CPU time and peak memory scaling curves as fig5c_scaling_curves.png)

## Examples

```
conda env create -f env/MetacellAnalysisToolkit_env.yml
```

## Evaluation signals

- scaling_benchmark.csv covers every method x variant x dataset-size combination
- Recorded CPU-time and memory curves reproduce the Fig 5C scaling behavior
- Divide-and-conquer MC2 and projection-based SuperCell runs complete on the 900k-cell dataset while the standard variants do not
- Per-sample parallel MOCA run times are captured as a separate variant in the benchmark table
- All construction runs used graining level 75 so curves are comparable across methods

## Limitations

- Except for divide-and-conquer MC2, metacell construction is time- and memory-consuming on large datasets
- SEACells is tractable only on a subset of dataset sizes, so it cannot be benchmarked across the full 10k–900k range
- Benchmark results are specific to graining level 75 and the MOCA dataset; other graining levels or atlases may shift the scaling behavior

## Evidence

- [abstract] a fully integrated pipeline to rapidly build, visualize and evaluate metacells with different methods: "a fully integrated pipeline to rapidly build, visualize and evaluate metacells with different methods"
- [readme] scalable for larger data sets: [evidence span withheld — anchor status: external_doc]
- [readme] SEACells identifies metacells: "SEACells identifies **metacells**: groups of cells in the same biological state, found by
archetypal analysis on a nearest-neighbor kernel built from a low-dimensional embedding"
- [readme] Toolkit supports three methods: "It consists of the `MATK` command line tool to easily identify metacells with either [SEACells](https://github.com/dpeerlab/SEACells), [SuperCell](https://github.com/GfellerLab/SuperCell) or"
