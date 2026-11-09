---
name: pancreas-reference-atlas-construction
description: Use when you must build a reusable reference atlas from a multi-batch
  pancreas (or similar tissue) scRNA-seq dataset so that new query datasets can be
  mapped and annotated without retraining, as done for GSE114297, GSE85247 and GSE83312
  with scCobra.
when_to_use_negative:
- The query dataset originates from a different tissue type or lacks cell types similar
  to the reference — scCobra label transfer requires reference and query to share
  tissue/similar cell types.
- The reference atlas has no cell-type labels, since the KNN classifier must be fit
  on reference labels.
- The goal is benchmarking against methods run with default parameters (Seurat, Harmony,
  Scanorama, scVI, scDML, scDREAMER) rather than building a deployable reference atlas.
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scCobra
  role: Contrastive deep generative encoder trained in three phases on the reference
    atlas to produce batch-corrected embeddings for the frozen KNN classifier
  repo: github.com/mcgilldinglab/scCobra
- name: Scanpy
  role: Cell/gene filtering, count normalization to 10,000, and log1p transformation
    of reference and query datasets
- name: PyTorch
  role: Backend for training the scCobra contrastive deep generative encoder
- name: scIB
  role: Leiden clustering across resolutions 0.1–2.0 in increments of 0.1 for integration
    evaluation
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s42003-025-07692-x
    title: scCobra allows contrastive cell embedding learning with domain adaptation
      for single cell data integration and harmonization
  source_content_hash: sha256:02f131de130e93d00b984eee11d63e03d302dab4b5d446888aed6fc645647bb7
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
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# pancreas-reference-atlas-construction

## Summary

Constructs a scCobra reference atlas from a multi-batch pancreas scRNA-seq dataset (16,382 cells, 14 cell types, 9 batches) by Scanpy preprocessing followed by three-phase training of the scCobra contrastive deep generative encoder, then fits a pre-trained KNN classifier on the reference embeddings for online label transfer.

## When to use

Use when you must build a reusable reference atlas from a multi-batch pancreas (or similar tissue) scRNA-seq dataset so that new query datasets can be mapped and annotated without retraining, as done for GSE114297, GSE85247 and GSE83312 with scCobra.

## When NOT to use

- The query dataset originates from a different tissue type or lacks cell types similar to the reference — scCobra label transfer requires reference and query to share tissue/similar cell types.
- The reference atlas has no cell-type labels, since the KNN classifier must be fit on reference labels.
- The goal is benchmarking against methods run with default parameters (Seurat, Harmony, Scanorama, scVI, scDML, scDREAMER) rather than building a deployable reference atlas.

## Inputs

- Pancreas reference scRNA-seq dataset from scCobra repo (figshare download, 16382 cells, 14 cell types, 9 batches)
- Query pancreas datasets: GSE114297, GSE85247, GSE83312
- Reference cell-type label annotations

## Outputs

- Batch-corrected reference cell embeddings from the frozen scCobra encoder
- Pre-trained KNN classifier mapping reference embeddings to cell-type labels
- Query dataset latent embeddings and per-cell predicted labels
- Weighted F1 scores and per-cell-type accuracy (ACC) heatmap

## How to apply

Load the pancreas reference dataset as packaged in the scCobra repository (Integration_pancreas_label_transfer.ipynb) and preprocess with Scanpy: filter cells expressing fewer than 200 genes, genes detected in fewer than 3 cells, and cells with >5% mitochondrial gene fraction; normalize counts per cell to 10,000 and apply a log1p transformation to stabilize variance. Train the scCobra contrastive deep generative encoder on the reference atlas using the three-phase optimization protocol to obtain batch-corrected cell embeddings. Fit a KNN classifier on the reference embeddings using the reference cell-type labels; this frozen encoder-plus-KNN pair is the deliverable atlas. Rationale: scCobra's contrastive learning with domain adaptation mitigates batch effects without gene-expression distribution assumptions, and freezing the encoder/KNN enables continuous integration of new data without retraining, which standard label-transfer methods lack.

## Related tools

- **scCobra** (Contrastive deep generative encoder trained in three phases on the reference atlas to produce batch-corrected embeddings for the frozen KNN classifier) — github.com/mcgilldinglab/scCobra
- **Scanpy** (Cell/gene filtering, count normalization to 10,000, and log1p transformation of reference and query datasets)
- **PyTorch** (Backend for training the scCobra contrastive deep generative encoder)
- **scIB** (Leiden clustering across resolutions 0.1–2.0 in increments of 0.1 for integration evaluation)

## Examples

```
sc.pp.normalize_total(adata, target_sum=1e4); sc.pp.log1p(adata)
```

## Evaluation signals

- Reference atlas contains the expected scale: 16382 cells, 14 cell types, 9 batches (pancreas dataset).
- scIB Leiden clustering across resolutions 0.1–2.0 yields batch-corrected embeddings suitable for classification.
- Applying the frozen encoder to the three pancreas query datasets without retraining reproduces weighted F1 ≈ 0.95 (Fig. 6b-e), matching SCALEX and exceeding TOSICA (0.94) and scVI (0.92).
- The per-cell-type ACC heatmap is generated with all reference cell types annotated.
- Preprocessing thresholds applied exactly: <200 genes/cell, <3 cells/gene, >5% mitochondrial fraction removed, counts normalized to 10,000 with log1p.

## Limitations

- Label transfer requires the reference and query datasets to originate from the same tissue type or contain similar cell types; cross-tissue transfer is unsupported.
- Over-correction scores are derived from two-dimensional UMAP embeddings, introducing dependence on UMAP parameters.
- scCobra's shared encoder requires selecting shared features between modalities, which is difficult for CITE-seq protein data with only a few hundred features.

## Evidence

- [discussion] filter thresholds: "we first filtered out cells that expressed fewer than 200 genes and genes that were detected in fewer than 3 cells"
- [discussion] mitochondrial filter and log1p: "we removed cells where mitochondrial genes accounted for more than 5% of the total gene expression"
- [discussion] normalization: "This was followed by a log1p transformation to stabilize the variance"
- [discussion] three-phase training: "optimization performed in three distinct phases"
- [discussion] KNN annotation: [evidence span withheld — anchor status: not_found]
- [abstract] online transfer rationale: "It enables online label transfer across datasets with batch effects, allowing continuous integration of new data without retraining."
- [readme] dataset scale: [evidence span withheld — anchor status: external_doc]
- [discussion] tissue constraint: "the label transfer performed by scCobra requires that the reference and query datasets originate from the same tissue type or contain similar cell types"
