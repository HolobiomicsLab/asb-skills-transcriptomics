---
name: zinb-wave-latent-factor-extraction
description: Use when raw scRNA-seq count matrices (e.g., GEO GSE95601 olfactory epithelium
  data) need a low-dimensional representation that handles zero inflation and over-dispersion
  without prior normalization, especially as input to clustering or Slingshot lineage
  inference.
when_to_use_negative:
- Unwanted technical variation affects the data and is not accounted for in the model
  or in prior normalization — the low-rank matrix W will capture such confounding
  effects instead of the biological signal
- Supervised downstream analyses that need confounder-adjusted differential expression
  within zinbwave are required — this capability is not yet available, only envisioned
  for a future version
- 'A preliminary normalized, non-count representation is the required input (e.g.,
  log-transformed data for ZIFA: Y = log2(1 + count_data)) — zinbwave instead models
  raw counts directly'
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: zinbwave
  role: Implements the zero-inflated negative binomial model used to infer the latent
    factors from raw counts
  repo: https://github.com/drisso/zinbwave
- name: scRNAseq
  role: Bioconductor package used to load the single-cell RNA-seq count data
- name: Slingshot
  role: Lineage trajectory algorithm run on the ZINB-WaVE factors (with RSEC cluster
    labels) to infer branching lineages
  repo: https://github.com/kstreet13/slingshot
- name: clusterExperiment
  role: Provides RSEC clustering of cells on the 50 ZINB-WaVE factors to supply cluster
    labels for Slingshot
  repo: https://github.com/epurdom/clusterExperiment
- name: scater
  role: Used to explore correlation between estimated latent factors and QC measures
- name: R (>=3.4)
  role: Runtime requirement for zinbwave
- name: Bioconductor (>=3.6)
  role: Runtime requirement for zinbwave
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-017-02554-5
    title: A general and flexible method for signal extraction from single-cell RNA-seq
      data
  source_content_hash: sha256:de3c32fd974fd933d904553793a1c346ecfc962433fcb0abbac3fdcc4b17b2dd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/drisso/zinbwave
  tools:
  - Bioconductor (>=3.6)
  - R (>=3.4)
  - Slingshot
  - clusterExperiment
  - scRNAseq
  - scater
  - zinbwave
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# zinb-wave-latent-factor-extraction

## Summary

Extract low-dimensional latent factors from raw single-cell RNA-seq counts using ZINB-WaVE, a zero-inflated negative binomial model that accounts for dropouts, over-dispersion, and the count nature of the data without prior normalization. The resulting factors give a more stable and accurate representation than PCA or ZIFA and support downstream clustering and trajectory inference.

## When to use

Use when raw scRNA-seq count matrices (e.g., GEO GSE95601 olfactory epithelium data) need a low-dimensional representation that handles zero inflation and over-dispersion without prior normalization, especially as input to clustering or Slingshot lineage inference.

## When NOT to use

- Unwanted technical variation affects the data and is not accounted for in the model or in prior normalization — the low-rank matrix W will capture such confounding effects instead of the biological signal
- Supervised downstream analyses that need confounder-adjusted differential expression within zinbwave are required — this capability is not yet available, only envisioned for a future version
- A preliminary normalized, non-count representation is the required input (e.g., log-transformed data for ZIFA: Y = log2(1 + count_data)) — zinbwave instead models raw counts directly

## Inputs

- Raw scRNA-seq count matrix (e.g., Fletcher mouse olfactory epithelium data, GEO GSE95601)
- Number of latent factors K (e.g., 50 ZINB-WaVE components)
- Optional gene-level and sample-level covariate matrices (column of ones included so the cell-level intercept is a global-scaling normalization factor)
- Optional QC measures to explore as covariates via scater

## Outputs

- W matrix of K latent factors (low-dimensional representation of cells)
- Cell pairwise distances in ZINB-WaVE space for t-SNE visualization
- Cluster labels (e.g., RSEC) and lineage structures (Slingshot) computed on the factors
- Fitted zinbwave model object with estimated latent factors and normalization intercept

## How to apply

