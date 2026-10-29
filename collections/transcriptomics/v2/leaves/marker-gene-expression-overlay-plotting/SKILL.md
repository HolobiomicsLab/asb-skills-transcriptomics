---
name: marker-gene-expression-overlay-plotting
description: Use when you have a learned latent embedding (e.g. scMDC Z from CITE-seq
  data like BMNC or multi-batch SLN111) and need to visually verify cell-type separation,
  marker-gene concordance, and batch-effect removal.
when_to_use_negative:
- You need quantitative clustering accuracy (ARI/NMI) rather than qualitative visual
  inspection — use the clustering metrics path instead
- Data is single-modal scRNA-seq without protein modality or batch structure, so batch-mixing
  overlays have no meaning
- The embedding is not from scMDC (e.g. raw PCA space), where marker overlays will
  not reflect the learned joint latent features
edam_operation: http://edamontology.org/operation_3208
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: scMDC
  role: Produces the latent embedding Z and predicted cluster labels that are visualized
  repo: https://github.com/xianglin226/scMDC
- name: Scanpy
  role: Computes the 2D t-SNE of the latent representation and renders the marker/label/batch
    overlay panels
- name: Seurat
  role: Optional baseline for t-SNE visualization and cluster plotting on RNA/ADT
    assays for comparison
  repo: https://github.com/satijalab/seurat
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-35031-9
    title: Clustering of single-cell multi-omics data with a multimodal deep learning
      method
  source_content_hash: sha256:ba9aad8e7eedd9b6d881b169c10c470d53790a2b94d43e21e67c19b19ee46218
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xianglin226/scMDC
  tools:
  - IDEC
  - SCVIS
  - Scanpy
  - Seurat
  - h5py
  - scMDC
  edam_operation: http://edamontology.org/operation_3208
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# Marker-gene-expression overlay plotting on t-SNE embeddings

## Summary

Projects the scMDC latent representation Z into 2D with t-SNE (Scanpy) and renders multi-panel overlays colored by marker-gene expression (LYZ, CD8A, NKG7), predicted cluster labels, batch IDs, and true cell type labels to visually validate cell-type separation and batch mixing (Figures 5-6 of the scMDC paper).

## When to use

Use when you have a learned latent embedding (e.g. scMDC Z from CITE-seq data like BMNC or multi-batch SLN111) and need to visually verify cell-type separation, marker-gene concordance, and batch-effect removal.

## When NOT to use

- You need quantitative clustering accuracy (ARI/NMI) rather than qualitative visual inspection — use the clustering metrics path instead
- Data is single-modal scRNA-seq without protein modality or batch structure, so batch-mixing overlays have no meaning
- The embedding is not from scMDC (e.g. raw PCA space), where marker overlays will not reflect the learned joint latent features

## Inputs

- scMDC latent embedding Z (saved via --embedding_file)
- Predicted cluster labels (saved via --prediction_file)
- H5 file with X1 (RNA), X2 (ADT), Y (cell type labels), Batch structures for BMNC (GSE128639) and SLN111 (GSE150599)
- Marker gene expression values (LYZ, CD8A, NKG7) per cell

## Outputs

- 2D t-SNE embedding of the scMDC latent space (Scanpy)
- Multi-panel t-SNE overlays colored by marker-gene expression (LYZ, CD8A, NKG7), predicted cluster labels, batch IDs, and true cell type labels (Figure 5-6 style)

## How to apply

First run scMDC (PyTorch) on the H5 input (X1 RNA, X2 ADT, Y cell type labels, Batch structures) with CITE-Seq defaults — encodeLayer [256,64,32,16], decodeLayer1 [16,64,256], decodeLayer2 [16,20], pretrain_epochs=400, maxiter=10000, gamma=0.1, update_interval=1, tol=0.001, sigma1=2.5, sigma2=1.5 — enabling --embedding_file and --prediction_file to save the latent representation and predicted labels. Embed the latent Z in 2D with t-SNE using Scanpy, then color the embedding in separate overlay panels by canonical CITE-seq markers (LYZ for monocytes, CD8A for CD8 T cells, NKG7 for NK cells), the predicted cluster labels, the batch IDs, and the true cell type labels, reproducing the panel layouts of Figures 5-6. The marker panels confirm clusters express the expected markers, the batch panel confirms batch mixing within clusters, and the true-label panel supports qualitative cluster concordance.

## Related tools

- **scMDC** (Produces the latent embedding Z and predicted cluster labels that are visualized) — https://github.com/xianglin226/scMDC
- **Scanpy** (Computes the 2D t-SNE of the latent representation and renders the marker/label/batch overlay panels)
- **Seurat** (Optional baseline for t-SNE visualization and cluster plotting on RNA/ADT assays for comparison) — https://github.com/satijalab/seurat

## Examples

```
sc.pl.tsne(adata, color=['LYZ','CD8A','NKG7','pred_labels','Batch','true_labels'])
```

## Evaluation signals

- The embedding file and prediction file from scMDC were both written (--embedding_file, --prediction_file)
- The t-SNE plot exists and renders all required overlay panels: marker expression (LYZ, CD8A, NKG7), predicted labels, batch IDs, true labels
- Marker panels show expected concordance: LYZ high in monocyte clusters, CD8A high in CD8 T-cell clusters, NKG7 high in NK clusters
- Batch-ID panel shows cells from the two SLN111 batches mixed within most clusters rather than separated
- Predicted cluster layout qualitatively matches the published Figure 5-6 layouts and the true cell-type labels

## Limitations

- Cluster concordance with true labels is checked qualitatively against the published figure, not with quantitative metrics in this workflow
- The scMDC embedding step requires GPU hardware (Nvidia Tesla P100 16 G in the study) and the number of clusters K must be specified manually
- Figure reproduction depends on Scanpy t-SNE, which is stochastic unless random state is fixed; exact point positions will differ run to run

## Evidence

- [other] Color the t-SNE embedding by marker-gene expression (LYZ, CD8A, NKG7), predicted cluster labels, batch IDs, and true cell type labels: [evidence span withheld — anchor status: not_found]
- [results] scMDC separates most cell types in the latent space while mixing cells from the two batches in most clusters: [evidence span withheld — anchor status: not_found]
- [discussion] The raw CITE-seq data is preprocessed and normalized by the Python package SCANPY: "The raw CITE-seq data is preprocessed and normalized by the Python package SCANPY48."
- [readme] Plot clusters on RNA and ADT t-SNE.: "### Plot clusters on RNA and ADT t-SNE. "
