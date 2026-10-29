---
name: umi-count-matrix-processing
description: Use when the input is a raw single-cell UMI count matrix and you need
  zero-inflation- and over-dispersion-aware low-dimensional signal extraction without
  a preliminary normalization step.
when_to_use_negative:
- If unwanted technical variation affects the data and is not accounted for in the
  model or prior normalization, the inferred W matrix will capture confounding effects
  rather than biological signal.
- If you need supervised differential expression on the model factors — this capability
  was only envisioned for a future zinbwave version.
- 'When a lightweight linear method suffices: ZINB-WaVE requires numerically optimizing
  a non-convex likelihood, incurring a computational cost relative to PCA.'
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0091
tools:
- name: zinbwave
  role: Bioconductor R package implementing the zero-inflated negative binomial model
    used to extract the low-dimensional W matrix from raw UMI counts
  repo: https://github.com/drisso/zinbwave
- name: scRNAseq
  role: Bioconductor package ecosystem used to load the 10x Genomics 68k PBMC UMI
    count matrix
- name: ZIFA
  role: Comparison dimensionality-reduction method (zero-inflated factor analysis);
    requires log2(1 + count) preprocessing unlike zinbwave on raw counts
  repo: https://github.com/epierson9/ZIFA
- name: scater
  role: Used to explore correlation between estimated latent factors and QC measures
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-017-02554-5
    title: A general and flexible method for signal extraction from single-cell RNA-seq
      data
  source_content_hash: sha256:6974ee23488eeb5c81d12928effa17738ee560338c9cff5e8bc58f4044f70553
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/drisso/zinbwave
  tools:
  - Bioconductor (>=3.6)
  - R (>=3.4)
  - ZIFA
  - scRNAseq
  - scater
  - zinbwave
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# umi-count-matrix-processing

## Summary

Loads a raw scRNA-seq UMI count matrix (e.g., the 10x Genomics 68k PBMC data set) into R/Bioconductor and models it with ZINB-WaVE directly on the counts, using the cell-level intercept as a global-scaling normalization factor so no prior normalization is needed.

## When to use

Use when the input is a raw single-cell UMI count matrix and you need zero-inflation- and over-dispersion-aware low-dimensional signal extraction without a preliminary normalization step.

## When NOT to use

- If unwanted technical variation affects the data and is not accounted for in the model or prior normalization, the inferred W matrix will capture confounding effects rather than biological signal.
- If you need supervised differential expression on the model factors — this capability was only envisioned for a future zinbwave version.
- When a lightweight linear method suffices: ZINB-WaVE requires numerically optimizing a non-convex likelihood, incurring a computational cost relative to PCA.

## Inputs

- Raw scRNA-seq UMI count matrix (e.g., 10x Genomics 68k PBMC data set, accessible via the scRNAseq Bioconductor package)
- R environment with R >=3.4 and Bioconductor >=3.6

## Outputs

- Low-dimensional W matrix (cells x K=10 ZINB-WaVE latent factors)
- Pairwise cell-to-cell Euclidean distances in the ZINB-WaVE space for t-SNE visualization
- Cluster assignments and per-cluster cell proportions (e.g., pbmc_zinbwave_clusters.csv)

## How to apply

Load the raw UMI count matrix (e.g., the 10x Genomics 68k PBMC data available via the scRNAseq package ecosystem) into R (>=3.4) with Bioconductor (>=3.6), installing zinbwave via BiocManager. Run zinbwave directly on the raw counts with K=10 latent factors: because the model includes a sample-level intercept that acts as a global-scaling normalization factor (enabled by a column of ones in the gene-level covariate matrix), no prior normalization is required. Extract the inferred cells-by-10 W matrix as the low-dimensional representation; per the paper it is more stable and accurate than PCA or ZIFA on count data. Downstream, compute Euclidean cell-to-cell distances in this space for t-SNE embedding (as in Fig. 3c) and build a shared-nearest-neighbor graph for Seurat-style clustering.

## Related tools

- **zinbwave** (Bioconductor R package implementing the zero-inflated negative binomial model used to extract the low-dimensional W matrix from raw UMI counts) — https://github.com/drisso/zinbwave
- **scRNAseq** (Bioconductor package ecosystem used to load the 10x Genomics 68k PBMC UMI count matrix)
- **ZIFA** (Comparison dimensionality-reduction method (zero-inflated factor analysis); requires log2(1 + count) preprocessing unlike zinbwave on raw counts) — https://github.com/epierson9/ZIFA
- **scater** (Used to explore correlation between estimated latent factors and QC measures)

## Examples

```
BiocManager::install("zinbwave"); library(zinbwave); w <- assay(zinbwave(sce, K = 10))
```

## Evaluation signals

- Session reports R >=3.4 and Bioconductor >=3.6 with zinbwave installed via BiocManager::install("zinbwave").
- The W matrix has exactly cells rows and K=10 columns of inferred latent factors.
- Cluster proportion table reproduces the published 68k PBMC composition within a few percentage points (~80% T-cells, 9% activated cytotoxic T-cells, 6% NK, 5% B-cells, 3% CD14+ monocytes, 2.5% CD16+ monocytes, 1% CD1C+ DC, 0.5% pDC, 0.2% megakaryocytes).
- Major populations (T-cells, NK, B-cells, CD14+/CD16+ monocytes, dendritic cells, megakaryocytes) are identifiable, recapitulating the Zheng et al. clustering.
- t-SNE embedding on the ZINB-WaVE-derived distances visually separates major cell populations as in Fig. 3c.

## Limitations

- There is no guarantee that the low-dimensional signal extracted by ZINB-WaVE is biologically relevant; unaccounted technical variation will be captured by W.
- Estimation requires numerically optimizing a non-convex likelihood, adding computational cost relative to PCA (computing time ~linear in cells and genes, ~quadratic in the number of latent factors).
- To exactly reproduce the paper's analyses, zinbwave version 0.1.1 (installed from drisso/archive-zinbwave@v0.1.1) is required, per the zinb_analysis README.

## Evidence

- [intro] zinbwave implemented in Bioconductor: "The approach is implemented in the open-source R package zinbwave, publicly available through the Bioconductor Project"
- [methods] version requirements: "Note that `zinbwave` requires R (>=3.4) and Bioconductor (>=3.6)."
- [discussion] raw counts without prior normalization: "the corresponding cell-level intercept acts as a global-scaling normalization factor, allowing the modeling of raw count data, with no need for prior normalization"
- [discussion] distances from ZINB-WaVE space for t-SNE: "An alternative approach is to derive such distances from the low-dimensional space defined by the factors inferred by ZINB-WaVE."
- [readme] package purpose from README: "This package implements a zero-inflated negative binomial model for single-cell RNA-seq data, with latent factors."
