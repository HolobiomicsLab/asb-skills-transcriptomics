---
name: tsne-visualization-of-latent-embeddings
description: Use when a high-dimensional scRNA-seq embedding (e.g. a 20-dimensional
  scGAE latent space) must be rendered in 2D to visually verify cluster separation
  or resolve subpopulations before/after re-clustering.
when_to_use_negative:
- The input is already a 2D embedding or the goal is direct 2D output from the graph
  autoencoder — the paper notes complex data structure is hard to be directly embedded
  into two-dimensional space by the graph autoencoder.
- The analysis requires faithful preservation of global/inter-cluster distances in
  the 2D view itself — the authors note a better visualization method than t-SNE is
  still needed to preserve topological structure.
edam_operation: http://edamontology.org/operation_0337
tools:
- name: scGAE
  role: Produces the intermediate-dimensional (e.g. 20D) cell embedding that is then
    projected to 2D with t-SNE
  repo: github.com/ZixiangLuo1161/scGAE
- name: t-SNE
  role: Visualizes the scGAE latent embedding in a two-dimensional space for cluster/subpopulation
    inspection
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-021-99003-7
    title: A topology-preserving dimensionality reduction method for single-cell RNA-seq
      data using graph autoencoder
  source_content_hash: sha256:b58ff792d69c62e5ca88adefa3ce9d5d637eb1d0b407b2be55356de2fa8777ff
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - scGAE
  - t-SNE
  edam_operation: http://edamontology.org/operation_0337
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# tsne-visualization-of-latent-embeddings

## Summary

Two-dimensional visualization of a scGAE latent embedding via t-SNE, following the paper's convention of first embedding scRNA-seq data into an intermediate dimension (e.g. 20D) and then projecting to 2D with t-SNE to inspect cell clusters and subpopulations such as the Ba/Eo/MaP subclusters.

## When to use

Use when a high-dimensional scRNA-seq embedding (e.g. a 20-dimensional scGAE latent space) must be rendered in 2D to visually verify cluster separation or resolve subpopulations before/after re-clustering.

## When NOT to use

- The input is already a 2D embedding or the goal is direct 2D output from the graph autoencoder — the paper notes complex data structure is hard to be directly embedded into two-dimensional space by the graph autoencoder.
- The analysis requires faithful preservation of global/inter-cluster distances in the 2D view itself — the authors note a better visualization method than t-SNE is still needed to preserve topological structure.

## Inputs

- Preprocessed scRNA-seq count matrix (e.g. NGDC HRA000084 HSPC data, preprocessed with scTransform in the Seurat package)
- Intermediate-dimensional latent embedding from scGAE (e.g. 20-dimensional), produced by model.embedding()

## Outputs

- Two-dimensional t-SNE coordinates/plot of the scGAE latent embedding
- Subpopulation-label table for re-clustered cells (e.g. >=2 subclusters resolved within Ba/Eo/MaP, matching Fig. 4a,b)
- Saved embedding (h5ad/CSV) for downstream traceability

## How to apply

Run the scGAE pipeline (per Run_scGAE.ipynb: get_adj, model.train, optional clustering_train with Louvain-derived centers via get_centers_louvain) and obtain the latent embedding with model.embedding(). Do not attempt to make the graph autoencoder output 2D directly — the paper states complex data structure is hard to directly embed into two dimensions, so t-SNE is applied to the intermediate-dimension embedding. Visualize the 2D coordinates, then re-cluster annotated cell sets (e.g. the Basophil/Eosinophil/Mast progenitor cells from NGDC HRA000084) using Louvain clustering on the scGAE embedding and inspect the t-SNE plot for resolved subclusters. Export the 2D coordinates alongside the latent embedding (h5ad/CSV) and a subpopulation-label table so the plot can be traced to labels.

## Related tools

- **scGAE** (Produces the intermediate-dimensional (e.g. 20D) cell embedding that is then projected to 2D with t-SNE) — github.com/ZixiangLuo1161/scGAE
- **t-SNE** (Visualizes the scGAE latent embedding in a two-dimensional space for cluster/subpopulation inspection)

## Examples

```
emb = model.embedding(); sc.tl.tsne(emb, n_components=2)
```

## Evaluation signals

- The t-SNE input is the intermediate-dimensional scGAE embedding (e.g. 20D), not the raw count matrix or the autoencoder output forced to 2D.
- The 2D plot separates the known cell populations and resolves at least 2 subclusters within the previously annotated Ba/Eo/MaP population, matching the structure in Fig. 4a,b.
- Louvain re-clustering of annotated Ba/Eo/MaP cells on the scGAE embedding yields subpopulation labels consistent with the visualized t-SNE layout.
- The latent embedding and 2D coordinates are exported (h5ad/CSV) together with a subpopulation-label table so plot points trace to labels.
- Running times/memory are in the range reported for scGAE vs. scGNN (scGAE much shorter running time), indicating the correct pipeline (not a heavier competitor) generated the embedding.

## Limitations

- t-SNE does not fully preserve the topological structure of scRNA-seq data; the authors state a better visualization method is needed.
- The graph autoencoder cannot directly embed complex data structure into two-dimensional space, so an intermediate dimension plus t-SNE is used instead.
- The optimal K for the KNN cell graph varies among datasets and even across parts of a dataset, affecting the embedding that t-SNE visualizes; no random seed or stochastic-variation control is reported.

## Evidence

- [discussion] Embed scRNA-seq data into intermediate dimension then use tSNE for 2D visualization: "we embedded the scRNA-seq data into an intermediate dimension and used tSNE to visualize the embedded data into a two-dimensional space."
- [discussion] Direct 2D embedding by the GAE is not feasible: "because the complex data structure is hard to be directly embedded into two-dimensional space by graph autoencoder"
- [intro] t-SNE is a conventional dimensionality reduction technique for scRNA-seq: "Conventional dimensionality reduction techniques such as principal component analysis (PCA) and t-Distributed Stochastic Neighbor Embedding (t-SNE)"
- [readme] Embedding generation step from the README: "Get the latent embedding of the data by model.embedding()"
