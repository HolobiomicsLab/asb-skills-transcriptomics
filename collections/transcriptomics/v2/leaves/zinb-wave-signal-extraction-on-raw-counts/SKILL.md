---
name: zinb-wave-signal-extraction-on-raw-counts
description: Use when scRNA-seq raw count matrices need low-dimensional signal extraction
  or dimensionality reduction, and preliminary normalization (TC, TMM, FQ) is undesirable
  or unknown.
when_to_use_negative:
- When unwanted technical variation affects the data and is not accounted for in the
  model or prior normalization — the inferred W will capture confounding effects,
  so the extracted signal is not guaranteed to be biologically relevant
- When supervised differential expression is the goal — this capability is not yet
  available and is only envisioned for a future zinbwave version
- When computational cost is prohibitive — ZINB-WaVE requires numerical optimization
  of a non-convex likelihood, unlike PCA
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: ZINB-WaVE
  role: Zero-inflated negative binomial model applied to raw counts for signal extraction
    and low-dimensional representation
  repo: https://github.com/drisso/zinbwave
- name: zinbwave
  role: Bioconductor R implementation of ZINB-WaVE; installed via BiocManager::install("zinbwave"),
    requires R >=3.4 and Bioconductor >=3.6
  repo: https://bioconductor.org/packages/zinbwave
- name: ZIFA
  role: Zero-inflated factor analysis baseline compared against ZINB-WaVE representations
    in silhouette-width benchmarks
  repo: https://github.com/epierson9/ZIFA
- name: edgeR
  role: Provides TMM normalization for the normalization-method comparison (RAW, TC,
    TMM, FQ) against ZINB-WaVE on raw counts
- name: scater
  role: Used to explore correlation between ZINB-WaVE estimated latent factors and
    known QC measures
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-017-02554-5
    title: A general and flexible method for signal extraction from single-cell RNA-seq
      data
  source_content_hash: sha256:477db023a0e91685c2e160d1e4e1439aa480793343e1bd7cc9f65a664f130c27
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/drisso/zinbwave
  tools:
  - BiocManager
  - Bioconductor (>=3.6)
  - R (>=3.4)
  - ZIFA
  - ZINB-WaVE
  - edgeR
  - scater
  - zinbwave
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# zinb-wave-signal-extraction-on-raw-counts

## Summary

Apply ZINB-WaVE, a zero-inflated negative binomial factor model, directly to unnormalized scRNA-seq count matrices to obtain low-dimensional representations that account for dropouts, over-dispersion, and count structure without prior normalization.

## When to use

Use when scRNA-seq raw count matrices need low-dimensional signal extraction or dimensionality reduction, and preliminary normalization (TC, TMM, FQ) is undesirable or unknown.

## When NOT to use

- When unwanted technical variation affects the data and is not accounted for in the model or prior normalization — the inferred W will capture confounding effects, so the extracted signal is not guaranteed to be biologically relevant
- When supervised differential expression is the goal — this capability is not yet available and is only envisioned for a future zinbwave version
- When computational cost is prohibitive — ZINB-WaVE requires numerical optimization of a non-convex likelihood, unlike PCA

## Inputs

- Unnormalized scRNA-seq raw count matrix (genes x cells), e.g., from the V1, S1/CA1, glioblastoma, or mESC data sets in drisso/zinb_analysis
- Optional gene-level and sample-level covariate matrices (e.g., a column of ones, QC measures, batch indicators)
- Author-provided cell-type labels for downstream silhouette-width evaluation

## Outputs

- Low-dimensional matrix W of inferred latent factors per cell
- Fitted ZINB model parameters (coefficients for zero-inflation and count components)
- Silhouette-width tabulations when used for normalization-method benchmarking (silhouette_widths.csv)

## How to apply

