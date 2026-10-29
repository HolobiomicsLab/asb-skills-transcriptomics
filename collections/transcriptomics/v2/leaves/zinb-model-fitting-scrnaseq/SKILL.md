---
name: zinb-model-fitting-scrnaseq
description: Use when raw scRNA-seq UMI count matrices need dimensionality reduction
  that models zero inflation and over-dispersion directly, replacing PCA or ZIFA without
  a preliminary normalization step.
when_to_use_negative:
- When the goal is supervised differential expression — this capability is not yet
  available in zinbwave and is only envisioned for a future version
- When a preliminary low-dimensional representation already exists or a simple fast
  linear method suffices and ZINB's non-convex optimization cost is prohibitive
- When unwanted technical variation is unaccounted for and cannot be modeled — W will
  capture such confounding rather than biological signal
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: zinbwave
  role: R/Bioconductor package implementing the zero-inflated negative binomial model
    fitting on raw UMI counts
  repo: https://bioconductor.org/packages/zinbwave
- name: scater
  role: Used to explore correlation between the estimated latent factors and QC measures
- name: scRNAseq
  role: Bioconductor package used to load the PBMC count data into R
- name: drisso/zinb_analysis
  role: Reproduction code and simulations for the ZINB-WaVE paper analyses
  repo: https://github.com/drisso/zinb_analysis
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-017-02554-5
    title: A general and flexible method for signal extraction from single-cell RNA-seq
      data
  source_content_hash: sha256:07e9730b2703800b691264c2974eb70258f4d12eb60167f9b7f326cb7d9a548d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bioconductor (>=3.6)
  - R (>=3.4)
  - drisso/zinb_analysis
  - scRNAseq
  - scater
  - zinbwave
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# zinb-model-fitting-scrnaseq

## Summary

Fit a zero-inflated negative binomial (ZINB-WaVE) model to raw single-cell RNA-seq UMI counts to extract K low-dimensional latent factors (the W matrix) that account for dropouts, over-dispersion, and the count nature of the data, without prior normalization.

## When to use

Use when raw scRNA-seq UMI count matrices need dimensionality reduction that models zero inflation and over-dispersion directly, replacing PCA or ZIFA without a preliminary normalization step.

## When NOT to use

- When the goal is supervised differential expression — this capability is not yet available in zinbwave and is only envisioned for a future version
- When a preliminary low-dimensional representation already exists or a simple fast linear method suffices and ZINB's non-convex optimization cost is prohibitive
- When unwanted technical variation is unaccounted for and cannot be modeled — W will capture such confounding rather than biological signal

## Inputs

- Raw scRNA-seq UMI count matrix (e.g. 10x Genomics 68k PBMC data loadable via the scRNAseq package ecosystem)
- SummarizedExperiment / SingleCellExperiment object in R/Bioconductor
- Optional gene-level covariate matrix (column of ones to enable global-scaling normalization intercept)
- Optional sample-level covariates or QC measures (e.g. library metrics from scater)

## Outputs

- Low-dimensional W matrix (cells x K inferred latent factors, e.g. cells x 10)
- Fitted zinbmodel object with estimated coefficients
- Cell-to-cell Euclidean distances in the ZINB-WaVE low-dimensional space, usable for t-SNE or SNN-graph clustering

## How to apply

Install zinbwave via BiocManager::install("zinbwave") in R (>=3.4) / Bioconductor (>=3.6), then run zinbwave on the raw UMI count matrix with K=10 latent factors (for the 68k PBMC example). Include a column of ones in the gene-level covariate matrix so the cell-level intercept acts as a global-scaling normalization factor, allowing modeling of raw counts with no prior normalization. Extract the inferred cells-by-K low-dimensional W matrix as the output representation. Optionally explore correlation between estimated latent factors and QC measures (e.g. via scater) and include highly correlated QC measures as sample-level covariates so W does not capture unwanted technical variation. Be aware optimization of the non-convex likelihood carries a computational cost, with runtime approximately linear in cells and genes and approximately quadratic in the number of latent factors.

## Related tools

- **zinbwave** (R/Bioconductor package implementing the zero-inflated negative binomial model fitting on raw UMI counts) — https://bioconductor.org/packages/zinbwave
- **scater** (Used to explore correlation between the estimated latent factors and QC measures)
- **scRNAseq** (Bioconductor package used to load the PBMC count data into R)
- **drisso/zinb_analysis** (Reproduction code and simulations for the ZINB-WaVE paper analyses) — https://github.com/drisso/zinb_analysis

## Examples

```
BiocManager::install("zinbwave"); library(zinbwave); se <- scRNAseq::KolodziejczakCellLine(); fit <- zinbwave(Y = assays(se)$counts, K = 10); W <- reducedDim(fit)
```

## Evaluation signals

- The extracted W matrix reproduces low-dimensional representations more stable and accurate than PCA and ZIFA on the same data, per the paper's benchmark
- Downstream clustering of W (e.g. SNN clustering on the 68k PBMC data, K=10) recapitulates known populations with ~80% T-cells, 6% NK, 5% B-cells, 3% CD14+ monocytes and rare populations (pDC, megakaryocytes) within a few percentage points
- Fitted model runs on raw counts directly, confirming the cell-level intercept is serving as a global-scaling normalization factor
- Latent factors show low correlation with QC measures (checked via scater), indicating unwanted technical variation was not absorbed into W
- The zinbwave package version and environment match requirements: R >=3.4 and Bioconductor >=3.6 (or zinbwave v0.1.1 to exactly reproduce the paper)

## Limitations

- If unwanted technical variation affects the data and is not accounted for in the model or prior normalization, the low-rank matrix W will capture such confounding effects
- There is no guarantee that the low-dimensional signal extracted by ZINB-WaVE is biologically relevant
- Better performance comes at a computational cost from numerically optimizing a non-convex likelihood function
- Supervised differential expression is not yet available; only envisioned for a future zinbwave version

## Evidence

- [abstract] the model and its associated estimation procedure are able to give a more stable and accurate low-dimensional representation of the data than principal component analysis (PCA) and zero-inflated: "the model and its associated estimation procedure are able to give a more stable and accurate low-dimensional representation of the data than principal component analysis (PCA)"
- [intro] includes a sample-level intercept, which serves as a global-scaling normalization factor, and gives the user the ability to include both gene-level and sample-level covariates: "includes a sample-level intercept, which serves as a global-scaling normalization factor, and gives the user the ability to include both gene-level and sample-level covariates"
- [methods] Note that `zinbwave` requires R (>=3.4) and Bioconductor (>=3.6).: "Note that `zinbwave` requires R (>=3.4) and Bioconductor (>=3.6)."
- [discussion] An alternative approach is to derive such distances from the low-dimensional space defined by the factors inferred by ZINB-WaVE.: "An alternative approach is to derive such distances from the low-dimensional space defined by the factors inferred by ZINB-WaVE."
- [discussion] the corresponding cell-level intercept acts as a global-scaling normalization factor, allowing the modeling of raw count data, with no need for prior normalization: "the corresponding cell-level intercept acts as a global-scaling normalization factor, allowing the modeling of raw count data, with no need for prior normalization"
- [readme] BiocManager::install("zinbwave"): "This can be installed in R with the following code.

{r}
library(devtools)
install_github("drisso/archive-zinbwave@v0.1.1")"
