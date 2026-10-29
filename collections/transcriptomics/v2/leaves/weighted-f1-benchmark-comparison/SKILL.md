---
name: weighted-f1-benchmark-comparison
description: Use when benchmarking single-cell label-transfer/integration methods
  on query datasets with weighted F1, e.g. reproducing Fig. 6b-e pancreas comparisons
  (scCobra 0.95, SCALEX 0.95, TOSICA 0.94, scVI 0.92).
when_to_use_negative:
- Reference and query datasets come from different tissue types or contain dissimilar
  cell types — the article states scCobra label transfer requires the same tissue
  type or similar cell types for optimal performance
- The analysis goal is de novo cell-type discovery on unlabeled data rather than transferring
  known reference labels to query cells
edam_operation: http://edamontology.org/operation_3801
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scCobra
  role: Contrastive deep generative encoder trained on the reference atlas with three-phase
    optimization; frozen encoder provides online label-transfer embeddings for query
    datasets
  repo: https://github.com/mcgilldinglab/scCobra
- name: Scanpy
  role: Preprocessing of reference and query pancreas datasets (cell/gene filtering,
    normalization, log1p transformation)
- name: PyTorch
  role: Backend for training and applying the scCobra neural network encoder
- name: scIB
  role: Leiden clustering across resolutions 0.1-2.0 for evaluating integrated embeddings
- name: scVI
  role: Benchmark label-transfer method (scvi-tools v0.11) for weighted F1 comparison
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s42003-025-07692-x
    title: scCobra allows contrastive cell embedding learning with domain adaptation
      for single cell data integration and harmonization
  source_content_hash: sha256:617bab6d31351fed98faa79d5fbebd3d072db8d1ce913138c14148820cf99214
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mcgilldinglab/scCobra
  tools:
  - PyTorch
  - Scanpy
  - scCobra
  - scIB
  - scVI
  edam_operation: http://edamontology.org/operation_3801
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# weighted-f1-benchmark-comparison

## Summary

Reproduce the online label-transfer benchmark of the scCobra pancreas study: annotate three query datasets (GSE114297, GSE85247, GSE83312) through a frozen scCobra encoder plus pre-trained KNN classifier, compute weighted F1 scores, and compare against SCALEX, TOSICA, and scVI (Fig. 6b-e).

## When to use

Use when benchmarking single-cell label-transfer/integration methods on query datasets with weighted F1, e.g. reproducing Fig. 6b-e pancreas comparisons (scCobra 0.95, SCALEX 0.95, TOSICA 0.94, scVI 0.92).

## When NOT to use

- Reference and query datasets come from different tissue types or contain dissimilar cell types — the article states scCobra label transfer requires the same tissue type or similar cell types for optimal performance
- The analysis goal is de novo cell-type discovery on unlabeled data rather than transferring known reference labels to query cells

## Inputs

- Pancreas reference dataset (16382 cells, 14 cell types, 9 batches) as packaged in Integration_pancreas_label_transfer.ipynb
- Pancreas query datasets GSE114297, GSE85247, GSE83312 (count matrices, .h5ad via Scanpy)
- Pre-trained scCobra encoder trained on the reference atlas
- Pre-trained KNN classifier fitted on reference embeddings with reference cell-type labels
- Label-transfer predictions from benchmark methods SCALEX, TOSICA, and scVI (scvi-tools v0.11)

## Outputs

- Weighted F1 scores per method (expected: scCobra 0.95, SCALEX 0.95, TOSICA 0.94, scVI 0.92)
- Fig. 6b-e comparison plot of weighted F1 across methods
- Per-cell-type accuracy (ACC) heatmap with all reference cell types annotated

## How to apply

Load the pancreas reference atlas and the three query datasets packaged with the Integration_pancreas_label_transfer.ipynb tutorial in the scCobra repository, preprocessing with Scanpy (cell/gene filtering, count normalization with log1p). Train the scCobra contrastive deep generative encoder on the reference atlas with the three-phase optimization protocol to obtain batch-corrected embeddings, then fit a KNN classifier on reference embeddings using reference cell-type labels. Apply the frozen encoder to each query dataset to infer latent embeddings without retraining (online label transfer), and pass query embeddings through the pre-trained KNN classifier to annotate cells. Compute weighted F1 for scCobra and the benchmark methods (SCALEX, TOSICA, scVI), reproduce the Fig. 6b-e comparison, and compute per-cell-type accuracy to render the ACC heatmap.

## Related tools

- **scCobra** (Contrastive deep generative encoder trained on the reference atlas with three-phase optimization; frozen encoder provides online label-transfer embeddings for query datasets) — https://github.com/mcgilldinglab/scCobra
- **Scanpy** (Preprocessing of reference and query pancreas datasets (cell/gene filtering, normalization, log1p transformation))
- **PyTorch** (Backend for training and applying the scCobra neural network encoder)
- **scIB** (Leiden clustering across resolutions 0.1-2.0 for evaluating integrated embeddings)
- **scVI** (Benchmark label-transfer method (scvi-tools v0.11) for weighted F1 comparison)

## Examples

```
git clone https://github.com/mcgilldinglab/scCobra.git  # then run Integration_pancreas_label_transfer.ipynb to compute weighted F1 on GSE114297/GSE85247/GSE83312 queries
```

## Evaluation signals

- Reproduced scCobra weighted F1 matches the ~0.95 value reported in Fig. 6b-e
- SCALEX also reaches weighted F1 0.95, TOSICA 0.94, and scVI 0.92, matching the reported ordering
- Per-cell-type ACC heatmap is generated with all reference cell types annotated
- Query datasets are annotated without retraining the scCobra encoder (online label transfer invariant)
- Preprocessing used the article's filter thresholds (cells <200 genes removed, genes in <3 cells removed, >5% mitochondrial fraction removed, 2000 highly variable genes)

## Limitations

- Label transfer requires reference and query datasets to originate from the same tissue type or contain similar cell types
- Benchmarking methods (Seurat, Harmony, Scanorama, scVI, scDML, scDREAMER) were run with default parameters, which may not reflect their tuned performance
- Query datasets cited in the source card are GSE114297, GSE85247, GSE83312; the article's extracted datasets do not list these accessions directly, and no explicit data availability statement with accessioned datasets appears beyond the repo footer

## Evidence

- [abstract] It enables online label transfer across datasets with batch effects, allowing continuous integration of new data without retraining.: "It enables online label transfer across datasets with batch effects, allowing continuous integration of new data without retraining."
- [discussion] the pre-trained KNN classifier is then used to annotate the query dataset, ensuring the model generalizes effectively to unseen data: [evidence span withheld — anchor status: not_found]
- [discussion] we first filtered out cells that expressed fewer than 200 genes: "we first filtered out cells that expressed fewer than 200 genes"
- [discussion] we identified 2000 highly variable genes for downstream analysis: "we identified 2000 highly variable genes for downstream analysis"
- [readme] Please refer run_integration.ipynb: [evidence span withheld — anchor status: external_doc]
