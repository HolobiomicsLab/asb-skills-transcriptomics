---
name: tsne-embedding-of-single-cell-data
description: Use when a high-dimensional single-cell feature matrix (e.g., latent
  representations from Miscell, Seurat, Scanpy, or scVI) needs a fast 2D embedding
  for visualization and downstream clustering on datasets up to ~55,656 cells.
when_to_use_negative:
- The input is already a 2D embedding or cluster labels are already available — embedding
  again adds no information.
- Subtle small-cluster detail is required and the multi-scale Annoy-based near-neighbor
  approximation is insufficient; the FIt-SNE README notes vantage-point trees should
  be used when 'subtle detail is required (e.g. in identifying small clusters)'.
- Batch effects are present and uncorrected — Miscell leaves batch correction as a
  preprocessing step, so embedding uncorrected features can confound clusters.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: FIt-SNE
  role: FFT-accelerated interpolation-based t-SNE used to embed single-cell feature
    matrices into 2D via fast_tsne(features, seed=123, nthreads=12, perplexity_list=[30,
    36, 42, 48])
  repo: https://github.com/KlugerLab/FIt-SNE
- name: Miscell
  role: Self-supervised deep neural network encoder that produces the latent feature
    matrix fed into the t-SNE embedding
  repo: https://github.com/lixiangchun/Miscell
- name: Scanpy
  role: Builds the neighborhood graph (sc.pp.neighbors, n_neighbors=10, use_rep='X'),
    runs Leiden clustering, and renders the t-SNE plot colored by leiden clusters
- name: scikit-learn
  role: Provides sklearn.cluster.dbscan for clustering the 2D t-SNE embedding (eps=1,
    min_samples=5, algorithm='auto')
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.isci.2021.103200
    title: 'Miscell: An efficient self-supervised learning approach for dissecting
      single-cell transcriptome'
  source_content_hash: sha256:f25193eda8b4fd6e8200e34c2b7c6d987e21ba770b8e778a3c7289ae9ecfa5a0
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/lixiangchun/Miscell
  repo_url: https://github.com/KlugerLab/FIt-SNE
  tools:
  - FIt-SNE
  - Miscell
  - Scanpy
  - Seurat
  - scVI
  - scikit-learn
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# tsne-embedding-of-single-cell-data

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compute 2D t-SNE embeddings of single-cell latent feature matrices using FIt-SNE (FFT-accelerated, interpolation-based t-SNE) with multi-scale perplexities, then cluster and visualize the embedding for benchmarking single-cell clustering methods.

## When to use

Use when a high-dimensional single-cell feature matrix (e.g., latent representations from Miscell, Seurat, Scanpy, or scVI) needs a fast 2D embedding for visualization and downstream clustering on datasets up to ~55,656 cells.

## When NOT to use

- The input is already a 2D embedding or cluster labels are already available — embedding again adds no information.
- Subtle small-cluster detail is required and the multi-scale Annoy-based near-neighbor approximation is insufficient; the FIt-SNE README notes vantage-point trees should be used when 'subtle detail is required (e.g. in identifying small clusters)'.
- Batch effects are present and uncorrected — Miscell leaves batch correction as a preprocessing step, so embedding uncorrected features can confound clusters.

## Inputs

- High-dimensional single-cell feature matrix (e.g., npz file data/x_new_scanpy_HVG.npz with key 'x', in_features=1686, or latent features from Miscell/Seurat/Scanpy/scVI)
- Trained model checkpoint for feature extraction (e.g., checkpoint_0199.pth.tar) when embeddings come from Miscell

## Outputs

- 2D t-SNE embedding array (tsne) with one (x, y) coordinate per cell
- DBSCAN cluster assignments (y) computed on the 2D embedding
- AnnData object with obsm['X_tsne'] populated and leiden cluster labels
- t-SNE visualization plot (e.g., figures/tsne_scanpy.pdf)

## How to apply

Take the feature matrix (e.g., features extracted from the Miscell pretrained densenet encoder, or latent spaces from competing methods) and run FIt-SNE with seed=123, nthreads=12, and perplexity_list=[30, 36, 42, 48]; the multi-scale perplexity combination captures both local and global structure. Import the Python wrapper fast_tsne after adding the FIt-SNE repo path to sys.path. On the resulting 2D embedding, cluster with DBSCAN (eps=1, min_samples=5, algorithm='auto') or build a Scanpy AnnData (storing the embedding in adata.obsm['X_tsne']), build a neighborhood graph with sc.pp.neighbors(n_neighbors=10, use_rep='X'), run Leiden clustering, and visualize with sc.pl.tsne colored by the leiden clusters. FIt-SNE is preferred over Barnes-Hut t-SNE because FFT-accelerated interpolation of the N-body computation and Annoy-approximated nearest neighbors dramatically reduce runtime on large single-cell datasets.

## Related tools

- **FIt-SNE** (FFT-accelerated interpolation-based t-SNE used to embed single-cell feature matrices into 2D via fast_tsne(features, seed=123, nthreads=12, perplexity_list=[30, 36, 42, 48])) — https://github.com/KlugerLab/FIt-SNE
- **Miscell** (Self-supervised deep neural network encoder that produces the latent feature matrix fed into the t-SNE embedding) — https://github.com/lixiangchun/Miscell
- **Scanpy** (Builds the neighborhood graph (sc.pp.neighbors, n_neighbors=10, use_rep='X'), runs Leiden clustering, and renders the t-SNE plot colored by leiden clusters)
- **scikit-learn** (Provides sklearn.cluster.dbscan for clustering the 2D t-SNE embedding (eps=1, min_samples=5, algorithm='auto'))

## Examples

```
tsne = fast_tsne(features, seed=123, nthreads=12, perplexity_list=[30, 36, 42, 48]); _, y = sklearn.cluster.dbscan(tsne, eps=1, min_samples=5, algorithm='auto')
```

## Evaluation signals

- The fast_tsne call uses exactly seed=123, nthreads=12, and perplexity_list=[30, 36, 42, 48]; deviation from these parameters breaks comparability with the published Miscell benchmark.
- The DBSCAN call uses eps=1, min_samples=5, algorithm='auto'; verify these thresholds before computing clustering metrics.
- The embedding has exactly 2 dimensions with one point per input cell (no cells dropped between feature extraction and embedding).
- In the Scanpy workflow, adata.obsm['X_tsne'] is set before sc.pl.tsne, and the plot is saved with save='_scanpy.pdf'.
- Downstream validation: metrics (ARI, NMI, V-measure) computed from clusters on this embedding against reference labels must reproduce the reported Miscell result of a 10% ARI gain over the second-best method on the 55,656-cell Tabula Muris dataset with comparable NMI and V-measure.

## Limitations

- FIt-SNE's Annoy-based approximate nearest neighbors use 'near' rather than strictly 'nearest' neighbors, which is faster but can smooth over subtle structure; use vantage-point trees if identifying small clusters.
- If updating FIt-SNE with git pull, the C++ code must be recompiled before use.
- Miscell-based feature extraction demands a large sample size for good performance, limiting embedding quality on small datasets.
- Batch effects must be corrected before feature extraction/embedding, since Miscell does not include internal batch-correction.

## Evidence

- [readme] FIt-SNE acceleration rationale: [evidence span withheld — anchor status: external_doc]
- [readme] Multi-scale perplexity combination: [evidence span withheld — anchor status: external_doc]
