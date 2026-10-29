---
name: pca-versus-zinbwave-dimensionality-reduction-comparison
description: Use when raw scRNA-seq count matrices must be reduced to a low-dimensional
  space for clustering or trajectory inference, and dropouts/over-dispersion may make
  PCA representations miss lineage endpoints or cell types.
when_to_use_negative:
- The input is already normalized, log-transformed, or otherwise not raw counts, since
  the ZINB-WaVE model is formulated for count data with zero inflation and over-dispersion
- Unwanted technical variation (batch, QC effects) is unaccounted for, in which case
  the low-rank matrix W will capture confounding effects rather than biological signal
- A quick, cheap exploratory reduction is sufficient — the non-convex likelihood optimization
  of ZINB-WaVE carries a real computational cost relative to PCA
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: zinbwave
  role: Fits the zero-inflated negative binomial model on raw counts and outputs the
    low-dimensional factors used as the ZINB-WaVE representation
  repo: https://github.com/drisso/zinbwave
- name: ZIFA
  role: Zero-inflated dimensionality reduction comparator against which ZINB-WaVE's
    low-dimensional representation is evaluated
  repo: https://github.com/epierson9/ZIFA
- name: clusterExperiment (RSEC)
  role: Clusters cells on the 50-factor or 50-PC space to provide cluster labels for
    Slingshot
  repo: http://bioconductor.org/packages/release/bioc/html/clusterExperiment.html
- name: Slingshot
  role: Infers the unsupervised minimum spanning tree lineage structure whose endpoints
    are compared between PCA and ZINB-WaVE representations
  repo: https://github.com/kstreet13/slingshot
- name: scRNAseq
  role: Bioconductor package used to load the single-cell RNA-seq count matrix into
    R
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-017-02554-5
    title: A general and flexible method for signal extraction from single-cell RNA-seq
      data
  source_content_hash: sha256:6c423db0a8dcb11a3ae4116bf0641e627c98e6a2a7ff8478bdde131c7f07825d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/drisso/zinbwave
  tools:
  - Bioconductor (>=3.6)
  - R (>=3.4)
  - Slingshot
  - ZIFA
  - clusterExperiment (RSEC)
  - scRNAseq
  - zinbwave
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# pca-versus-zinbwave-dimensionality-reduction-comparison

## Summary

A benchmarking skill comparing PCA-based and ZINB-WaVE-based low-dimensional representations of raw single-cell RNA-seq counts to decide which better preserves biological structure for downstream tasks such as clustering and lineage inference. It is motivated by zero-inflated negative binomial modeling that accounts for dropouts and over-dispersion, where plain PCA can distort the developmental signal.

## When to use

Use when raw scRNA-seq count matrices must be reduced to a low-dimensional space for clustering or trajectory inference, and dropouts/over-dispersion may make PCA representations miss lineage endpoints or cell types.

## When NOT to use

- The input is already normalized, log-transformed, or otherwise not raw counts, since the ZINB-WaVE model is formulated for count data with zero inflation and over-dispersion
- Unwanted technical variation (batch, QC effects) is unaccounted for, in which case the low-rank matrix W will capture confounding effects rather than biological signal
- A quick, cheap exploratory reduction is sufficient — the non-convex likelihood optimization of ZINB-WaVE carries a real computational cost relative to PCA

## Inputs

- Raw single-cell RNA-seq count matrix (e.g., Fletcher mouse olfactory epithelium, GEO GSE95601, loaded via the scRNAseq package)
- Cell-level cluster labels (e.g., RSEC clusters from clusterExperiment) shared across both representations
- Number of retained dimensions (50 factors or 50 principal components)

## Outputs

- ZINB-WaVE low-dimensional factor matrix W (first 50 components) inferred from raw counts
- PCA score matrix (first 50 dimensions) on the same cells
- Unsupervised Slingshot minimum spanning tree per representation, with terminal endpoint clusters (mOSN, mSUS, MV) for comparison

## How to apply

