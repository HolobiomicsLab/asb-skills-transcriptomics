---
name: single-cell-count-matrix-processing
description: Use when a raw scRNA-seq count matrix (e.g., GSE57872 glioblastoma cells
  with batch and patient labels) must be modeled without preliminary normalization,
  especially when batch and biology are partially confounded and covariates like detection
  rate must be included.
when_to_use_negative:
- When the data have already been normalized with a method that removes the global-scaling
  signal — the cell-level intercept in ZINB-WaVE is designed to model raw counts with
  no need for prior normalization
- When the requested adjustment is a batch indicator fully confounded with the biological
  variable of interest (each patient processed in a single batch) — this covariate
  model fails to preserve patient effects
- When the end goal is supervised differential expression — the article notes this
  capability is only envisioned for a future zinbwave version
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: zinbwave
  role: R/Bioconductor package that implements the zero-inflated negative binomial
    model used to fit raw counts and extract latent factors W
  repo: https://bioconductor.org/packages/zinbwave
- name: ZINB-WaVE
  role: The model itself (zero-inflated negative binomial based wanted variation extraction)
    with gene-level and sample-level covariate support
  repo: https://github.com/drisso/zinbwave
- name: scater
  role: Used to explore correlation between estimated latent factors and QC measures
    (e.g., Fig. 2f)
- name: ZIFA
  role: Zero-inflated factor analysis baseline method; requires log2(1 + count_data)
    transformation of count data prior to use
  repo: https://github.com/epierson9/ZIFA
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-017-02554-5
    title: A general and flexible method for signal extraction from single-cell RNA-seq
      data
  source_content_hash: sha256:cab24d281e853e59117caa79d730074dd4c9e1a5268191f1b9dba03d53b95f87
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiocManager
  - Bioconductor (>=3.6)
  - R (>=3.4)
  - ZIFA
  - ZINB-WaVE
  - scater
  - zinbwave
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# single-cell-count-matrix-processing

## Summary

Process raw single-cell RNA-seq count matrices by fitting the ZINB-WaVE zero-inflated negative binomial model, which extracts a low-dimensional representation (W) that accounts for dropouts, over-dispersion, and count data, using sample-level covariates such as the detection rate to handle unwanted variation.

## When to use

Use when a raw scRNA-seq count matrix (e.g., GSE57872 glioblastoma cells with batch and patient labels) must be modeled without preliminary normalization, especially when batch and biology are partially confounded and covariates like detection rate must be included.

## When NOT to use

- When the data have already been normalized with a method that removes the global-scaling signal — the cell-level intercept in ZINB-WaVE is designed to model raw counts with no need for prior normalization
- When the requested adjustment is a batch indicator fully confounded with the biological variable of interest (each patient processed in a single batch) — this covariate model fails to preserve patient effects
- When the end goal is supervised differential expression — the article notes this capability is only envisioned for a future zinbwave version

## Inputs

- Raw scRNA-seq count matrix (genes x cells), e.g., from GEO accession GSE57872
- Sample/cell metadata with batch and patient labels
- Sample-level covariate matrix X (intercept + detection-rate covariate)
- Number of latent factors K

## Outputs

- Estimated low-dimensional matrix W of latent factors from the zinbwave fit
- 2D PCA projection of W colored by batch and by patient (e.g., patel_zinbwave_pca.png)
- Contrast W from a batch-indicator covariate fit for qualitative comparison

## How to apply

Set up R (>=3.4) with Bioconductor (>=3.6) and install zinbwave via BiocManager, then load the raw count matrix and sample metadata (batch, patient). Compute the sample-level detection rate (proportion of genes with non-zero counts per cell) and supply it, alongside an intercept column, as the sample-level covariate matrix X; because a column of ones is included in the gene-level covariate matrix, the cell-level intercept acts as a global-scaling normalization factor so raw counts can be modeled directly. Fit the ZINB-WaVE model with K unobserved latent factors so W captures only variation not explained by the detection-rate covariate. As a decision contrast, refit with a batch-indicator covariate (confounded with patient) and compare the low-dimensional spaces via PCA colored by batch and by patient: the detection-rate fit should mix batches while separating patients, whereas the confounded fit should fail to separate patients. Distances derived from the ZINB-WaVE low-dimensional space can also be used for t-SNE visualization, and latent factors can be correlated with QC measures using scater.

## Related tools

- **zinbwave** (R/Bioconductor package that implements the zero-inflated negative binomial model used to fit raw counts and extract latent factors W) — https://bioconductor.org/packages/zinbwave
- **ZINB-WaVE** (The model itself (zero-inflated negative binomial based wanted variation extraction) with gene-level and sample-level covariate support) — https://github.com/drisso/zinbwave
- **scater** (Used to explore correlation between estimated latent factors and QC measures (e.g., Fig. 2f))
- **ZIFA** (Zero-inflated factor analysis baseline method; requires log2(1 + count_data) transformation of count data prior to use) — https://github.com/epierson9/ZIFA

## Examples

```
BiocManager::install("zinbwave")
```

## Evaluation signals

- In the primary fit's low-dimensional space, cells from the two MGH26 batches are well mixed while cells cluster by patient
- The confounded batch-indicator covariate model's W fails to separate patients, confirming the detection-rate covariate is the correct choice
- The model was fit to raw counts with a column of ones in the gene-level covariate matrix, so no prior normalization step was applied
- Low-dimensional representation is compared against PCA and ZIFA baselines for stability and accuracy
- The qualitative pattern is confirmable in the saved figure patel_zinbwave_pca.png

## Limitations

- There is no guarantee that the low-dimensional signal extracted by ZINB-WaVE is biologically relevant; if unwanted technical variation is unaccounted for, W will capture such confounding effects
- Performance comes at a computational cost because a non-convex likelihood must be numerically optimized; computing time is approximately linear in cells and genes and quadratic in latent factors
- Supervised differential expression is not yet available in the package
- ZIFA, a baseline for this workflow, requires a log2(1 + count_data) transform of count data, unlike ZINB-WaVE

## Evidence

- [abstract] ZINB-WaVE model abstract framing: "we present a general and flexible zero-inflated negative binomial model (ZINB-WaVE)"
- [discussion] Global-scaling normalization via cell-level intercept: "the corresponding cell-level intercept acts as a global-scaling normalization factor, allowing the modeling of raw count data"
- [intro] Sample-level covariates enable normalization for batch effects: "includes a sample-level intercept, which serves as a global-scaling normalization factor, and gives the user the ability to include both gene-level and sample-level covariates"
- [methods] Software requirements: "Note that `zinbwave` requires R (>=3.4) and Bioconductor (>=3.6)."
- [readme] ZIFA count-data preprocessing requirement: [evidence span withheld — anchor status: external_doc]
- [discussion] No guarantee of biological relevance of W: "there is no guarantee that the low-dimensional signal extracted by ZINB-WaVE is biologically relevant"
