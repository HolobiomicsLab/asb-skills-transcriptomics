---
name: tsne-visualization-of-scrnaseq-latent-space
description: Use when you have a low-dimensional per-cell representation (e.g., ZINB-WaVE
  W factors) and need a 2D t-SNE visualization of cell populations for clustering
  inspection or publication figures.
when_to_use_negative:
- When the latent factors W may capture unaccounted unwanted technical variation —
  the low-rank matrix will confound the visualization with batch effects
- When a supervised differential expression or trajectory output is the goal rather
  than population-level 2D visualization
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: zinbwave
  role: Infers the low-dimensional W matrix whose cell-to-cell Euclidean distances
    feed the t-SNE embedding
  repo: https://github.com/drisso/zinbwave
- name: Rtsne
  role: R package used in the ZINB-WaVE paper workflow to perform the t-SNE embedding
    of latent-space distances
  repo: https://github.com/drisso/zinb_analysis
- name: scRNAseq
  role: Provides the 10x Genomics 68k PBMC UMI count matrix used to build the latent
    space
  repo: https://github.com/drisso/zinb_analysis
- name: scater
  role: Bioconductor package used to explore correlations between latent factors and
    QC measures (Fig. 2f), complementing the t-SNE visualization
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-017-02554-5
    title: A general and flexible method for signal extraction from single-cell RNA-seq
      data
  source_content_hash: sha256:557e8e5be78e14206a1c85e2c34f0258f2fe4a077da45f8e55ec9df7ee338555
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/drisso/zinbwave
  tools:
  - Bioconductor (>=3.6)
  - R (>=3.4)
  - Rtsne
  - scRNAseq
  - scater
  - zinbwave
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# tsne-visualization-of-scrnaseq-latent-space

## Summary

Visualize single-cell RNA-seq structure by computing Euclidean cell-to-cell distances in a model-derived latent space (e.g., the cells x K W matrix inferred by ZINB-WaVE with K=10) and embedding them with t-SNE (Rtsne), as done in Fig. 3c of the 68k PBMC analysis.

## When to use

Use when you have a low-dimensional per-cell representation (e.g., ZINB-WaVE W factors) and need a 2D t-SNE visualization of cell populations for clustering inspection or publication figures.

## When NOT to use

- When the latent factors W may capture unaccounted unwanted technical variation — the low-rank matrix will confound the visualization with batch effects
- When a supervised differential expression or trajectory output is the goal rather than population-level 2D visualization

## Inputs

- ZINB-WaVE W matrix (cells x K inferred latent factors, e.g., K=10) from raw UMI counts of the 10x Genomics 68k PBMC data set
- Per-cell cluster labels from shared-nearest-neighbor (Seurat-style) clustering on W
- Rtsne R package (dependency of the drisso/zinb_analysis reproducibility repository)

## Outputs

- 2D t-SNE embedding of cells colored by cluster (Fig. 3c-style plot)
- Cell-to-cell Euclidean distance matrix computed in the ZINB-WaVE low-dimensional space

## How to apply

Rather than running t-SNE on PCA coordinates or normalized expression, derive cell pairwise distances from the low-dimensional space defined by the factors inferred by ZINB-WaVE: extract the W matrix (cells x K, K=10 for the PBMC analysis) and compute Euclidean distances between cells in that space. Embed these distances with t-SNE (Rtsne in the zinb_analysis workflow) to produce the 2D map, coloring cells by SNN-cluster assignment to identify populations such as CD4+ T-cells, CD8+ T-cells, NK, B-cells, monocytes, dendritic cells, and megakaryocytes. The rationale is that ZINB-WaVE accounts for zero inflation (dropouts), over-dispersion, and the count nature of the data, so the latent distances are more stable than PCA-based ones and require no prior normalization (the cell-level intercept acts as a global-scaling normalization factor). This strategy was used effectively in Fig. 3c to visualize the PBMC data set.

## Related tools

- **zinbwave** (Infers the low-dimensional W matrix whose cell-to-cell Euclidean distances feed the t-SNE embedding) — https://github.com/drisso/zinbwave
- **Rtsne** (R package used in the ZINB-WaVE paper workflow to perform the t-SNE embedding of latent-space distances) — https://github.com/drisso/zinb_analysis
- **scRNAseq** (Provides the 10x Genomics 68k PBMC UMI count matrix used to build the latent space) — https://github.com/drisso/zinb_analysis
- **scater** (Bioconductor package used to explore correlations between latent factors and QC measures (Fig. 2f), complementing the t-SNE visualization)

## Examples

```
library(Rtsne); ts <- Rtsne(W, pca = FALSE, check_duplicates = FALSE)  # W = zinbwave@W (cells x 10), Euclidean t-SNE as in Fig. 3c
```

## Evaluation signals

- The t-SNE map should spatially separate the Zheng et al. PBMC populations, with ~80% of cells falling in T-cell clusters (Clusters 0–1 CD4+; Clusters 2–6 CD8+)
- Cluster proportions on the map should be within a few percentage points of the published composition: ~9% activated cytotoxic T-cells, ~6% NK, ~5% B-cells, ~3% CD14+ monocytes, ~2.5% CD16+ monocytes, ~1% CD1C+ DC, ~0.5% pDC, ~0.2% megakaryocytes
- Distances must be computed in the ZINB-WaVE W space (K=10), not on PCA or log-normalized expression, to reproduce Fig. 3c
- For exact paper reproduction, use zinbwave version 0.1.1 as specified in the zinb_analysis README
- Runtime should remain tractable: zinbwave computing time is approximately linear in the number of cells and genes and approximately quadratic in the number of latent factors

## Limitations

- There is no guarantee that the low-dimensional signal extracted by ZINB-WaVE is biologically relevant; if unwanted technical variation is unaccounted for, W will capture confounding effects and the t-SNE will be misleading
- t-SNE coordinates are not suitable as quantitative inputs — the underlying Euclidean distances in W are the analysis object
- ZINB-WaVE requires numerical optimization of a non-convex likelihood, incurring computational cost before the t-SNE step

## Evidence

- [discussion] Derive cell pairwise distances from ZINB-WaVE low-dimensional space for t-SNE visualization: "An alternative approach is to derive such distances from the low-dimensional space defined by the factors inferred by ZINB-WaVE."
- [discussion] Fig. 3c PBMC visualization: "This strategy was used effectively in Fig. 3c to visualize the PBMC data set."
- [abstract] More stable than PCA: "the model and its associated estimation procedure are able to give a more stable and accurate low-dimensional representation of the data than principal component analysis (PCA)"
- [readme] Rtsne dependency for paper reproduction: [evidence span withheld — anchor status: external_doc]
