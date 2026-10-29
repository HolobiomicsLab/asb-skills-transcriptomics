---
name: mass-cytometry-pseudotime-inference
description: Use when a proteomic (CyTOF/mass cytometry) time-series single-cell dataset
  annotated with sampling-day labels needs pseudotime ordering, or when benchmarking
  whether lazy-teleporting MCMC (memory) improves pseudotime over TI baselines.
when_to_use_negative:
- Input is scRNA-seq or an already-computed pseudotime; this skill targets proteomic
  CyTOF data with true day labels
- The goal is topology/lineage discovery without any time-point labels to validate
  against — the Pearson-vs-days metric cannot be computed
- Data requires no ablation question (i.e., you are not testing the lazy-teleporting
  MCMC's contribution against memory=1 or TI baselines)
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: VIA (pyVIA)
  role: Primary trajectory inference method; computes pseudotime via lazy-teleporting
    random walks with MCMC on a PARC-clustered KNN graph; memory=1 reproduces the
    ablated Via 1.0 mode
  repo: https://github.com/ShobiStassen/VIA
- name: PARC
  role: Clustering algorithm used inside VIA to build the cluster-graph of cells (edgepruning_clustering_resolution=0.15)
  repo: https://github.com/ShobiStassen/VIA
- name: SlingShot
  role: TI baseline run on the same CyTOF data and KNN settings for correlation comparison
- name: Palantir
  role: TI baseline run on the same CyTOF data and KNN settings for correlation comparison
- name: STREAM
  role: TI baseline run on the same CyTOF data and KNN settings for correlation comparison
- name: pandas
  role: Computes Pearson correlations of each method's pseudotime vs day labels and
    assembles cytof_pseudotime_correlations.csv
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-25773-3
    title: Generalized and scalable trajectory inference in single-cell omics data
      with VIA
  source_content_hash: sha256:e7d0cbaf82c3a2a446e0d01258b3330a45f013f5b2e931032141e3101f5d8fde
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ShobiStassen/VIA
  tools:
  - PARC
  - Palantir
  - STREAM
  - SlingShot
  - VIA
  - VIA (pyVIA)
  - pandas
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# mass-cytometry-pseudotime-inference

## Summary

Infers developmental pseudotime on mass cytometry (CyTOF) time-series data using VIA's lazy-teleporting MCMC random walks, and quantifies accuracy by Pearson correlation of inferred pseudotime against annotated sampling days (~0.88 on mESC CyTOF).

## When to use

Use when a proteomic (CyTOF/mass cytometry) time-series single-cell dataset annotated with sampling-day labels needs pseudotime ordering, or when benchmarking whether lazy-teleporting MCMC (memory) improves pseudotime over TI baselines.

## When NOT to use

- Input is scRNA-seq or an already-computed pseudotime; this skill targets proteomic CyTOF data with true day labels
- The goal is topology/lineage discovery without any time-point labels to validate against — the Pearson-vs-days metric cannot be computed
- Data requires no ablation question (i.e., you are not testing the lazy-teleporting MCMC's contribution against memory=1 or TI baselines)

## Inputs

- mESC CyTOF mass cytometry time-series dataset (~90,000 cells, 28 antibody markers) with annotated sampling-day labels from pyVIA Datasets (mESC_timeseries)
- KNN sweep settings around knn=30 for the HNSWlib graph
- PARC clustering parameters (edgepruning_clustering_resolution=0.15, pseudotime_threshold_TS=30)

## Outputs

- single_cell_pt_markov pseudotime values per cell for VIA and the memory=1 ablated run
- Comparable pseudotime orderings from Slingshot, Palantir, STREAM and PAGA
- Table of Pearson correlations of each method's pseudotime vs annotated days across KNN settings, saved as cytof_pseudotime_correlations.csv

## How to apply

Load the mESC CyTOF time-series dataset (~90K cells, 28 antibody markers) with annotated sampling-day labels from the pyVIA Datasets resources (mESC_timeseries). Run VIA via via.via_wrapper on an HNSWlib KNN graph, sweeping KNN around the default knn=30, using PARC clustering with edgepruning_clustering_resolution=0.15 and pseudotime_threshold_TS=30 to obtain single_cell_pt_markov pseudotime. Re-run VIA with memory=1 (Via 1.0 non-memory mode) under the same KNN sweep to ablate the lazy-teleporting stage, and run Slingshot, Palantir, STREAM and PAGA baselines with comparable settings. In pandas, compute the Pearson correlation of each method's inferred pseudotime against the annotated day labels for every KNN setting, then assemble the correlations into a single table (cytof_pseudotime_correlations.csv).

## Related tools

- **VIA (pyVIA)** (Primary trajectory inference method; computes pseudotime via lazy-teleporting random walks with MCMC on a PARC-clustered KNN graph; memory=1 reproduces the ablated Via 1.0 mode) — https://github.com/ShobiStassen/VIA
- **PARC** (Clustering algorithm used inside VIA to build the cluster-graph of cells (edgepruning_clustering_resolution=0.15)) — https://github.com/ShobiStassen/VIA
- **SlingShot** (TI baseline run on the same CyTOF data and KNN settings for correlation comparison)
- **Palantir** (TI baseline run on the same CyTOF data and KNN settings for correlation comparison)
- **STREAM** (TI baseline run on the same CyTOF data and KNN settings for correlation comparison)
- **pandas** (Computes Pearson correlations of each method's pseudotime vs day labels and assembles cytof_pseudotime_correlations.csv)

## Examples

```
v = via.via_wrapper(adata, true_label, embedding, knn=30, jac_std_global=0.15, root=[42], pseudotime_threshold_TS=30, memory=1)
```

## Evaluation signals

- VIA's pseudotime-vs-day Pearson correlation is approximately 0.88, matching the paper's CyTOF benchmark
- VIA's correlation exceeds that of the memory=1 (no lazy-teleporting) ablation, which drops to near PAGA's level
- VIA's correlation exceeds Slingshot, Palantir and STREAM baselines across the KNN sweep
- cytof_pseudotime_correlations.csv contains one correlation per method per KNN setting with no missing entries

## Limitations

- Ablating the lazy-teleporting stage (memory=1) substantially degrades pseudotime accuracy to near PAGA's level, so the non-memory mode should not be relied on for CyTOF ordering
- Results depend on KNN and PARC pruning settings (e.g., edgepruning_clustering_resolution, pseudotime_threshold_TS); a sweep is required to show robustness
- No software version numbers for VIA or the compared methods are reported in the article, complicating exact reproduction

## Evidence

- [discussion] VIA outperforms baselines on mESC CyTOF via lazy-teleporting MCMC: "We first applied VIA to the mESC CyTOF dataset and showed that the lazy-teleporting MCMCs strategy in VIA enables it to outperform other methods"
- [methods] PARC clustering and pseudotime threshold defaults: [evidence span withheld — anchor status: not_found]
- [intro] VIA graph-based TI with lazy-teleporting random walks: "we present VIA, a graph-based TI algorithm that uses a new strategy to compute pseudotime, and reconstruct cell lineages based on lazy-teleporting random walks"
- [readme] Memory/higher-order walks rationale from StaVia README: "StaVia extends the lazy-teleporting walks to higher order random walks with **memory** to allow better lineage detection, pathway recovery and preservation of global features"
- [readme] Dataset availability for reproduction: "Dataset are available in the [Datasets folder](https://github.com/ShobiStassen/VIA/tree/master/Datasets) (smaller files)"
