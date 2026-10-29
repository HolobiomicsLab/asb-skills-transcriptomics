---
name: zinb-low-dimensional-signal-extraction
description: Use when raw scRNA-seq counts need dimensionality reduction that handles
  zero inflation without prior normalization, or when unwanted variation (e.g., batch,
  detection rate) must be removed while preserving biological structure such as patient
  or cell-type effects.
when_to_use_negative:
- When the data have already been normalized and batch/biology are fully separable
  — simpler methods such as PCA may suffice, and ZINB-WaVE adds computational cost
  from non-convex likelihood optimization
- When a gene-level covariate is proposed to improve signal extraction — the authors
  found no compelling examples where this helps
- When the batch indicator is fully confounded with the biological condition of interest
  — including it as a covariate removes the biological signal (as in the batch-indicator
  contrast model)
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: zinbwave
  role: R/Bioconductor implementation of the ZINB-WaVE model used to fit the zero-inflated
    negative binomial model with latent factors and covariates
  repo: https://bioconductor.org/packages/zinbwave
- name: R (>=3.4)
  role: Runtime environment required by the zinbwave package
- name: Bioconductor (>=3.6)
  role: Bioconductor version required to install zinbwave via BiocManager
- name: scater
  role: Used to explore correlation between estimated latent factors and QC measures
    to decide which covariates to include
- name: drisso/zinb_analysis
  role: Reproducible data analysis and simulation code for the ZINB-WaVE paper
  repo: https://github.com/drisso/zinb_analysis
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-017-02554-5
    title: A general and flexible method for signal extraction from single-cell RNA-seq
      data
  source_content_hash: sha256:4befe3d1440eaf05b0ee762a137cd2115a53a55ffeb097e6e1474e33f57364c6
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BiocManager
  - Bioconductor (>=3.6)
  - R (>=3.4)
  - drisso/zinb_analysis
  - scater
  - zinbwave
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# zinb-low-dimensional-signal-extraction

## Summary

Fit a zero-inflated negative binomial model (ZINB-WaVE) to raw single-cell RNA-seq count matrices to extract a low-dimensional representation that accounts for dropouts, over-dispersion, and count structure, optionally adjusting for sample-level covariates such as detection rate or batch indicators.

## When to use

Use when raw scRNA-seq counts need dimensionality reduction that handles zero inflation without prior normalization, or when unwanted variation (e.g., batch, detection rate) must be removed while preserving biological structure such as patient or cell-type effects.

## When NOT to use

- When the data have already been normalized and batch/biology are fully separable — simpler methods such as PCA may suffice, and ZINB-WaVE adds computational cost from non-convex likelihood optimization
- When a gene-level covariate is proposed to improve signal extraction — the authors found no compelling examples where this helps
- When the batch indicator is fully confounded with the biological condition of interest — including it as a covariate removes the biological signal (as in the batch-indicator contrast model)

## Inputs

- Raw single-cell RNA-seq count matrix (genes x cells), unnormalized
- Sample-level covariate matrix X (e.g., intercept plus per-cell detection rate or batch indicator)
- Gene-level covariate matrix V including a column of ones
- Number of latent factors K
- Sample metadata (batch and patient/cell-type labels) for validation and visualization

## Outputs

- Estimated low-dimensional matrix W (cells x K latent factors)
- Fitted zinbwave model object with coefficient estimates
- 2D projections (e.g., PCA on W) colored by batch and biological condition
- Optionally, cell pairwise distances from the W space for t-SNE

## How to apply