Install zinbwave via BiocManager::install("zinbwave") under R (>=3.4) and Bioconductor (>=3.6); to exactly reproduce the paper's analyses, pin version 0.1.1 from drisso/archive-zinbwave@v0.1.1. Load raw scRNA-seq count matrices (e.g., the V1, S1/CA1, glioblastoma, and mESC data sets from drisso/zinb_analysis) and run ZINB-WaVE directly on the unnormalized counts: the model includes a sample-level intercept that acts as a global-scaling normalization factor (enabled by a column of ones in the gene-level covariate matrix), so raw counts can be modeled with no prior normalization. Extract the inferred low-dimensional matrix W (latent factors) and use it for clustering, t-SNE visualization, or silhouette-width benchmarking against PCA and ZIFA. Judge success by comparing average silhouette widths of author-provided cell-type labels (computed on Euclidean distances) against PCA/ZIFA applied to the best-performing normalization; ZINB-WaVE on raw counts should be comparable to the best normalization-based representation.

## Related tools

- **ZINB-WaVE** (Zero-inflated negative binomial model applied to raw counts for signal extraction and low-dimensional representation) — https://github.com/drisso/zinbwave
- **zinbwave** (Bioconductor R implementation of ZINB-WaVE; installed via BiocManager::install("zinbwave"), requires R >=3.4 and Bioconductor >=3.6) — https://bioconductor.org/packages/zinbwave
- **ZIFA** (Zero-inflated factor analysis baseline compared against ZINB-WaVE representations in silhouette-width benchmarks) — https://github.com/epierson9/ZIFA
- **edgeR** (Provides TMM normalization for the normalization-method comparison (RAW, TC, TMM, FQ) against ZINB-WaVE on raw counts)
- **scater** (Used to explore correlation between ZINB-WaVE estimated latent factors and known QC measures)

## Examples

```
BiocManager::install("zinbwave"); library(zinbwave); zinb <- zinbFit(counts, K = 2, X = model.matrix(~1, data = colData(counts)))
```

## Evaluation signals

- silhouette_widths.csv exists with one row per data set per normalization per method, and ZINB-WaVE on raw counts shows average silhouette widths comparable to PCA/ZIFA with the best-performing normalization
- Relative silhouette ordering reproduces the paper's pattern: normalization choice matters more than PCA-vs-ZIFA method choice
- zinbwave version is 0.1.1 (via drisso/archive-zinbwave@v0.1.1) when exactly reproducing the paper's figures
- Estimated latent factors show low correlation with known QC measures, or QC measures are included as covariates if highly correlated
- Latent-factor dimensionality is specified per the paper's recommendations; runtime is approximately linear in cells and genes and quadratic in latent factors

## Limitations

- If unwanted technical variation is unaccounted for, the low-rank matrix W captures confounding effects and the extracted signal is not guaranteed to be biologically relevant
- Computational cost: non-convex likelihood optimization makes ZINB-WaVE slower than PCA
- Supervised differential expression capability is not yet available; only envisioned for a future zinbwave version
- No compelling examples were found where adding a gene-level covariate improves signal extraction

## Evidence

- [abstract] ZINB-WaVE beats competitors without normalization: "the model and its associated estimation procedure are able to give a more stable and accurate low-dimensional representation of the data than principal component analysis (PCA) and zero-inflated"
- [intro] Sample-level intercept as normalization: "includes a sample-level intercept, which serves as a global-scaling normalization factor, and gives the user the ability to include both gene-level and sample-level covariates"
- [methods] zinbwave installation and requirements: "BiocManager::install("zinbwave")"
- [methods] zinbwave version requirements: "Note that `zinbwave` requires R (>=3.4) and Bioconductor (>=3.6)."
- [discussion] Raw counts without prior normalization: "the corresponding cell-level intercept acts as a global-scaling normalization factor, allowing the modeling of raw count data, with no need for prior normalization"
- [readme] zinbwave 0.1.1 for exact reproduction: [evidence span withheld — anchor status: external_doc]
- [discussion] Compute scaling: "the computing time was approximately linear in both the number of cells and the number of genes, and approximately quadratic in the number of latent factors"
