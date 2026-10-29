---
name: feature-space-alignment-single-cell
description: Use when running SIMS label-transfer prediction on a query .h5ad dataset
  whose gene set differs in size or ordering from the genes the reference TabNet model
  was trained on.
when_to_use_negative:
- The query dataset already contains exactly the same genes in the same order as the
  model's training features — no intersection/reordering is needed
- 'The goal is cross-species label transfer: the article notes ''we have limited our
  work to samples within the same species'', so alignment across species is untested'
- The model needs metadata such as sequencing technology or cell line as features
  — the article states this is not yet integrated into the SIMS model
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: SIMS
  role: Pipeline providing the trained TabNet reference model whose feature space
    the query data must be aligned to, and the predict() entry point that consumes
    the intersected matrix
  repo: https://github.com/braingeneers/SIMS
- name: TabNet
  role: Underlying tabular-data architecture whose fixed input feature ordering necessitates
    the gene intersection/reordering step
  repo: https://github.com/braingeneers/SIMS
- name: PyTorch Lightning
  role: Framework in which SIMS is implemented, providing the trainer/checkpoint used
    to load the reference model
  repo: https://github.com/braingeneers/SIMS
- name: Scanpy
  role: Used to load and preprocess (filter, normalize_total, log1p, scale) the query
    AnnData prior to and after intersection with the model's gene panel
  repo: https://github.com/braingeneers/SIMS
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.xgen.2024.100581
    title: 'SIMS: A deep-learning label transfer tool for single-cell RNA sequencing
      analysis'
  source_content_hash: sha256:c89ac63c92cc10feafa5182853f779bf4445eb21983985315251f6b0afd182dc
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/jlehrer1/sims
  repo_url: https://github.com/braingeneers/SIMS
  tools:
  - PyTorch Lightning
  - SIMS
  - Scanpy
  - TabNet
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# feature-space-alignment-single-cell

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Aligns the gene panel of a query single-cell RNA-seq AnnData object to the feature space of a trained SIMS/TabNet label-transfer model by computing the set intersection of gene-name vectors and reordering the query expression matrix, enabling cross-dataset prediction (e.g., PBMC68K ~20k genes vs Han landscape ~5k genes) without feature-mismatch errors.

## When to use

Use when running SIMS label-transfer prediction on a query .h5ad dataset whose gene set differs in size or ordering from the genes the reference TabNet model was trained on.

## When NOT to use

- The query dataset already contains exactly the same genes in the same order as the model's training features — no intersection/reordering is needed
- The goal is cross-species label transfer: the article notes 'we have limited our work to samples within the same species', so alignment across species is untested
- The model needs metadata such as sequencing technology or cell line as features — the article states this is not yet integrated into the SIMS model

## Inputs

- Trained SIMS TabNet model checkpoint (e.g., weights_path='myawesomemodel.ckpt') with its ordered training gene panel
- Query single-cell dataset as AnnData (.h5ad) or CSV, in cell x gene orientation, log1p-normalized
- Query gene-name vector from the AnnData var namespace

## Outputs

- Intersected, reordered query expression matrix whose columns match the reference model's feature ordering
- gene_intersection.csv listing exactly the genes shared between the query panel and the model's training features
- Cell-type label predictions (pandas DataFrame with pred_0/prob_0 columns when top_k=1)

## How to apply

Load the query dataset as an AnnData object alongside the trained SIMS TabNet reference model (initialized with SIMS(adata=adata, labels_key='class_label'), setup_model(n_a=64, n_d=64), trained for 100 epochs). Extract the ordered gene-name vectors from the reference model's training expression matrices and from the query AnnData var namespace. Compute the set intersection of the two gene panels, then subset and reorder the query expression matrix so its columns match the reference model's feature ordering exactly. Process the intersected data the same way as the training data (filter_cells min_genes=100, filter_genes min_cells=3, normalize_total, log1p, scale) before calling predict. The rationale is that SIMS/TabNet consumes a cell-by-gene matrix, so cross-technology prediction on differently sized panels only proceeds via direct comparison in the shared (intersected) feature space.

## Related tools

- **SIMS** (Pipeline providing the trained TabNet reference model whose feature space the query data must be aligned to, and the predict() entry point that consumes the intersected matrix) — https://github.com/braingeneers/SIMS
- **TabNet** (Underlying tabular-data architecture whose fixed input feature ordering necessitates the gene intersection/reordering step) — https://github.com/braingeneers/SIMS
- **PyTorch Lightning** (Framework in which SIMS is implemented, providing the trainer/checkpoint used to load the reference model) — https://github.com/braingeneers/SIMS
- **Scanpy** (Used to load and preprocess (filter, normalize_total, log1p, scale) the query AnnData prior to and after intersection with the model's gene panel) — https://github.com/braingeneers/SIMS

## Examples

```
sims = SIMS(weights_path='myawesomemodel.ckpt'); top1 = sims.predict(unlabeled_data, top_k=1); unlabeled_data.obs['sims_label'] = top1['pred_0'].values
```

## Evaluation signals

- Prediction executes without feature-mismatch errors on both benchmark panels with different feature counts (PBMC68K ~20k genes vs Han landscape ~5k genes)
- gene_intersection.csv contains exactly the genes shared between the query panel and the model's training features — no extra or missing genes
- The reordered query matrix has column order identical to the reference model's training feature ordering
- Query data was processed identically to training data (min_genes=100, min_cells=3, normalize_total, log1p, scale) before prediction
- predict() returns a DataFrame with the canonical pred_0/prob_0 columns, one row per input cell

## Limitations

- SIMS label transfer has only been validated within the same species; trans-species prediction remains untested
- Technical factors such as sequencing technology or incorrect normalization may affect downstream differential expression and thus the reference model's labels
- Metadata such as sequencing technology and cell line is not yet integrated into the SIMS model, so alignment is gene-level only

## Evidence

- [readme] Process query data identically before inference: [evidence span withheld — anchor status: external_doc]
