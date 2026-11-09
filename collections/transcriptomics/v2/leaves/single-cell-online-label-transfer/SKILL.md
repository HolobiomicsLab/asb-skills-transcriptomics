---
name: single-cell-online-label-transfer
description: Use when new scRNA-seq query datasets with batch effects must be annotated
  against an existing reference atlas, and retraining a model or re-fitting the classifier
  for each arriving dataset is impractical.
when_to_use_negative:
- Reference and query datasets originate from different tissue types or contain dissimilar
  cell types — the paper states label transfer requires same-tissue or similar-cell-type
  reference/query pairs for optimal performance
- The goal is integration benchmarking of batch-correction methods rather than annotating
  new queries (use the scCobra integration workflow instead)
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scCobra
  role: Contrastive deep generative encoder with domain adaptation; trained on the
    reference atlas in three optimization phases, then applied frozen to query datasets
    for online label transfer
  repo: github.com/mcgilldinglab/scCobra
- name: Scanpy
  role: 'Preprocessing of reference and query datasets: cell/gene filtering, count
    normalization with log1p transformation'
- name: PyTorch
  role: Backend for training and applying the scCobra neural network encoder
- name: scIB
  role: Evaluation pipeline (Leiden clustering across resolutions 0.1–2.0) used alongside
    the label-transfer accuracy assessment
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s42003-025-07692-x
    title: scCobra allows contrastive cell embedding learning with domain adaptation
      for single cell data integration and harmonization
  source_content_hash: sha256:b03ac6cc29869226e395ff20bc7ef87b9feffcb95c6d27bd44e207cc90f397f7
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
yes_rate: 0.7777777777777778
n_claims: 9
---

# single-cell-online-label-transfer

## Summary

Annotate new single-cell query datasets by mapping them onto a pre-trained scCobra reference atlas using a frozen encoder and a pre-trained KNN classifier, without retraining. This online label transfer benchmarks weighted F1 = 0.95 on pancreas queries, matching SCALEX and beating TOSICA (0.94) and scVI (0.92).

## When to use

Use when new scRNA-seq query datasets with batch effects must be annotated against an existing reference atlas, and retraining a model or re-fitting the classifier for each arriving dataset is impractical.

## When NOT to use

- Reference and query datasets originate from different tissue types or contain dissimilar cell types — the paper states label transfer requires same-tissue or similar-cell-type reference/query pairs for optimal performance
- The goal is integration benchmarking of batch-correction methods rather than annotating new queries (use the scCobra integration workflow instead)

## Inputs

- Pancreas reference dataset and query datasets GSE114297, GSE85247, GSE83312 (packaged with Integration_pancreas_label_transfer.ipynb in the scCobra repository)
- Reference cell-type labels for training the KNN classifier
- Query .h5ad/expression count matrices preprocessed with Scanpy (cell/gene filtering, log1p normalization, 2000 HVGs)

## Outputs

- Cell-type label predictions for every query cell
- Weighted F1 scores per query dataset (expected ~0.95 for scCobra)
- Per-cell-type accuracy (ACC) heatmap covering all reference cell types

## How to apply

Preprocess the reference atlas and query datasets (e.g., the three pancreas queries GSE114297, GSE85247, GSE83312 from the Integration_pancreas_label_transfer.ipynb tutorial) with Scanpy: filter cells expressing fewer than 200 genes, filter genes detected in fewer than 3 cells, remove cells with mitochondrial fraction above 5%, normalize with log1p transformation, and select 2000 highly variable genes. Train the scCobra contrastive deep generative encoder on the reference atlas using the three-phase optimization protocol to obtain batch-corrected embeddings, then fit a KNN classifier on those reference embeddings with reference cell-type labels. Apply the frozen encoder to each query dataset to infer latent embeddings without any retraining, and annotate query cells by passing the query embeddings through the pre-trained KNN classifier. Compute weighted F1 scores against benchmark label-transfer methods (SCALEX, TOSICA, scVI) and per-cell-type accuracy to reproduce the Fig. 6b-e comparison.

## Related tools

- **scCobra** (Contrastive deep generative encoder with domain adaptation; trained on the reference atlas in three optimization phases, then applied frozen to query datasets for online label transfer) — github.com/mcgilldinglab/scCobra
- **Scanpy** (Preprocessing of reference and query datasets: cell/gene filtering, count normalization with log1p transformation)
- **PyTorch** (Backend for training and applying the scCobra neural network encoder)
- **scIB** (Evaluation pipeline (Leiden clustering across resolutions 0.1–2.0) used alongside the label-transfer accuracy assessment)

## Examples

```
git clone https://github.com/mcgilldinglab/scCobra.git && jupyter notebook Integration_pancreas_label_transfer.ipynb
```

## Evaluation signals

- Reproduced scCobra weighted F1 on the pancreas query datasets matches the ~0.95 value reported in Fig. 6b-e
- Benchmark comparison reproduces the ordering: scCobra 0.95 ≈ SCALEX 0.95 > TOSICA 0.94 > scVI 0.92
- Per-cell-type ACC heatmap is generated with all reference cell types annotated
- No retraining occurs between query datasets — the same frozen encoder and pre-trained KNN classifier annotate all three queries
- Preprocessing invariants hold: cells with <200 genes, genes in <3 cells, and cells with >5% mitochondrial fraction are removed before normalization

## Limitations

- Label transfer requires the reference and query datasets to originate from the same tissue type or contain similar cell types to ensure optimal performance
- Accuracy depends on the quality of the pre-trained reference atlas; batch effects in queries must be within the domain-adaptation capacity of the encoder
- Benchmark comparisons (SCALEX, TOSICA, scVI) were evaluated under the paper's specific preprocessing and settings; weighted F1 may vary with different atlas compositions

## Evidence

- [abstract] It enables online label transfer across datasets with batch effects, allowing continuous integration of new data without retraining.: "It enables online label transfer across datasets with batch effects, allowing continuous integration of new data without retraining."
- [discussion] the pre-trained KNN classifier is then used to annotate the query dataset, ensuring the model generalizes effectively to unseen data: [evidence span withheld — anchor status: not_found]
- [discussion] the label transfer performed by scCobra requires that the reference and query datasets originate from the same tissue type or contain similar cell types to ensure optimal performance: "the label transfer performed by scCobra requires that the reference and query datasets originate from the same tissue type or contain similar cell types to ensure optimal performance"
- [discussion] we first filtered out cells that expressed fewer than 200 genes: "we first filtered out cells that expressed fewer than 200 genes"
- [discussion] optimization performed in three distinct phases: "optimization performed in three distinct phases"
