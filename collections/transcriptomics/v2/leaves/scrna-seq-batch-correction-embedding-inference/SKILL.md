---
name: scrna-seq-batch-correction-embedding-inference
description: Use when multiple scRNA-seq batches or studies (e.g., 9-batch pancreas
  atlas, immune or lung atlas datasets) need batch-corrected embeddings, or when new
  query datasets must be annotated against a pre-trained reference atlas without retraining.
when_to_use_negative:
- The reference and query datasets come from different tissue types or lack similar
  cell types — the article states label transfer requires the same tissue type or
  similar cell types for optimal performance
- The data are CITE-seq protein measurements with only a few hundred features, where
  scCobra's shared-feature encoder requirement makes optimization difficult
- The goal is scATAC-seq integration where peak-to-gene-activity conversion (e.g.,
  via MAESTRO) may cause information loss
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scCobra
  role: Contrastive deep generative encoder with domain adaptation for batch-corrected
    embeddings and online label transfer
  repo: github.com/mcgilldinglab/scCobra
- name: Scanpy
  role: Cell/gene filtering, count normalization to 10,000 with log1p transformation,
    and 2000 highly variable gene selection
- name: PyTorch
  role: Backend for training the scCobra neural encoder under the three-phase optimization
    protocol
- name: scIB
  role: Leiden clustering (resolutions 0.1–2.0 in 0.1 increments) and integration
    metric computation on embeddings
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s42003-025-07692-x
    title: scCobra allows contrastive cell embedding learning with domain adaptation
      for single cell data integration and harmonization
  source_content_hash: sha256:001e044e7d0c80bf0797905336058ee1fd350523eeebfc8a1c810d49d54faa4e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PyTorch
  - Scanpy
  - scCobra
  - scIB
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# scRNA-seq-batch-correction-embedding-inference

## Summary

Train the scCobra contrastive deep generative encoder (with domain adaptation) on a single-cell RNA-seq reference atlas to obtain batch-corrected cell embeddings, then apply the frozen encoder to query datasets for online label transfer without retraining.

## When to use

Use when multiple scRNA-seq batches or studies (e.g., 9-batch pancreas atlas, immune or lung atlas datasets) need batch-corrected embeddings, or when new query datasets must be annotated against a pre-trained reference atlas without retraining.

## When NOT to use

- The reference and query datasets come from different tissue types or lack similar cell types — the article states label transfer requires the same tissue type or similar cell types for optimal performance
- The data are CITE-seq protein measurements with only a few hundred features, where scCobra's shared-feature encoder requirement makes optimization difficult
- The goal is scATAC-seq integration where peak-to-gene-activity conversion (e.g., via MAESTRO) may cause information loss

## Inputs

- Raw scRNA-seq count matrices for a reference atlas and one or more query datasets (e.g., the pancreas dataset with 16382 cells, 14 cell types, 9 batches packaged with Integration_pancreas_label_transfer.ipynb)
- Reference cell-type labels for training the KNN classifier
- GEO accessioned query datasets (GSE114297, GSE85247, GSE83312) or figshare-packaged simulated/pancreas/immune/lung datasets

## Outputs

- Batch-corrected latent cell embeddings for reference and query datasets
- Per-cell-type label predictions for query cells from the pre-trained KNN classifier
- Weighted F1 scores (target ~0.95, comparable to SCALEX, vs TOSICA 0.94 and scVI 0.92) and a per-cell-type ACC heatmap

## How to apply

Preprocess count matrices with Scanpy: filter cells expressing fewer than 200 genes, filter genes detected in fewer than 3 cells, remove cells with mitochondrial fraction above 5%, normalize counts to 10,000 with a log1p transformation, and select 2000 highly variable genes. Train the scCobra contrastive deep generative encoder on the reference atlas using the three-phase optimization protocol to produce batch-corrected embeddings, then fit a KNN classifier on the reference embeddings using reference cell-type labels. For new query datasets (e.g., GSE114297, GSE85247, GSE83312 pancreas queries), apply the frozen pre-trained encoder to infer latent embeddings with no retraining, and pass query embeddings through the pre-trained KNN classifier to annotate cells. The rationale is that scCobra avoids assuming a gene expression distribution (unlike scVI/scDREAMER) and avoids MNN-style over-correction, while its domain-adaptation design supports continuous integration of new data.

## Related tools

- **scCobra** (Contrastive deep generative encoder with domain adaptation for batch-corrected embeddings and online label transfer) — github.com/mcgilldinglab/scCobra
- **Scanpy** (Cell/gene filtering, count normalization to 10,000 with log1p transformation, and 2000 highly variable gene selection)
- **PyTorch** (Backend for training the scCobra neural encoder under the three-phase optimization protocol)
- **scIB** (Leiden clustering (resolutions 0.1–2.0 in 0.1 increments) and integration metric computation on embeddings)

## Examples

```
git clone https://github.com/mcgilldinglab/scCobra.git && pip install scanpy scib && pip3 install torch torchvision torchaudio torch_optimizer
```

## Evaluation signals

- Reproduced scCobra weighted F1 on the three pancreas query datasets matches the ~0.95 reported in Fig. 6b-e
- Benchmark comparison reproduces the ordering scCobra 0.95 ≈ SCALEX 0.95 > TOSICA 0.94 > scVI 0.92
- Per-cell-type ACC heatmap is generated with all reference cell types annotated
- Preprocessing checks: no cells with <200 expressed genes, no genes seen in <3 cells, no cells with >5% mitochondrial fraction, 2000 HVGs retained
- Query embeddings are produced with the frozen pre-trained encoder and identical gene feature space as the reference atlas (no retraining performed)

## Limitations

- Label transfer requires that the reference and query datasets originate from the same tissue type or contain similar cell types
- Benchmarking methods (Seurat, Harmony, Scanorama, scVI, scDML, scDREAMER) were run with default parameters, which may not reflect their tuned performance
- Over-correction scores are derived from two-dimensional UMAP embeddings, introducing dependence on UMAP
- scATAC-seq multimodal integration relies on converting peak matrices into gene activity matrices, which may lead to information loss

## Evidence

- [abstract] scCobra effectively mitigates batch effects, minimizes over-correction, and ensures biologically meaningful data integration without assuming specific gene expression distributions.: "scCobra effectively mitigates batch effects, minimizes over-correction, and ensures biologically meaningful data integration without assuming specific gene expression distributions."
- [abstract] It enables online label transfer across datasets with batch effects, allowing continuous integration of new data without retraining.: "It enables online label transfer across datasets with batch effects, allowing continuous integration of new data without retraining."
- [discussion] we removed cells where mitochondrial genes accounted for more than 5% of the total gene expression: "we removed cells where mitochondrial genes accounted for more than 5% of the total gene expression"
- [discussion] the pre-trained KNN classifier is then used to annotate the query dataset, ensuring the model generalizes effectively to unseen data: [evidence span withheld — anchor status: not_found]
- [discussion] optimization performed in three distinct phases: "optimization performed in three distinct phases"
- [readme] a deep generative model that combines a contrastive neural network with domain adaptation: "a deep generative model that combines a contrastive neural network with domain adaptation"
