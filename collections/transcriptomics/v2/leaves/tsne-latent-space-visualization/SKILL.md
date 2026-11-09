---
name: tsne-latent-space-visualization
description: Use when a learned single-cell latent embedding (e.g., from scMDC on
  CITE-seq RNA+ADT data) must be visually assessed for cell-type separation and batch-effect
  removal before/instead of relying solely on quantitative clustering metrics.
when_to_use_negative:
- Input is raw counts with no learned embedding available — preprocessing and scMDC
  clustering must run first; t-SNE on the latent space is the visualization step,
  not the clustering step
- Quantitative validation (e.g., ARI/NMI against true labels) is required as the deliverable
  — t-SNE panels only support qualitative concordance checks against published figures
- Single-modal data or methods without a batch structure; the batch-mixing overlay
  assessment assumes multi-batch multimodal input
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3941
- http://edamontology.org/topic_3170
tools:
- name: scMDC
  role: Produces the latent embedding Z and predicted cluster labels that are visualized
    by t-SNE
- name: Scanpy
  role: Computes the 2D t-SNE embedding of the latent representation and renders overlay
    panels
- name: h5py
  role: Reads the X1/X2/Y/Batch structures from the H5 input files
- name: Pytorch
  role: Runs scMDC with CITE-Seq default hyperparameters to generate embeddings
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-022-35031-9
    title: Clustering of single-cell multi-omics data with a multimodal deep learning
      method
  source_content_hash: sha256:e245fe5d57154be9bcf65fd4753f75c7be2a9874734e367583b7ccb9cb435db0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - IDEC
  - Monocle
  - Pytorch
  - R
  - Rtsne
  - SCVIS
  - Scanpy
  - Seurat
  - TCM
  - TSCAN
  - Wishbone
  - devtools
  - diffusionMap
  - h5py
  - scDatasets
  - scMDC
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3941
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
merged_aliases:
- latent-space-visualization
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 14
---

# tsne-latent-space-visualization

## Summary

Embed the scMDC learned latent representation Z in 2D with t-SNE (Scanpy) and overlay marker-gene expression, predicted cluster labels, batch IDs, and true cell-type labels to visually verify cell-type separation and batch mixing, as done for BMNC and multi-batch SLN111 in Figures 5-6.

## When to use

Use when a learned single-cell latent embedding (e.g., from scMDC on CITE-seq RNA+ADT data) must be visually assessed for cell-type separation and batch-effect removal before/instead of relying solely on quantitative clustering metrics.

## When NOT to use

- Input is raw counts with no learned embedding available — preprocessing and scMDC clustering must run first; t-SNE on the latent space is the visualization step, not the clustering step
- Quantitative validation (e.g., ARI/NMI against true labels) is required as the deliverable — t-SNE panels only support qualitative concordance checks against published figures
- Single-modal data or methods without a batch structure; the batch-mixing overlay assessment assumes multi-batch multimodal input

## Inputs

- H5 file with X1 (RNA counts), X2 (ADT counts), Y (cell-type labels), and Batch structures (e.g., BMNC / SLN111)
- scMDC latent representation Z from --embedding_file output
- scMDC predicted cluster labels from --prediction_file output

## Outputs

- 2D t-SNE embedding of the scMDC latent space (Scanpy AnnData / plot)
- Overlay figure panels colored by marker expression (LYZ, CD8A, NKG7), predicted labels, batch IDs, and true labels

## How to apply

Load the CITE-seq input from H5 files with h5py (X1 RNA, X2 ADT, Y cell-type labels, Batch), then run scMDC with PyTorch using CITE-Seq defaults (encodeLayer [256,64,32,16], decodeLayer1 [16,64,256], decodeLayer2 [16,20], pretrain_epochs=400, maxiter=10000, gamma=0.1, update_interval=1, tol=0.001, sigma1=2.5, sigma2=1.5) with the --embedding_file and --prediction_file flags enabled so the latent Z and predicted labels are written to disk. Embed Z in 2D with t-SNE via Scanpy, then produce overlay panels coloring cells by marker-gene expression (LYZ, CD8A, NKG7), predicted cluster labels, batch IDs, and true cell-type labels, reproducing the panel layouts of Figures 5-6. The rationale is that scMDC is the only method claimed to achieve both superior cell-type partition and batch-effect removal, and t-SNE panels let a reviewer confirm most cell types separate while cells from different batches mix within clusters.

## Related tools

- **scMDC** (Produces the latent embedding Z and predicted cluster labels that are visualized by t-SNE)
- **Scanpy** (Computes the 2D t-SNE embedding of the latent representation and renders overlay panels)
- **h5py** (Reads the X1/X2/Y/Batch structures from the H5 input files)
- **Pytorch** (Runs scMDC with CITE-Seq default hyperparameters to generate embeddings)

## Examples

```
python run_scMDC.py --input sln111.h5 --embedding_file Z.npy --prediction_file labels.npy; then in Scanpy: sc.tl.tsne(adata_latent, use_rep='Z') and color by ['LYZ','CD8A','NKG7','pred','batch','Y']
```

## Evaluation signals

- The --embedding_file and --prediction_file outputs were actually written by the scMDC run
- The t-SNE plot renders all required overlay panels: marker expression (LYZ, CD8A, NKG7), predicted labels, batch IDs, and true cell-type labels
- Most cell types appear spatially separated in the 2D t-SNE of the latent space
- Cells from the two batches appear mixed within most clusters (batch-effect removal check on SLN111)
- Cluster layout qualitatively matches the published Figures 5-6 panels

## Limitations

- Cluster concordance with true labels is validated qualitatively against the published figure rather than by a quantitative metric in this workflow
- All scMDC experiments in the study were conducted on an Nvidia Tesla P100 (16 G) GPU, which may limit reproducibility on CPU-only systems
- The number of clusters K must be specified by the user; no automatic estimation is provided, so a wrong K will distort the visual separation pattern

## Evidence

- [other] On the multi-batch SLN111 dataset (and BMNC), scMDC separates most cell types in the latent space while mixing cells from the two batches: [evidence span withheld — anchor status: not_found]
- [methods] Load H5 structures with h5py: "1) Prepare the input data in h5 format. (See readme in 'dataset' folder)"
- [methods] save embedding and predictions: [evidence span withheld — anchor status: not_found]
- [other] Embed Z in 2D with t-SNE using Scanpy: [evidence span withheld — anchor status: not_found]
- [discussion] Users must specify K: "After pretraining, the users need to specify the number of clusters (K)."
- [discussion] GPU requirement: "All experiments of scMDC in this study are conducted on Nvidia Tesla P100 (16 G) GPU."
