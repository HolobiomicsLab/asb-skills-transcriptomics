---
name: shared-nearest-neighbor-clustering
description: Use when raw scRNA-seq UMI counts (e.g., 10x Genomics 68k PBMC) have
  been embedded into a low-dimensional ZINB-WaVE W matrix and you need cell-population
  clustering robust to dropouts and over-dispersion.
when_to_use_negative:
- If unwanted technical variation is unaccounted for, W will capture confounding effects
  and the clusters may not be biologically relevant
- When a preliminary log-normalized expression matrix or a non-count representation
  is the intended clustering input (e.g., ZIFA expects log2(1+counts)), ZINB-WaVE's
  raw-count W-based clustering may not be the right route
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: zinbwave
  role: Infers the K=10 latent factors / W matrix on raw counts on which the SNN graph
    is built
  repo: https://github.com/drisso/zinbwave
- name: R (>=3.4)
  role: Runtime environment required by zinbwave
- name: Bioconductor (>=3.6)
  role: Ecosystem distributing zinbwave and companion packages (scRNAseq, scater,
    clusterExperiment)
  repo: https://bioconductor.org/packages/zinbwave
- name: scRNAseq
  role: Provides the 10x Genomics 68k PBMC UMI count data as input
- name: scater
  role: Explore correlation between estimated latent factors and QC measures before
    clustering
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-017-02554-5
    title: A general and flexible method for signal extraction from single-cell RNA-seq
      data
  source_content_hash: sha256:5aab8fa3ee1fd708134d9657285230c4b409f36d2b4fc511451b3ca636f6c862
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/drisso/zinbwave
  tools:
  - Bioconductor (>=3.6)
  - R (>=3.4)
  - scRNAseq
  - scater
  - zinbwave
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# shared-nearest-neighbor-clustering

## Summary

Cells are clustered by building a shared-nearest-neighbor graph on the low-dimensional W matrix inferred by ZINB-WaVE (e.g., K=10 factors on the 68k PBMC data), enabling Seurat-style identification of major and rare cell populations without prior normalization of raw UMI counts.

## When to use

Use when raw scRNA-seq UMI counts (e.g., 10x Genomics 68k PBMC) have been embedded into a low-dimensional ZINB-WaVE W matrix and you need cell-population clustering robust to dropouts and over-dispersion.

## When NOT to use

- If unwanted technical variation is unaccounted for, W will capture confounding effects and the clusters may not be biologically relevant
- When a preliminary log-normalized expression matrix or a non-count representation is the intended clustering input (e.g., ZIFA expects log2(1+counts)), ZINB-WaVE's raw-count W-based clustering may not be the right route

## Inputs

- Raw scRNA-seq UMI count matrix (e.g., 10x Genomics 68k PBMC), loaded via the scRNAseq package as a SummarizedExperiment
- Low-dimensional W matrix (cells x K, K=10) inferred by zinbwave

## Outputs

- Cell-to-cluster assignments with per-cluster cell proportions (e.g., pbmc_zinbwave_clusters.csv)
- Annotated cell populations (T-cells, NK, B-cells, monocytes, DCs, megakaryocytes)
- t-SNE embedding of Euclidean distances in the ZINB-WaVE low-dimensional space (as in Fig. 3c)

## How to apply

First run zinbwave on the raw count matrix with K=10 latent factors, relying on the cell-level intercept as a global-scaling normalization factor so no preliminary normalization is needed, and extract the inferred cells-by-K W matrix. Build a shared-nearest-neighbor graph on W and cluster the cells Seurat-style (as supported by the clusterExperiment ecosystem used in the paper's workflows). Compute Euclidean cell-to-cell distances in the ZINB-WaVE low-dimensional space and embed with t-SNE (as in Fig. 3c) for visualization of the resulting clusters. Tabulate the proportion of cells per cluster and annotate populations (T-cells, cytotoxic T-cells, NK, B-cells, CD14+/CD16+ monocytes, dendritic cells, megakaryocytes); the rationale is that ZINB-WaVE's W gives a more stable and accurate low-dimensional representation than PCA or ZIFA for count data. Optionally check correlation of the estimated latent factors with QC measures (e.g., via scater) before trusting the clustering.

## Related tools

- **zinbwave** (Infers the K=10 latent factors / W matrix on raw counts on which the SNN graph is built) — https://github.com/drisso/zinbwave
- **R (>=3.4)** (Runtime environment required by zinbwave)
- **Bioconductor (>=3.6)** (Ecosystem distributing zinbwave and companion packages (scRNAseq, scater, clusterExperiment)) — https://bioconductor.org/packages/zinbwave
- **scRNAseq** (Provides the 10x Genomics 68k PBMC UMI count data as input)
- **scater** (Explore correlation between estimated latent factors and QC measures before clustering)

## Examples

```
BiocManager::install("zinbwave"); W <- zinbwave(sce, K = 10)@assays$data$W  # cluster cells Seurat-style on the 10-factor W matrix
```

## Evaluation signals

- Cluster proportion table reproduces the published composition within a few percentage points: ~80% T-cells, 9% activated cytotoxic T-cells, 6% NK, 5% B-cells, 3% CD14+ monocytes, 2.5% CD16+ monocytes, 1% CD1C+ DC, 0.5% pDC, 0.2% megakaryocytes
- W matrix has one row per cell and exactly K=10 factor columns
- t-SNE embedding derived from Euclidean distances in W-space visually separates the same major populations as Zheng et al. (Fig. 3c-like structure)
- Raw counts modeled directly with the cell-level intercept acting as global-scaling normalization (no prior normalization applied)
- Cluster identities confirmed by marker genes (CD4+/CD8+ T-cells dominating clusters 0–6)

## Limitations

- There is no guarantee that the low-dimensional signal extracted by ZINB-WaVE is biologically relevant; unaccounted technical variation will be captured by W and propagate into clusters
- Computational cost: ZINB-WaVE requires numerical optimization of a non-convex likelihood; computing time is roughly linear in cells/genes and quadratic in the number of latent factors
- Exact reproduction of the paper's analyses requires version 0.1.1 of the zinbwave package (drisso/archive-zinbwave@v0.1.1)

## Evidence

- [discussion] Distances in low-dimensional space for t-SNE visualization: "An alternative approach is to derive such distances from the low-dimensional space defined by the factors inferred by ZINB-WaVE."
- [discussion] Raw counts modeled without prior normalization: "the corresponding cell-level intercept acts as a global-scaling normalization factor, allowing the modeling of raw count data, with no need for prior normalization"
- [abstract] More stable low-dimensional representation than PCA/ZIFA: "give a more stable and accurate low-dimensional representation of the data than principal component analysis (PCA)"
- [readme] Package dependencies for reproducing the analysis: "- cluster
- matrixStats
- magrittr
- RColorBrewer
- ggplot2
- reshape
- dplyr
- knitr
- rmarkdown
- mclust
- cowplot
- rARPACK
- Rtsne"
- [discussion] Cluster W-derived factors with QC correlation checks: "explore the correlation between the latent factors estimated by our procedure and known measures of quality control"
