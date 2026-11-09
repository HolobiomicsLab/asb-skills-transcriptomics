---
name: knn-classifier-cell-type-annotation
description: Use when a pre-built single-cell reference atlas exists and new query
  datasets (e.g., pancreas GSE114297, GSE85247, GSE83312) must be cell-type annotated
  by mapping onto frozen embeddings without retraining the generative model.
when_to_use_negative:
- Reference and query datasets come from different tissues or contain dissimilar cell
  types — the article states label transfer requires the same tissue type or similar
  cell types for optimal performance.
- The reference atlas does not yet exist and no pre-trained scCobra encoder is available;
  the skill presumes a reference was already trained via the three-phase protocol.
- Benchmarks where retraining per query is acceptable and a fully supervised model
  fine-tuned on the query would outperform a frozen-encoder transfer.
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scCobra
  role: Provides the contrastive deep generative encoder whose frozen embeddings feed
    the KNN classifier for online label transfer
  repo: github.com/mcgilldinglab/scCobra
- name: Scanpy
  role: 'Preprocessing of reference and query datasets: cell/gene filtering and count
    normalization with log1p transformation'
- name: PyTorch
  role: Implementation backend for the scCobra encoder used to embed query datasets
- name: scIB
  role: Benchmarking/evaluation of the integrated embeddings supporting label-transfer
    comparison
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s42003-025-07692-x
    title: scCobra allows contrastive cell embedding learning with domain adaptation
      for single cell data integration and harmonization
  source_content_hash: sha256:1bb5c3e134ea8392a1c3cbf23df86471725e776536a9828532aa05af12647582
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PyTorch
  - Scanpy
  - scCobra
  - scIB
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# knn-classifier-cell-type-annotation

## Summary

Annotates query single-cell datasets by fitting a KNN classifier on batch-corrected reference-atlas embeddings and passing frozen-encoder query embeddings through it — scCobra's online label transfer, which avoids retraining when new data arrive.

## When to use

Use when a pre-built single-cell reference atlas exists and new query datasets (e.g., pancreas GSE114297, GSE85247, GSE83312) must be cell-type annotated by mapping onto frozen embeddings without retraining the generative model.

## When NOT to use

- Reference and query datasets come from different tissues or contain dissimilar cell types — the article states label transfer requires the same tissue type or similar cell types for optimal performance.
- The reference atlas does not yet exist and no pre-trained scCobra encoder is available; the skill presumes a reference was already trained via the three-phase protocol.
- Benchmarks where retraining per query is acceptable and a fully supervised model fine-tuned on the query would outperform a frozen-encoder transfer.

## Inputs

- Pre-trained scCobra encoder and batch-corrected reference-atlas embeddings (e.g., pancreas atlas, 16382 cells, 14 cell types, 9 batches)
- Query scRNA-seq count matrices (e.g., GSE114297, GSE85247, GSE83312), Scanpy-preprocessed (filtered cells <200 genes, genes <3 cells, mito fraction >5% removed, normalized with log1p)
- Reference cell-type labels for fitting the KNN classifier

## Outputs

- Per-cell cell-type annotations for each query dataset
- Weighted F1 scores per method (target ~0.95 for scCobra as in Fig. 6b-e)
- Per-cell-type accuracy (ACC) heatmap

## How to apply

First train the scCobra contrastive deep generative encoder on the reference atlas using the three-phase optimization protocol to obtain batch-corrected cell embeddings, after Scanpy preprocessing (cell/gene filtering, normalization with log1p transformation). Fit a KNN classifier on the reference embeddings using the reference cell-type labels. Apply the frozen, pre-trained scCobra encoder to each query dataset to infer latent embeddings without any model retraining, then pass those query embeddings through the pre-trained KNN classifier to assign cell-type labels. Evaluate by computing weighted F1 scores (scCobra reached 0.95 on pancreas queries, matching SCALEX and beating TOSICA 0.94 and scVI 0.92) and a per-cell-type accuracy heatmap. Decision point: reference and query must share tissue origin or similar cell types, since the label transfer assumes comparable cell-type repertoires.

## Related tools

- **scCobra** (Provides the contrastive deep generative encoder whose frozen embeddings feed the KNN classifier for online label transfer) — github.com/mcgilldinglab/scCobra
- **Scanpy** (Preprocessing of reference and query datasets: cell/gene filtering and count normalization with log1p transformation)
- **PyTorch** (Implementation backend for the scCobra encoder used to embed query datasets)
- **scIB** (Benchmarking/evaluation of the integrated embeddings supporting label-transfer comparison)

## Examples

```
git clone https://github.com/mcgilldinglab/scCobra.git; # then run the Integration_pancreas_label_transfer.ipynb tutorial: fit KNN on reference embeddings, apply frozen encoder to query, predict labels
```

## Evaluation signals

- Reproduced scCobra weighted F1 on the pancreas query datasets matches the ~0.95 value reported in Fig. 6b-e.
- Comparison ordering holds: scCobra 0.95 ≈ SCALEX 0.95 > TOSICA 0.94 > scVI 0.92.
- Per-cell-type ACC heatmap is generated with all reference cell types annotated.
- No encoder retraining occurs when applying the model to the three query datasets — inference uses the frozen pre-trained encoder only.
- Query embeddings land in the same latent space as the reference embeddings (KNN classifier transfers without modification).

## Limitations

- Label transfer requires reference and query to originate from the same tissue type or contain similar cell types to ensure optimal performance.
- Benchmark methods (Seurat, Harmony, Scanorama, scVI, scDML, scDREAMER) were run with default parameters, so comparative F1 values may not reflect tuned performance.
- Existing label-transfer approaches (motivating this skill) necessitate model retraining whenever new data becomes available, limiting reuse — the frozen-encoder design here addresses but does not eliminate distribution-shift risk on unseen tissues.

## Evidence

- [discussion] Online label transfer: reference atlas construction plus pre-trained KNN classifier annotation: [evidence span withheld — anchor status: not_found]
- [discussion] Label transfer requires matched tissue/cell types: "the label transfer performed by scCobra requires that the reference and query datasets originate from the same tissue type or contain similar cell types"
- [abstract] Online label transfer without retraining: "It enables online label transfer across datasets with batch effects, allowing continuous integration of new data without retraining."
- [discussion] Preprocessing pipeline for reference and query data: "we first filtered out cells that expressed fewer than 200 genes"
- [readme] README data resource for the pancreas reference atlas: [evidence span withheld — anchor status: external_doc]
