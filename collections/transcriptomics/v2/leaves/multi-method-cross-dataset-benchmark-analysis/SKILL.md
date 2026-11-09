---
name: multi-method-cross-dataset-benchmark-analysis
description: Use when you must objectively compare multiple patient-level single-cell/pathomics
  analysis methods across several public datasets, requiring per-method clustering
  and trajectory metrics plus statistically corrected method rankings.
when_to_use_negative:
- You need to analyze a single dataset with a single method (e.g., PILOT trajectory
  estimation on one cohort) rather than statistically compare competing methods across
  many datasets
- Your patient samples have reliable labels and you want supervised patient classification
  — the compared methods (Multi-scale PHATE, SCANCell) require patient labels, whereas
  this benchmark targets unsupervised sample-level analysis
- Your data mixes distinct technologies (single nucleus vs. single cell) or isolation
  methods (droplet vs. well based), which the article states is an open challenge
  invalidating uniform comparison
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: PILOT
  role: Method under test; computes Wasserstein distances between patient cluster
    distributions via optimal transport (emd2) for clustering and trajectory benchmarking
  repo: https://github.com/CostaLab/PILOT
- name: PhEMD
  role: Competing method; EMD-based specimen distances with Monocle 2 for normalization,
    dimension reduction, and clustering of cells
- name: SCANCell
  role: Competing method; uses PhEMD non-linear embeddings followed by association
    networks across cell clusters
- name: Multi-scale PHATE
  role: Competing method; non-linear embeddings and multi-resolution clustering of
    cells at distinct resolutions
- name: POT library
  role: Provides emd2 function implementing the Wasserstein-distance solver used by
    PILOT
- name: Seurat
  role: Uniform pre-processing pipeline for normalization and clustering of pathomics
    structures
- name: Monocle 2
  role: Pre-processing (normalization, dimension reduction, clustering) required for
    the PhEMD input pipeline
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s44320-023-00003-8
    title: Detection of PatIent-Level distances from single cell genomics and pathomics
      data with Optimal Transport (PILOT)
  source_content_hash: sha256:d0a6005ef7572e77b3086c668e29e1d9fdae23c1e6c77400a9de44ee0a02dcf5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CostaLab/PILOT
  tools:
  - Monocle 2
  - Multi-scale PHATE
  - PILOT
  - POT library
  - PhEMD
  - SCANCell
  - Seurat
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# multi-method-cross-dataset-benchmark-analysis

## Summary

Benchmark PILOT against competing sample-level methods (PhEMD, SCANCell/PhEMD embeddings, Multi-scale PHATE, and cell-proportion baselines) on 12 public single-cell and pathomics datasets, scoring patient-level clustering (ARI, Silhouette) and trajectory inference (AUCPR, Spearman) with Friedman–Nemenyi ranking statistics.

## When to use

Use when you must objectively compare multiple patient-level single-cell/pathomics analysis methods across several public datasets, requiring per-method clustering and trajectory metrics plus statistically corrected method rankings.

## When NOT to use

- You need to analyze a single dataset with a single method (e.g., PILOT trajectory estimation on one cohort) rather than statistically compare competing methods across many datasets
- Your patient samples have reliable labels and you want supervised patient classification — the compared methods (Multi-scale PHATE, SCANCell) require patient labels, whereas this benchmark targets unsupervised sample-level analysis
- Your data mixes distinct technologies (single nucleus vs. single cell) or isolation methods (droplet vs. well based), which the article states is an open challenge invalidating uniform comparison

## Inputs

- 12 pre-processed public single-cell datasets (.h5ad, e.g. GSE174188_CLUES1_adjusted.h5ad.gz) and pathomics structure datasets (e.g. Kidney_IgAN_T.h5ad)
- Disease-status labels per patient/sample for ARI and AUCPR evaluation
- Known disease-severity ordering per patient for Spearman correlation
- Method parameterizations: PILOT (POT emd2), PhEMD (Monocle 2 pipeline), SCANCell, Multi-scale PHATE

## Outputs

- benchmark_metrics.csv with ARI, Silhouette, AUCPR, and Spearman per method per dataset (4 methods x 12 datasets)
- Friedman–Nemenyi method-ranking tables reproducing Appendix Tables S1–S2
- Four benchmark panels corresponding to Fig. 2A–D

## How to apply

