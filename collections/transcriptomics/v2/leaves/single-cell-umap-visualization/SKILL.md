---
name: single-cell-umap-visualization
description: Use when a trained single-cell multi-omics embedding (e.g., Mowgli latent
  dimensions on TEA-seq, PBMC 10X, OP Multiome, OP CITE, or BM CITE data) needs 2D
  visualization to inspect cell-type separation, compare methods, or overlay factor
  weights.
when_to_use_negative:
- The method under benchmarking provides no cell embedding (e.g., Seurat v4 in this
  paper), so no UMAP can be computed or fairly compared
- The data are unpaired multi-omics or batch-confounded; Mowgli (and thus its embedding
  for UMAP) assumes paired profiles and does not handle batch correction
- The goal is a quantitative clustering comparison rather than visual inspection —
  use silhouette score, ARI, and purity score directly
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3308
tools:
- name: Scanpy
  role: Computes the kNN graph (k=20, Euclidean) on the Mowgli embedding, runs sc.tl.umap,
    and renders UMAP scatter and factor-weighted interpretation panels
  repo: https://github.com/cantinilab/mowgli
- name: Muon
  role: Holds the multi-omics MuData object (RNA, ATAC, ADT modalities) whose Mowgli
    latent space is projected onto UMAP
  repo: https://github.com/cantinilab/mowgli
- name: Mowgli
  role: Provides the shared embedding matrix (W) across RNA/ATAC/ADT modalities that
    is visualized with UMAP
  repo: https://github.com/cantinilab/mowgli
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-43019-2
    title: Paired single-cell multi-omics data integration with Mowgli
  source_content_hash: sha256:44e173e42bec086f07b96e8a5f5bfe58f75966b9ae08153ad6bd833ef871daa0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/cantinilab/mowgli
  tools:
  - Azimuth
  - Human Protein Atlas
  - Mowgli
  - Muon
  - Python
  - Scanpy
  - Signac
  - gProfiler
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# single-cell-umap-visualization

## Summary

Compute and render UMAP visualizations of cell embeddings produced by paired multi-omics integration methods (Mowgli, integrative NMF, MOFA+) within the scverse/Scanpy ecosystem, used both for qualitative benchmarking across datasets and for factor-weighted biological interpretation panels (e.g., TEA-seq Fig. 5).

## When to use

Use when a trained single-cell multi-omics embedding (e.g., Mowgli latent dimensions on TEA-seq, PBMC 10X, OP Multiome, OP CITE, or BM CITE data) needs 2D visualization to inspect cell-type separation, compare methods, or overlay factor weights.

## When NOT to use

- The method under benchmarking provides no cell embedding (e.g., Seurat v4 in this paper), so no UMAP can be computed or fairly compared
- The data are unpaired multi-omics or batch-confounded; Mowgli (and thus its embedding for UMAP) assumes paired profiles and does not handle batch correction
- The goal is a quantitative clustering comparison rather than visual inspection — use silhouette score, ARI, and purity score directly

## Inputs

- Trained multi-omics embedding matrix (cells x latent dimensions) from Mowgli (W matrix), integrative NMF, or MOFA+
- Muon/AnnData (MuData) object with per-modality slots (RNA, ATAC, ADT) and cell metadata
- Cell-type annotations from marker-based Leiden clustering or Azimuth reference mapping
- Per-factor cell weights (rows of H normalized to 1) for factor-weighted UMAP panels

## Outputs

- 2D UMAP coordinates stored in the Muon/AnnData object (obsm['X_umap'])
- UMAP scatter plots colored by method, dataset, Leiden cluster, or cell type
- Factor-weighted UMAP interpretation panels reproducing Fig. 5 of the Mowgli paper

## How to apply

Build a kNN graph on the method's embedding (for Mowgli interpretation workflows: k=20 neighbors, Euclidean distance) with Scanpy's neighbors function, then project with UMAP and color by Leiden clusters or cell-type annotations derived from canonical gene/protein markers cross-confirmed with the Azimuth PBMC reference. In the Mowgli benchmark, UMAPs were computed for every combination of method (Mowgli, Seurat v4, MOFA+, integrative NMF, Cobolt, Multigrate) and dataset to visually complement silhouette score, ARI, and purity score metrics. For interpretability panels, render the factor-weighted UMAP by overlaying each factor's cell-level weights (H(rna) rows normalized to 1) so that one-to-one factor–cell-type associations (e.g., specific factors for NK, CD8 T-, and CD4 T-cells) can be visually verified. Note that silhouette scores and embeddings cannot be computed for methods like Seurat that do not output an embedding, so UMAP comparison is restricted to embedding-based methods.

## Related tools

- **Scanpy** (Computes the kNN graph (k=20, Euclidean) on the Mowgli embedding, runs sc.tl.umap, and renders UMAP scatter and factor-weighted interpretation panels) — https://github.com/cantinilab/mowgli
- **Muon** (Holds the multi-omics MuData object (RNA, ATAC, ADT modalities) whose Mowgli latent space is projected onto UMAP) — https://github.com/cantinilab/mowgli
- **Mowgli** (Provides the shared embedding matrix (W) across RNA/ATAC/ADT modalities that is visualized with UMAP) — https://github.com/cantinilab/mowgli

## Examples

```
sc.pp.neighbors(mdata, use_rep='mowgli', n_neighbors=20); sc.tl.umap(mdata); sc.pl.umap(mdata, color='leiden')
```

## Evaluation signals

- UMAP of the Mowgli embedding on TEA-seq should show Leiden clusters (k=20 graph, coarse resolution) matching cell types assigned from canonical gene/protein markers and the Azimuth PBMC reference
- Factor-weighted UMAP should display spatially restricted weight distributions consistent with one-to-one factor–cell-type associations (high specificity score (a_i - b_i)/max_j a_j) rather than diffuse weights across many cell types
- Across the benchmark (Liu/scCAT-seq, PBMC 10X, OP Multiome, OP CITE, BM CITE), Mowgli UMAPs should visually separate populations that MOFA+, Seurat v4, Cobolt, and Multigrate confuse (e.g., the 96% sparse three-population dataset)
- UMAP must be computed from the method's own embedding; if the method supplies no embedding (Seurat), the plot is impossible and the benchmark comparison must be flagged

## Limitations

- UMAP is a qualitative, non-metric visualization; in the Mowgli paper it complements, but does not replace, silhouette score, ARI, and purity score benchmarks
- Mowgli does not offer a straightforward approach to defining the number of latent dimensions (k); the UMAP appearance can change substantially with the chosen k
- Optimal transport embeddings are expensive to compute and require GPU availability to scale to larger datasets like BM CITE (29,803 cells), limiting how readily UMAPs can be regenerated

## Evidence

- [results] Computed UMAP visualizations for the benchmarked methods and datasets: "In addition, we computed UMAP visualiza- tions for the different methods and datasets"
- [results] Embedding-based UMAP and clustering on TEA-seq with marker-based annotation: "we separately clustered the embeddings obtained from Mowgli, inte- grative NMF, and MOFA+ and annotated them based on gene and protein markers"
- [results] Factor distribution visualization by mean weight within/outside cell types: [evidence span withheld — anchor status: not_found]
- [readme] scverse integration enabling Scanpy/Muon-based UMAP workflows: "Mowgli is implemented as a Python package seamlessly integrated within the scverse ecosystem, in particular Muon and Scanpy."