Fit the ZINB-WaVE model to the raw count matrix with K unobserved latent factors, supplying a gene-level covariate matrix containing a column of ones so the cell-level intercept acts as a global-scaling normalization factor (no prior normalization needed). Provide a sample-level covariate matrix X encoding known sources of unwanted variation — e.g., per-cell detection rate (proportion of genes with non-zero counts) to preserve patient effects in confounded data like the glioblastoma GSE57872 set, or QC measures found correlated with latent factors via scater. As a contrast/diagnostic, fit a confounded model where X encodes a batch indicator fully confounded with biology and compare the estimated W matrices projected into 2D (PCA on W) colored by batch and by biological label. Install via BiocManager (requires R >=3.4 and Bioconductor >=3.6); optionally derive cell pairwise distances from the W space for t-SNE visualization. If unwanted technical variation is not included in the model, W will capture it — so covariate choice is the key decision point.

## Related tools

- **zinbwave** (R/Bioconductor implementation of the ZINB-WaVE model used to fit the zero-inflated negative binomial model with latent factors and covariates) — https://bioconductor.org/packages/zinbwave
- **R (>=3.4)** (Runtime environment required by the zinbwave package)
- **Bioconductor (>=3.6)** (Bioconductor version required to install zinbwave via BiocManager)
- **scater** (Used to explore correlation between estimated latent factors and QC measures to decide which covariates to include)
- **drisso/zinb_analysis** (Reproducible data analysis and simulation code for the ZINB-WaVE paper) — https://github.com/drisso/zinb_analysis

## Examples

```
BiocManager::install("zinbwave")
```

## Evaluation signals

- In the covariate-adjusted fit's low-dimensional space, batches are well mixed while cells cluster by biological label (e.g., patient) — the confounded batch-indicator model's W should fail to separate patients
- Estimated latent factors can be correlated with QC measures (via scater); highly correlated unwanted sources should be absorbed by included covariates, leaving W capturing wanted variation
- The model is fitted to raw counts: the sample-level intercept serves as a global-scaling normalization factor, so results should not require a prior normalization step
- Runtime scaling checks: computing time approximately linear in number of cells and genes, approximately quadratic in the number of latent factors
- Confirmed qualitative batch-mixing/patient-separation pattern in the saved PCA figure of the estimated W (e.g., patel_zinbwave_pca.png)

## Limitations

- If unwanted technical variation affects the data and is not accounted for in the model or prior normalization, the low-rank matrix W inferred by ZINB-WaVE will capture such confounding effects
- There is no guarantee that the low-dimensional signal extracted by ZINB-WaVE is biologically relevant
- Requires numerical optimization of a non-convex likelihood function, which incurs computational cost
- zinbwave requires R (>=3.4) and Bioconductor (>=3.6); reproducing the paper's analyses exactly requires zinbwave version 0.1.1
- No compelling examples were found in which adding a gene-level covariate improves signal extraction

## Evidence

- [intro] The ZINB-WaVE model includes a sample-level intercept serving as a global-scaling normalization factor and allows inclusion of gene-level and sample-level covariates: "includes a sample-level intercept, which serves as a global-scaling normalization factor, and gives the user the ability to include both gene-level and sample-level covariates"
- [methods] Install requirement for zinbwave: "Note that `zinbwave` requires R (>=3.4) and Bioconductor (>=3.6)."
- [discussion] Unmodeled technical variation leaks into W: "If unwanted technical variation affects the data and is not accounted for in the model (or in prior normalization), the low-rank matrix W inferred by ZINB-WaVE will capture such confounding effects."
- [discussion] Raw counts need no prior normalization thanks to the intercept: "the corresponding cell-level intercept acts as a global-scaling normalization factor, allowing the modeling of raw count data, with no need for prior normalization"
- [abstract] Model outperforms PCA and ZIFA on zero-inflated counts: "the model and its associated estimation procedure are able to give a more stable and accurate low-dimensional representation of the data than principal component analysis (PCA) and zero-inflated"
- [discussion] Explore latent factors vs QC measures with scater before including covariates: "explore the correlation between the latent factors estimated by our procedure and known measures of quality control"
- [readme] Exact reproduction requires zinbwave 0.1.1: [evidence span withheld — anchor status: external_doc]