Download the twelve benchmark datasets (GSE174188, GSE158055, Zenodo 7227571, GSE211799, EGAS00001006052, Zenodo 7435911, GSE169285, GSA CRA001160, and kidney pathomics sets) and apply a uniform pre-processing pipeline — Seurat for normalizing and clustering pathomics structures, Monocle 2 for the PhEMD input pipeline — keeping dataset-specific filters (e.g., kidney-biopsy-only samples; 10 and 2 main PCA components for glomeruli and tubules). Run all four methods on each dataset: PILOT via POT's emd2 Wasserstein distances between cluster distributions, PhEMD with EMD-based specimen distances, SCANCell via PhEMD non-linear embeddings with association networks, and Multi-scale PHATE with multi-resolution clustering. For clustering evaluation, cluster samples from each method's distance matrix with Leiden and compute the Adjusted Rand Index against disease labels plus the Silhouette score of the sample distance matrix. For trajectory evaluation, estimate trajectories per method (diffusion maps followed by EIPLGraph backbone) and compute AUCPR of disease-status prediction along the trajectory and the Spearman correlation of trajectory position with known disease severity. Aggregate all metrics across the 12 datasets and apply the Friedman test with Nemenyi post-hoc correction to produce method rankings, writing the full metric and ranking table to benchmark_metrics.csv.

## Related tools

- **PILOT** (Method under test; computes Wasserstein distances between patient cluster distributions via optimal transport (emd2) for clustering and trajectory benchmarking) — https://github.com/CostaLab/PILOT
- **PhEMD** (Competing method; EMD-based specimen distances with Monocle 2 for normalization, dimension reduction, and clustering of cells)
- **SCANCell** (Competing method; uses PhEMD non-linear embeddings followed by association networks across cell clusters)
- **Multi-scale PHATE** (Competing method; non-linear embeddings and multi-resolution clustering of cells at distinct resolutions)
- **POT library** (Provides emd2 function implementing the Wasserstein-distance solver used by PILOT)
- **Seurat** (Uniform pre-processing pipeline for normalization and clustering of pathomics structures)
- **Monocle 2** (Pre-processing (normalization, dimension reduction, clustering) required for the PhEMD input pipeline)

## Examples

```
import pilotpy as pl; pl.pl.select_best_sil(adata, start = 0.2)
```

## Evaluation signals

- The Friedman–Nemenyi analysis must reproduce the reported method ordering in Fig. 2A–D and Appendix Tables S1–S2, with each metric (ARI, Silhouette, AUCPR, Spearman) computed for all four methods on all twelve datasets
- PILOT must obtain the highest average ARI score with a significantly superior ranking over all competing methods at p-value < 0.05 (Friedman–Nemenyi, Appendix Table S1)
- benchmark_metrics.csv must be complete: no missing cells across 4 methods x 12 datasets x 4 metrics
- Dataset-specific filters must be verifiable, e.g. kidney-biopsy-only restriction for pathomics sets and 10 vs. 2 PCA components retained for glomeruli and tubules
- Low-quality/outlier samples must be excluded per the article's filter rules (e.g. myocardial infarction sample with <1000 genes on average per cell; four pancreas embryos E12–E15 as trajectory outliers)

## Limitations

- None of the prior competing approaches (Chen 2020, Flores 2023, Kuchroo 2022, Zhang 2022) performed any benchmarking, so this cross-dataset protocol is the reference comparison
- The root sample for EIPLGraph trajectories was visually selected to reflect control-sample regions rather than chosen by an objective rule, a reproducibility gap for the trajectory metrics
- Comparison across distinct technologies (single nucleus vs. single cell) or isolation methods (droplet vs. well based) remains an open challenge; PILOT does not explicitly correct for technical artefacts, it only provides statistical tests to evaluate their influence

## Evidence

- [intro] benchmark scope: "we perform a benchmarking study to compare PILOT and competing approaches on their performance in 12 public single-cell and pathomics data sets"
- [supplementary] PILOT benchmark result: [evidence span withheld — anchor status: not_found]
- [discussion] PILOT implementation basis: "PILOT is based on the emd2 function from POT library (Flamary et al, 2021), which implements the best solver described in (Bonneel et al, 2011)."
- [readme] core distance computation under evaluation: "PILOT uses optimal transport to compute the Wasserstein distance between two individual single-cell samples."
