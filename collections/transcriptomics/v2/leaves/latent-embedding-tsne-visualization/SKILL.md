---
name: latent-embedding-tsne-visualization
description: Use when you have per-cell latent feature vectors (e.g. 64-dim Miscell/MoCo
  embeddings) and need a 2D t-SNE visualization plus clustering assignments for a
  large scRNA-seq dataset such as the 43,095-cell 10x PBMC data.
when_to_use_negative:
- Input is raw gene expression or already-processed Scanpy/Seurat PCA embeddings rather
  than Miscell latent features — the article benchmarks Miscell against those pipelines
  instead of reusing them
- 'The dataset is small: Miscell demands a large sample size to obtain good performance,
  so t-SNE on its latent features is not the right choice for small cohorts'
- Batch effects are present and uncorrected — Miscell includes no internal batch correction,
  so embedding uncorrected features is inappropriate
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Miscell
  role: Self-supervised model whose latent feature encoder produces the features embedded
    with t-SNE
  repo: github.com/lixiangchun/Miscell
- name: FIt-SNE
  role: FFT-accelerated t-SNE used to embed latent features into 2D with fixed seed
    and multi-scale perplexities
  repo: github.com/KlugerLab/FIt-SNE
- name: Scanpy
  role: Builds a 10-neighbor graph on latent features and runs Leiden clustering;
    also used for visualization via sc.pl.tsne
- name: scikit-learn
  role: Provides sklearn.cluster.dbscan used to cluster the 2D t-SNE embedding with
    eps=1, min_samples=5
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.isci.2021.103200
    title: 'Miscell: An efficient self-supervised learning approach for dissecting
      single-cell transcriptome'
  source_content_hash: sha256:281729c42ed933908a9c958cb372713821ad6bc2023063a85e3f8206a9504979
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/lixiangchun/Miscell
  tools:
  - FIt-SNE
  - Miscell
  - Scanpy
  - Seurat
  - scVI
  - scikit-learn
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# latent-embedding-tsne-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Embed cell latent features learned by a self-supervised model (Miscell) into a 2D FIt-SNE visualization and cluster the embedding with DBSCAN or a Scanpy Leiden graph, enabling visual delineation of single-cell clusters.

## When to use

Use when you have per-cell latent feature vectors (e.g. 64-dim Miscell/MoCo embeddings) and need a 2D t-SNE visualization plus clustering assignments for a large scRNA-seq dataset such as the 43,095-cell 10x PBMC data.

## When NOT to use

- Input is raw gene expression or already-processed Scanpy/Seurat PCA embeddings rather than Miscell latent features — the article benchmarks Miscell against those pipelines instead of reusing them
- The dataset is small: Miscell demands a large sample size to obtain good performance, so t-SNE on its latent features is not the right choice for small cohorts
- Batch effects are present and uncorrected — Miscell includes no internal batch correction, so embedding uncorrected features is inappropriate

## Inputs

- Per-cell latent feature matrix extracted from the trained Miscell model (numpy array, e.g. from checkpoint_0199.pth.tar with in_features matching the input HVG count such as 1686)
- FIt-SNE installation (compiled fast_tsne binary with FFTW) and its Python wrapper fast_tsne.py
- AnnData-compatible environment (scanpy, anndata) for the Leiden alternative

## Outputs

- 2D t-SNE coordinates of all cells (numpy array 'tsne') computed with seed=123 and perplexity_list=[30, 36, 42, 48]
- DBSCAN cluster labels (eps=1, min_samples=5) for each cell
- Leiden cluster assignments on a 10-neighbor graph built from the latent features
- t-SNE plot colored by Leiden clusters saved as t-SNE_scanpy.pdf

## How to apply

Run FIt-SNE on the extracted latent features using a fixed seed and multi-scale perplexities — fast_tsne(features, seed=123, nthreads=12, perplexity_list=[30, 36, 42, 48]) — because combining several perplexity values yields a multi-scale embedding that reveals both local and global structure. Cluster the 2D embedding directly with DBSCAN (sklearn.cluster.dbscan with eps=1, min_samples=5). Alternatively, wrap the features in an AnnData object, store the t-SNE coordinates in obsm['X_tsne'], build a neighborhood graph with sc.pp.neighbors(adata, n_neighbors=10, use_rep='X') on the latent representation, run sc.tl.leiden, and save the t-SNE plot colored by Leiden clusters (t-SNE_scanpy.pdf). The rationale for using the learned latent space rather than raw counts is that Miscell's contrastively trained encoder produces cell-type invariant representations that separate clusters more cleanly (ARI +33% vs the second-best method on PBMC).

## Related tools

- **Miscell** (Self-supervised model whose latent feature encoder produces the features embedded with t-SNE) — github.com/lixiangchun/Miscell
- **FIt-SNE** (FFT-accelerated t-SNE used to embed latent features into 2D with fixed seed and multi-scale perplexities) — github.com/KlugerLab/FIt-SNE
- **Scanpy** (Builds a 10-neighbor graph on latent features and runs Leiden clustering; also used for visualization via sc.pl.tsne)
- **scikit-learn** (Provides sklearn.cluster.dbscan used to cluster the 2D t-SNE embedding with eps=1, min_samples=5)

## Examples

```
tsne = fast_tsne(features, seed=123, nthreads=12, perplexity_list=[30, 36, 42, 48]); _, y = sklearn.cluster.dbscan(tsne, eps=1, min_samples=5)
```

## Evaluation signals

- t-SNE call uses seed=123, nthreads=12 and perplexity_list=[30, 36, 42, 48] so results are reproducible
- DBSCAN clustering of the embedding runs with eps=1 and min_samples=5; labels should be non-degenerate (multiple clusters found on the 43,095-cell PBMC data)
- Scanpy variant uses use_rep='X' (the latent features) with n_neighbors=10 before sc.tl.leiden, not default PCA
- Plot file t-SNE_scanpy.pdf is generated with cells colored by leiden clusters
- Downstream check: clustering metrics (ARI, NMI, V-measure) of the assignments against reference cell-type labels should reflect Miscell's reported gains (ARI +33%, NMI +6%, V-measure +7% over second-best)

## Limitations

- Miscell demands a large sample size to obtain good performance, limiting this embedding approach on small datasets
- The latent features lack explainability; the biological mechanisms underlying them remain unknown
- Miscell does not internally include any batch-correction method — correct batch effects before training/embedding
- DBSCAN (eps=1, min_samples=5) on t-SNE coordinates is parameter-sensitive; Scanpy Leiden is offered as an alternative clustering path

## Evidence