Install zinbwave via BiocManager (it requires R >=3.4 and Bioconductor >=3.6; version 0.1.1 reproduces the paper exactly). Fit the zero-inflated negative binomial model directly on the raw count matrix — a column of ones in the gene-level covariate matrix makes the cell-level intercept act as a global-scaling normalization factor, so no prior normalization is needed — optionally including gene-level and sample-level covariates (e.g., QC measures correlated with latent factors). Retain the desired number of latent factors (the olfactory epithelium workflow keeps the first 50 components, as computing time is approximately quadratic in the number of latent factors). Use the inferred W matrix as the low-dimensional space for downstream steps: derive cell pairwise distances for t-SNE, cluster cells with RSEC (clusterExperiment), and run Slingshot; on the mouse olfactory epithelium data this recovers all three mature lineage endpoints (mOSN, mSUS, MV) even in unsupervised mode, whereas PCA factors miss mSUS. Verify the biological relevance of the extracted signal against known cell types, since ZINB-WaVE itself gives no guarantee that the low-dimensional signal is biologically relevant.

## Related tools

- **zinbwave** (Implements the zero-inflated negative binomial model used to infer the latent factors from raw counts) — https://github.com/drisso/zinbwave
- **scRNAseq** (Bioconductor package used to load the single-cell RNA-seq count data)
- **Slingshot** (Lineage trajectory algorithm run on the ZINB-WaVE factors (with RSEC cluster labels) to infer branching lineages) — https://github.com/kstreet13/slingshot
- **clusterExperiment** (Provides RSEC clustering of cells on the 50 ZINB-WaVE factors to supply cluster labels for Slingshot) — https://github.com/epurdom/clusterExperiment
- **scater** (Used to explore correlation between estimated latent factors and QC measures)
- **R (>=3.4)** (Runtime requirement for zinbwave)
- **Bioconductor (>=3.6)** (Runtime requirement for zinbwave)

## Examples

```
BiocManager::install("zinbwave"); library(zinbwave); zinb <- zinbFit(counts, K = 50)
```

## Evaluation signals

- On the mouse olfactory epithelium data (GSE95601), the minimum spanning tree inferred by unsupervised Slingshot on the 50 ZINB-WaVE factors must terminate in exactly the three mature endpoints mOSN, mSUS, and MV
- As a contrast, the same unsupervised Slingshot run on the first 50 PCA dimensions is expected to miss the mSUS endpoint
- The ZINB-WaVE representation should be more stable and accurate than PCA and ZIFA computed on the same data, without a preliminary normalization step
- Confirm the model was fit on raw counts: the cell-level intercept acts as a global-scaling normalization factor, so no prior normalization was applied
- Environment sanity check: R >=3.4 and Bioconductor >=3.6 (zinbwave 0.1.1 for exact paper reproduction); check for convergence of the non-convex likelihood optimization, whose cost is approximately linear in cells and genes and quadratic in the number of latent factors

## Limitations

- If unwanted technical variation affects the data and is not accounted for in the model or in prior normalization, the low-rank matrix W inferred by ZINB-WaVE will capture such confounding effects
- There is no guarantee that the low-dimensional signal extracted by ZINB-WaVE is biologically relevant
- Better performance with respect to PCA comes at a computational cost, as a non-convex likelihood function must be numerically optimized; computing time is approximately quadratic in the number of latent factors
- Supervised differential expression capability (confounder-adjusted DE within the model) is not yet available in zinbwave
- No compelling examples were found in which adding a gene-level covariate improves signal extraction

## Evidence

- [abstract] the model and its associated estimation procedure are able to give a more stable and accurate low-dimensional representation of the data than principal component analysis (PCA) and zero-inflated: "the model and its associated estimation procedure are able to give a more stable and accurate low-dimensional representation of the data than principal component analysis (PCA) and zero-inflated"
- [methods] Note that `zinbwave` requires R (>=3.4) and Bioconductor (>=3.6).: "Note that `zinbwave` requires R (>=3.4) and Bioconductor (>=3.6)."
- [discussion] the corresponding cell-level intercept acts as a global-scaling normalization factor, allowing the modeling of raw count data, with no need for prior normalization: "the corresponding cell-level intercept acts as a global-scaling normalization factor, allowing the modeling of raw count data, with no need for prior normalization"
- [readme] After filtering and normalization, we clustered the data using clusterExperiment, performed developmental ordering and inferred lineage trajectories and branching with slingshot.: [evidence span withheld — anchor status: not_found]
- [discussion] If unwanted technical variation affects the data and is not accounted for in the model (or in prior normalization), the low-rank matrix W inferred by ZINB-WaVE will capture such confounding effects.: "If unwanted technical variation affects the data and is not accounted for in the model (or in prior normalization), the low-rank matrix W inferred by ZINB-WaVE will capture such confounding effects."
