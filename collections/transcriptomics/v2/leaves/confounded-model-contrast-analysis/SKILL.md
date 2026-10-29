---
name: confounded-model-contrast-analysis
description: Use when batch and biology are partially confounded (e.g., each patient
  processed in a separate batch) and you need to show whether latent factors extracted
  by ZINB-WaVE capture unwanted technical variation or the biological signal of interest.
when_to_use_negative:
- Batch and biology are fully crossed (not confounded) — a standard batch-covariate
  fit directly removes the batch effect and the confounded contrast model is unnecessary.
- The goal is quantitative DE analysis rather than diagnosing what the latent factors
  W captured — ZINB-WaVE supervised differential expression was not yet available,
  only envisioned for a future version.
- The low-dimensional signal from W is being interpreted as biologically relevant
  without checking covariates — there is no guarantee the extracted signal is biologically
  relevant.
tools:
- name: zinbwave
  role: Fits the zero-inflated negative binomial model with sample-level covariate
    matrix X (batch indicator for the contrast fit; detection rate for the primary
    fit) and returns the estimated latent factors W
  repo: https://bioconductor.org/packages/zinbwave
- name: R (>=3.4)
  role: Execution environment required by zinbwave
- name: Bioconductor (>=3.6)
  role: Required platform for installing zinbwave via BiocManager
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-017-02554-5
    title: A general and flexible method for signal extraction from single-cell RNA-seq
      data
  source_content_hash: sha256:cc7f9d9611c4a7d51598adae23fe32525f6c95e2ad0fd1e158321594f970b70b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiocManager
  - Bioconductor (>=3.6)
  - R (>=3.4)
  - zinbwave
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# confounded-model-contrast-analysis

## Summary

Fit a deliberately confounded ZINB-WaVE model (sample-level covariate matrix encoding the batch indicator, which is confounded with patient/biology) and contrast its estimated W matrix against the detection-rate-covariate fit, to demonstrate whether the low-dimensional factors capture technical batch effects instead of biology.

## When to use

Use when batch and biology are partially confounded (e.g., each patient processed in a separate batch) and you need to show whether latent factors extracted by ZINB-WaVE capture unwanted technical variation or the biological signal of interest.

## When NOT to use

- Batch and biology are fully crossed (not confounded) — a standard batch-covariate fit directly removes the batch effect and the confounded contrast model is unnecessary.
- The goal is quantitative DE analysis rather than diagnosing what the latent factors W captured — ZINB-WaVE supervised differential expression was not yet available, only envisioned for a future version.
- The low-dimensional signal from W is being interpreted as biologically relevant without checking covariates — there is no guarantee the extracted signal is biologically relevant.

## Inputs

- Raw (unnormalized) single-cell RNA-seq count matrix (genes x cells)
- Sample metadata with batch labels (e.g., the two MGH26 batches in GSE57872)
- Sample metadata with patient labels
- Sample-level covariate matrices: batch indicator (contrast fit) and per-cell detection rate, i.e., proportion of genes with non-zero counts (primary fit)

## Outputs

- Estimated W matrix (low-dimensional factors) from the confounded batch-indicator model
- Estimated W matrix from the detection-rate-covariate model
- 2D PCA projections of each W colored by batch and by patient (e.g., patel_zinbwave_pca.png)
- Qualitative verdict on batch mixing vs. patient separation for each fit

## How to apply

Build two ZINB-WaVE fits on the same raw count matrix (e.g., GSE57872 glioblastoma data, exploiting the two MGH26 batches): the contrast fit supplies X with the batch indicator (confounded with patient), while the primary fit supplies X with the sample-level detection-rate covariate (proportion of genes with non-zero counts per cell) alongside the intercept, so the cell-level intercept acts as a global-scaling normalization factor on raw counts. Fit both models with K unobserved latent factors and extract the low-rank matrix W from each. Project each W into 2D via PCA and color cells separately by batch and by patient. Judge success by comparing the two: the detection-rate fit should show well-mixed batches with cells clustering by patient, while the batch-indicator fit's W should fail to separate patients, revealing that its factors captured the confounded technical effect.

## Related tools

- **zinbwave** (Fits the zero-inflated negative binomial model with sample-level covariate matrix X (batch indicator for the contrast fit; detection rate for the primary fit) and returns the estimated latent factors W) — https://bioconductor.org/packages/zinbwave
- **R (>=3.4)** (Execution environment required by zinbwave)
- **Bioconductor (>=3.6)** (Required platform for installing zinbwave via BiocManager)

## Examples

```
BiocManager::install("zinbwave")
```

## Evaluation signals

- In the primary (detection-rate) fit's PCA on W, batches should appear well mixed while cells cluster by patient; confirm this in the saved figure (e.g., patel_zinbwave_pca.png).
- In the confounded (batch-indicator) fit's PCA on W, cells should fail to separate by patient, showing the factors absorbed the batch-confounded structure.
- The contrast fit must be run on the same raw count matrix as the primary fit so the two W matrices are directly comparable.
- The primary fit's X must contain the per-cell detection rate (proportion of non-zero genes) plus intercept; the cell-level intercept should act as a global-scaling normalization factor on raw counts.
- Version requirements: zinbwave with R (>=3.4) and Bioconductor (>=3.6); the paper's analyses exactly used zinbwave version 0.1.1 (install_github("drisso/archive-zinbwave@v0.1.1")).

## Limitations

- If unwanted technical variation is not accounted for in the model (or prior normalization), W will capture such confounding effects — exactly what the batch-indicator contrast fit demonstrates.
- There is no guarantee that the low-dimensional signal extracted by ZINB-WaVE is biologically relevant; the contrast analysis is a qualitative diagnostic, not a formal test.
- Computational cost: fitting requires numerical optimization of a non-convex likelihood, with time approximately linear in cells and genes and approximately quadratic in the number of latent factors.

## Evidence

- [discussion] Unaccounted technical variation is absorbed by W: "If unwanted technical variation affects the data and is not accounted for in the model (or in prior normalization), the low-rank matrix W inferred by ZINB-WaVE will capture such confounding effects."
- [intro] Sample-level covariates enable batch adjustment: "includes a sample-level intercept, which serves as a global-scaling normalization factor, and gives the user the ability to include both gene-level and sample-level covariates"
- [discussion] Cell-level intercept enables raw count modeling: "the corresponding cell-level intercept acts as a global-scaling normalization factor, allowing the modeling of raw count data, with no need for prior normalization"
- [discussion] No guarantee of biological relevance: "However, there is no guarantee that the low-dimensional signal extracted by ZINB-WaVE is biologically relevant"
- [results] Confounded fit fails to separate patients: [evidence span withheld — anchor status: not_found]