Compute ZINB-WaVE factors directly on the raw count matrix (e.g., the Fletcher mouse olfactory epithelium data from GEO GSE95601) without prior normalization, since the cell-level intercept acts as a global-scaling normalization factor; retain the first 50 components. Cluster cells with RSEC (clusterExperiment) on the 50 ZINB-WaVE factors, then run Slingshot in unsupervised mode with those cluster labels to infer the minimum spanning tree lineage structure. As a contrast, repeat the identical clustering-plus-Slingshot pipeline on the first 50 PCA dimensions so that only the representation differs. Compare the terminal (endpoint) clusters: on the OE data the ZINB-WaVE run must terminate in the three mature lineages mOSN, mSUS, and MV, whereas the PCA-based run is expected to miss mSUS. Note zinbwave requires R (>=3.4) and Bioconductor (>=3.6).

## Related tools

- **zinbwave** (Fits the zero-inflated negative binomial model on raw counts and outputs the low-dimensional factors used as the ZINB-WaVE representation) — https://github.com/drisso/zinbwave
- **ZIFA** (Zero-inflated dimensionality reduction comparator against which ZINB-WaVE's low-dimensional representation is evaluated) — https://github.com/epierson9/ZIFA
- **clusterExperiment (RSEC)** (Clusters cells on the 50-factor or 50-PC space to provide cluster labels for Slingshot) — http://bioconductor.org/packages/release/bioc/html/clusterExperiment.html
- **Slingshot** (Infers the unsupervised minimum spanning tree lineage structure whose endpoints are compared between PCA and ZINB-WaVE representations) — https://github.com/kstreet13/slingshot
- **scRNAseq** (Bioconductor package used to load the single-cell RNA-seq count matrix into R)

## Examples

```
BiocManager::install("zinbwave"); library(zinbwave); se_zinb <- zinbwave(sce, K = 50)  # 50 factors on raw counts, then Slingshot vs first 50 PCs
```

## Evaluation signals

- The unsupervised Slingshot minimum spanning tree computed on the 50 ZINB-WaVE factors must terminate in exactly the three mature lineage endpoints mOSN, mSUS, and MV
- The contrast run on the first 50 PCA dimensions is expected to miss the mSUS endpoint — absence of this failure in the ZINB-WaVE run indicates the skill was applied correctly
- Factors are estimated on raw counts with no preliminary normalization (cell-level intercept serving as global-scaling normalization), not on normalized or logged data
- In the paper's benchmarks, ZINB-WaVE representations should be more stable and accurate than both PCA and ZIFA on simulated and real data
- Clustering (RSEC) and Slingshot settings are identical between the ZINB-WaVE and PCA runs so that only the low-dimensional representation differs

## Limitations

- There is no guarantee that the low-dimensional signal extracted by ZINB-WaVE is biologically relevant; if unwanted technical variation is not accounted for, the low-rank matrix W captures confounding effects
- Better performance comes at a computational cost from numerically optimizing a non-convex likelihood; computing time is approximately linear in cells and genes but approximately quadratic in the number of latent factors
- Supervised differential expression capability is not yet available in the described zinbwave version; only envisioned for a future release

## Evidence

- [abstract] Low-dimensional representations accounting for count characteristics: "low-dimensional representations of the data that account for zero inflation (dropouts), over-dispersion, and the count nature of the data"
- [intro] Superiority over PCA demonstrated on simulated and real data: "We demonstrate, with simulated and real data, that the model and its associated estimation procedure are able to give a more stable and accurate low-dimensional representation of the data than PCA"
- [methods] Version requirements for zinbwave: "Note that `zinbwave` requires R (>=3.4) and Bioconductor (>=3.6)."
- [discussion] Computational cost relative to PCA: "The better performance of ZINB-WaVE with respect to PCA comes at a computational cost, as we need to numerically optimize a non-convex likelihood function."
- [discussion] Confounding risk in the inferred low-rank matrix: "If unwanted technical variation affects the data and is not accounted for in the model (or in prior normalization), the low-rank matrix W inferred by ZINB-WaVE will capture such confounding effects."
- [readme] Downstream clustering and Slingshot lineage usage in the OE analysis: [evidence span withheld — anchor status: not_found]
