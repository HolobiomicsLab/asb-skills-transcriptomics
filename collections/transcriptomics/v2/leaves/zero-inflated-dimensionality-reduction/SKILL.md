---
name: zero-inflated-dimensionality-reduction
description: Use when scRNA-seq count data show dropout-driven zero inflation and
  PCA components appear driven by technical effects (sequencing depth, detection rate),
  and you need a low-dimensional representation reflecting biology.
when_to_use_negative:
- If unwanted technical variation affects the data and is neither modeled as sample-level
  covariates nor removed by prior normalization, the inferred W may capture confounding
  effects rather than biological signal
- 'When a supervised differential-expression analysis is required: this capability
  is not yet available in zinbwave and is only envisioned for a future version'
- 'When computational cost is prohibitive: ZINB-WaVE requires non-convex likelihood
  optimization, unlike PCA'
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: ZINB-WaVE
  role: Zero-inflated negative binomial model fitted to counts to extract wanted low-dimensional
    variation
  repo: https://github.com/drisso/zinbwave
- name: zinbwave
  role: Bioconductor R implementation of ZINB-WaVE used to infer the low-dimensional
    factors; install via BiocManager::install("zinbwave")
  repo: https://bioconductor.org/packages/zinbwave
- name: ZIFA
  role: Zero-inflated factor analysis baseline for dimensionality reduction; applied
    to log2(1+count) transformed data
  repo: https://github.com/epierson9/ZIFA
- name: scRNAseq
  role: Bioconductor package used to load the Tasic V1 mouse visual cortex dataset
- name: scater
  role: Used to compute/explore correlation of latent factors with QC measures
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-017-02554-5
    title: A general and flexible method for signal extraction from single-cell RNA-seq
      data
  source_content_hash: sha256:481cd4a58431fcbd0b283abf48f6eec14a83dfcf99a4fda13d5f207b51620a98
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/drisso/zinbwave
  tools:
  - Bioconductor (>=3.6)
  - R (>=3.4)
  - ZIFA
  - ZINB-WaVE
  - scRNAseq
  - scater
  - zinbwave
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# zero-inflated-dimensionality-reduction

## Summary

Extract low-dimensional representations from scRNA-seq count data with ZINB-WaVE, a zero-inflated negative binomial model that accounts for dropouts, over-dispersion, and count nature, and benchmark it against PCA and ZIFA using silhouette widths and QC-measure correlations.

## When to use

Use when scRNA-seq count data show dropout-driven zero inflation and PCA components appear driven by technical effects (sequencing depth, detection rate), and you need a low-dimensional representation reflecting biology.

## When NOT to use

- If unwanted technical variation affects the data and is neither modeled as sample-level covariates nor removed by prior normalization, the inferred W may capture confounding effects rather than biological signal
- When a supervised differential-expression analysis is required: this capability is not yet available in zinbwave and is only envisioned for a future version
- When computational cost is prohibitive: ZINB-WaVE requires non-convex likelihood optimization, unlike PCA

## Inputs

- Raw scRNA-seq gene-by-cell count matrix (e.g., Tasic V1 mouse visual cortex dataset from the scRNAseq Bioconductor package)
- Cell-level QC measures (detection rate, total reads per cell)
- Cell labels (e.g., layer-of-origin / cluster annotations) for silhouette evaluation

## Outputs

- First two low-dimensional components (W matrix) per method (ZINB-WaVE, PCA, ZIFA)
- Per-method per-cluster average silhouette widths
- Absolute correlations of the two components with detection rate and total reads
- Benchmark table (e.g., tasic_v1_benchmark_results.csv)

## How to apply

Load the raw count matrix (e.g., the Tasic V1 mouse visual cortex dataset via the scRNAseq Bioconductor package) and optionally subset to the ~1000 most variable genes. Fit ZINB-WaVE with the zinbwave package on the counts (or total-count normalized data), including a column of ones in the gene-level covariate matrix so the cell-level intercept acts as a global-scaling normalization factor and raw counts can be modeled; extract the first two latent dimensions (W). In parallel, run PCA on normalized counts and ZIFA on log2(1+count) data as the Python-based zero-inflated comparison. Evaluate each method's 2D embedding two ways: per-cluster average silhouette widths computed from cell distances against layer-of-origin labels, and absolute correlation of the two components with QC measures (detection rate and total reads, in the spirit of the scater QC workflow); tabulate results side by side (e.g., to tasic_v1_benchmark_results.csv). Validation criterion: ZINB-WaVE should show the highest average silhouette width and the lowest absolute correlations with detection rate and total reads among the three methods.

## Related tools

- **ZINB-WaVE** (Zero-inflated negative binomial model fitted to counts to extract wanted low-dimensional variation) — https://github.com/drisso/zinbwave
- **zinbwave** (Bioconductor R implementation of ZINB-WaVE used to infer the low-dimensional factors; install via BiocManager::install("zinbwave")) — https://bioconductor.org/packages/zinbwave
- **ZIFA** (Zero-inflated factor analysis baseline for dimensionality reduction; applied to log2(1+count) transformed data) — https://github.com/epierson9/ZIFA
- **scRNAseq** (Bioconductor package used to load the Tasic V1 mouse visual cortex dataset)
- **scater** (Used to compute/explore correlation of latent factors with QC measures)

## Examples

```
BiocManager::install("zinbwave")
```

## Evaluation signals

- ZINB-WaVE achieves the highest average silhouette width among ZINB-WaVE, PCA, and ZIFA when cells are grouped by layer of origin
- ZINB-WaVE's first two components have the lowest absolute correlations with detection rate and total reads among the three methods
- ZINB-WaVE clusters in the 2D embedding are tighter and biologically meaningful by cortical layer of origin
- Results table (tasic_v1_benchmark_results.csv) contains silhouette widths and QC correlations side by side for all three methods

## Limitations

- There is no guarantee that the low-dimensional signal extracted by ZINB-WaVE is biologically relevant, especially if unwanted technical variation is unaccounted for
- Better performance than PCA comes at a computational cost due to numerical optimization of a non-convex likelihood function
- zinbwave requires R (>=3.4) and Bioconductor (>=3.6); exact paper reproduction needs zinbwave version 0.1.1
- ZIFA README recommends taking log2(1 + count_data) before applying ZIFA to count data

## Evidence

- [abstract] the model and its associated estimation procedure are able to give a more stable and accurate low-dimensional representation of the data than principal component analysis (PCA) and zero-inflated: "the model and its associated estimation procedure are able to give a more stable and accurate low-dimensional representation of the data than principal component analysis (PCA) and zero-inflated"
- [intro] zinbwave available through Bioconductor: "The approach is implemented in the open-source R package zinbwave, publicly available through the Bioconductor Project"
- [methods] zinbwave requires R >=3.4 and Bioconductor >=3.6: "Note that `zinbwave` requires R (>=3.4) and Bioconductor (>=3.6)."
- [discussion] explore correlation between latent factors and QC measures: "explore the correlation between the latent factors estimated by our procedure and known measures of quality control"
- [discussion] unaccounted technical variation confounds W: "If unwanted technical variation affects the data and is not accounted for in the model (or in prior normalization), the low-rank matrix W inferred by ZINB-WaVE will capture such confounding effects."
- [readme] ZIFA README recommends log transform: [evidence span withheld — anchor status: external_doc]
