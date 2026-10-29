---
name: batch-covariate-signal-extraction
description: Use when raw multi-batch scRNA-seq count data (e.g., Kolodziejczyk mESC
  across culture media) must be decomposed so the low-dimensional representation clusters
  by a known biological variable while dependence on batch is measured and removed.
when_to_use_negative:
- Unwanted technical variation is already removed by prior normalization or by including
  QC measures as cell-level covariates, so a batch-covariate contrast adds no information
- The goal is supervised differential expression between batches — the article notes
  this capability is only envisioned for a future zinbwave version
- There are no known batch or biological group labels, so the batch/cluster silhouette
  comparison cannot be evaluated
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: zinbwave
  role: Fits the zero-inflated negative binomial model; its X' argument takes the
    intercept-only or batch-indicator sample-level covariate matrix and returns the
    low-dimensional W factors
  repo: https://github.com/drisso/zinbwave
- name: BiocManager
  role: Installs the zinbwave package in the R (>=3.4) / Bioconductor (>=3.6) environment
- name: scRNAseq
  role: Bioconductor package used to load the scRNA-seq count data and batch annotation
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-017-02554-5
    title: A general and flexible method for signal extraction from single-cell RNA-seq
      data
  source_content_hash: sha256:1db4927f7f8e21963d45dc7414ba54c08db8f6d324da0f2d1fb94428bd03207b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/drisso/zinbwave
  tools:
  - BiocManager
  - Bioconductor
  - R
  - scRNAseq
  - zinbwave
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# batch-covariate-signal-extraction

## Summary

Fit ZINB-WaVE's zero-inflated negative binomial model with and without batch indicator variables as sample-level covariates (X' matrix) on raw scRNA-seq counts, then compare cluster and batch average silhouette widths to verify the batch covariates remove batch dependence from the low-dimensional W factors.

## When to use

Use when raw multi-batch scRNA-seq count data (e.g., Kolodziejczyk mESC across culture media) must be decomposed so the low-dimensional representation clusters by a known biological variable while dependence on batch is measured and removed.

## When NOT to use

- Unwanted technical variation is already removed by prior normalization or by including QC measures as cell-level covariates, so a batch-covariate contrast adds no information
- The goal is supervised differential expression between batches — the article notes this capability is only envisioned for a future zinbwave version
- There are no known batch or biological group labels, so the batch/cluster silhouette comparison cannot be evaluated

## Inputs

- Raw scRNA-seq count matrix (e.g., Kolodziejczyk mESC, via the scRNAseq Bioconductor package)
- Sample/cell-level batch annotation (batch indicator variable)
- Known biological cluster labels (e.g., culture medium groups) for silhouette evaluation

## Outputs

- Low-dimensional W factors from the intercept-only ZINB-WaVE fit
- Low-dimensional W factors from the batch-covariate ZINB-WaVE fit
- silhouette_comparison.csv: table of average silhouette widths for biological clusters and batches under both fits

## How to apply

Load the raw count matrix and batch annotation (e.g., Kolodziejczyk mESC) into R (>=3.4) under Bioconductor (>=3.6) with zinbwave installed via BiocManager::install('zinbwave') — use zinbwave version 0.1.1 to exactly reproduce the paper's analyses. Fit a default ZINB-WaVE model whose sample-level covariate matrix X' contains only an intercept, so the intercept acts as a global-scaling normalization factor and raw counts are modeled without prior normalization while the unobserved sample-level covariates W are estimated freely (K latent factors, W capturing both wanted and unwanted variation). Fit a second model adding the batch variable as known sample-level covariates in X' so unwanted batch variation is regressed out of the estimated W factors. In both low-dimensional representations, compute average silhouette widths of the known biological clusters (culture medium) and of the batches. Assemble the comparison table: the correct result has a larger biological silhouette with batch covariates and a much larger batch silhouette in the intercept-only fit, demonstrating the default model leaves cells segregated by batch.

## Related tools

- **zinbwave** (Fits the zero-inflated negative binomial model; its X' argument takes the intercept-only or batch-indicator sample-level covariate matrix and returns the low-dimensional W factors) — https://github.com/drisso/zinbwave
- **BiocManager** (Installs the zinbwave package in the R (>=3.4) / Bioconductor (>=3.6) environment)
- **scRNAseq** (Bioconductor package used to load the scRNA-seq count data and batch annotation)

## Examples

```
library(zinbwave); fit_batch <- zinbwave(counts, X = model.matrix(~ batch, colData(counts)))
```

## Evaluation signals

- Cluster-level (culture medium) average silhouette width is larger in the batch-covariate fit than in the intercept-only fit
- Batch-level average silhouette width is much larger in the intercept-only fit than in the batch-covariate fit
- The comparison table is produced as silhouette_comparison.csv and reproduces the reported pattern
- Both fits were run on raw counts with no prior normalization, relying on the intercept as global-scaling normalization factor
- Environment matches R (>=3.4), Bioconductor (>=3.6), and zinbwave v0.1.1 for exact paper reproduction

## Limitations

- There is no guarantee that the low-dimensional signal extracted by ZINB-WaVE is biologically relevant
- If unwanted technical variation is not accounted for, the low-rank matrix W inferred by ZINB-WaVE will capture such confounding effects
- Estimation requires numerically optimizing a non-convex likelihood, which adds computational cost
- Exact paper reproduction requires the archived zinbwave v0.1.1 from GitHub, not the current Bioconductor release

## Evidence

- [intro] includes a sample-level intercept, which serves as a global-scaling normalization factor, and gives the user the ability to include both gene-level and sample-level covariates: "includes a sample-level intercept, which serves as a global-scaling normalization factor, and gives the user the ability to include both gene-level and sample-level covariates"
- [discussion] the corresponding cell-level intercept acts as a global-scaling normalization factor, allowing the modeling of raw count data, with no need for prior normalization: "the corresponding cell-level intercept acts as a global-scaling normalization factor, allowing the modeling of raw count data, with no need for prior normalization"
- [discussion] the low-rank matrix W inferred by ZINB-WaVE will capture such confounding effects: "the low-rank matrix W inferred by ZINB-WaVE will capture such confounding effects"
- [methods] BiocManager::install("zinbwave"): "BiocManager::install("zinbwave")"
- [readme] To exactly reproduce the analyses of the paper, version `0.1.1` of the `zinbwave` package is required.: [evidence span withheld — anchor status: external_doc]
