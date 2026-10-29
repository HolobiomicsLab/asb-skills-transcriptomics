---
name: batch-effect-assessment
description: Use when single-cell RNA-seq raw count data have technical batches that
  are partially confounded with biology (e.g., each patient/biological replicate processed
  in a separate batch) and you must remove batch effects without erasing the biological
  signal.
when_to_use_negative:
- When batch and biology are completely confounded and no unconfounded technical covariate
  (such as detection rate) is available — the batch-indicator covariate model failed
  to preserve patient separation in this exact setting
- When the goal is supervised differential expression accounting for zero inflation
  — the article notes this capability is not yet available and only envisioned for
  a future zinbwave version
- When the input is already normalized/transformed expression rather than raw counts
  — zinbwave is designed to model raw count data directly via its global-scaling intercept
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: zinbwave
  role: R/Bioconductor package that fits the zero-inflated negative binomial model
    with sample-level covariates (detection rate or batch indicator) and returns the
    latent factors W used for batch-effect assessment
  repo: https://bioconductor.org/packages/zinbwave
- name: scater
  role: Used to explore the correlation between estimated ZINB-WaVE latent factors
    and known quality-control measures (see Fig. 2f), informing which QC covariates
    to include
- name: R (>=3.4) / Bioconductor (>=3.6)
  role: Execution environment requirement for running zinbwave
- name: drisso/zinb_analysis
  role: Repository with data analysis and simulation code to reproduce the results
    and figures of the ZINB-WaVE paper
  repo: https://github.com/drisso/zinb_analysis
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-017-02554-5
    title: A general and flexible method for signal extraction from single-cell RNA-seq
      data
  source_content_hash: sha256:6325c5624ecaa39784d1b77f83e41d8352fec153bb372e71d8aebc5f779cd4cb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiocManager
  - Bioconductor (>=3.6)
  - R (>=3.4)
  - R (>=3.4) / Bioconductor (>=3.6)
  - drisso/zinb_analysis
  - scater
  - zinbwave
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# batch-effect-assessment

## Summary

Assess and remove batch effects in single-cell RNA-seq by fitting a ZINB-WaVE zero-inflated negative binomial model in which unwanted variation (e.g., detection rate) enters as a sample-level covariate, so the latent factors W capture only unexplained variation. This enables judging, via PCA of W colored by batch and by biological condition, whether batches mix while biology (e.g., patient effects) is preserved.

## When to use

Use when single-cell RNA-seq raw count data have technical batches that are partially confounded with biology (e.g., each patient/biological replicate processed in a separate batch) and you must remove batch effects without erasing the biological signal.

## When NOT to use

- When batch and biology are completely confounded and no unconfounded technical covariate (such as detection rate) is available — the batch-indicator covariate model failed to preserve patient separation in this exact setting
- When the goal is supervised differential expression accounting for zero inflation — the article notes this capability is not yet available and only envisioned for a future zinbwave version
- When the input is already normalized/transformed expression rather than raw counts — zinbwave is designed to model raw count data directly via its global-scaling intercept

## Inputs

- Raw (unnormalized) single-cell RNA-seq count matrix (genes x cells), e.g., GSE57872 glioblastoma counts
- Sample/cell metadata with batch labels and biological condition labels (e.g., patient of origin)
- Sample-level detection-rate covariate (proportion of non-zero genes per cell) supplied in the X covariate matrix alongside an intercept column

## Outputs

- Estimated low-dimensional factors W from the zinbwave fit (one column per latent factor K)
- 2D PCA of W with cells colored by batch and by patient (e.g., patel_zinbwave_pca.png)
- Contrast fit: W from the confounded batch-indicator covariate model for comparison

## How to apply

Fit the ZINB-WaVE model (zinbwave R package, requiring R >=3.4 and Bioconductor >=3.6, installed via BiocManager) directly on the raw count matrix — no prior normalization is needed because a column of ones in the gene-level covariate matrix makes the cell-level intercept a global-scaling normalization factor. Compute the sample-level detection rate (proportion of genes with non-zero counts per cell) and supply it, alongside the intercept, in the sample-level covariate matrix X so the unobserved factors W capture only variation not explained by it; this was the strategy that removed the batch effect while preserving patient differences in the glioblastoma data (GSE57872), unlike a model using the batch indicator itself as a covariate (which is confounded with patient). Extract the estimated W, project cells into 2D by PCA, and color cells separately by batch and by biological label to inspect batch mixing versus condition separation. As a decision point, also explore the correlation between estimated latent factors and QC measures (e.g., with scater) and include such QC measures as covariates when highly correlated with the factors. The key rationale: any unwanted technical variation not included in the model (or in prior normalization) will be absorbed into W and contaminate the low-dimensional signal.

## Related tools

- **zinbwave** (R/Bioconductor package that fits the zero-inflated negative binomial model with sample-level covariates (detection rate or batch indicator) and returns the latent factors W used for batch-effect assessment) — https://bioconductor.org/packages/zinbwave
- **scater** (Used to explore the correlation between estimated ZINB-WaVE latent factors and known quality-control measures (see Fig. 2f), informing which QC covariates to include)
- **R (>=3.4) / Bioconductor (>=3.6)** (Execution environment requirement for running zinbwave)
- **drisso/zinb_analysis** (Repository with data analysis and simulation code to reproduce the results and figures of the ZINB-WaVE paper) — https://github.com/drisso/zinb_analysis

## Examples

```
BiocManager::install("zinbwave")
```

## Evaluation signals

- PCA of the primary fit's W shows cells well mixed across batches while cells cluster by patient of origin
- The confounded batch-indicator covariate model's W fails to separate patients — used as a negative control
- Latent factors show low correlation with QC measures such as detection rate after inclusion of the detection-rate covariate in X (check via scater)
- The qualitative batch-mixing/patient-separation pattern matches the saved figure patel_zinbwave_pca.png
- Environment check: zinbwave installed from Bioconductor under R >=3.4 and Bioconductor >=3.6

## Limitations

- If unwanted technical variation affects the data and is not accounted for in the model (or in prior normalization), the low-rank matrix W inferred by ZINB-WaVE will capture such confounding effects
- There is no guarantee that the low-dimensional signal extracted by ZINB-WaVE is biologically relevant
- Optimization of the non-convex likelihood carries a computational cost, though computing time is approximately linear in the number of cells and genes and approximately quadratic in the number of latent factors
- The article did not find compelling examples in which adding a gene-level covariate improves signal extraction

## Evidence

- [abstract] Detection-rate covariate removes batch while preserving biology: "the model and its associated estimation procedure are able to give a more stable and accurate low-dimensional representation of the data than principal component analysis (PCA)"
- [intro] Sample-level covariates and scaling intercept support raw-count fitting: "includes a sample-level intercept, which serves as a global-scaling normalization factor, and gives the user the ability to include both gene-level and sample-level covariates"
- [discussion] Unmodeled technical variation contaminates W: "If unwanted technical variation affects the data and is not accounted for in the model (or in prior normalization), the low-rank matrix W inferred by ZINB-WaVE will capture such confounding effects."
- [discussion] Correlate latent factors with QC measures: "explore the correlation between the latent factors estimated by our procedure and known measures of quality control"
- [methods] Version requirement for zinbwave: "Note that `zinbwave` requires R (>=3.4) and Bioconductor (>=3.6)."
- [readme] Reproducibility repository for the paper's figures: [evidence span withheld — anchor status: external_doc]
