---
name: annadata-expression-matrix-subsetting
description: Use when a query .h5ad dataset has a different gene set or gene count
  than the features a SIMS/TabNet reference model was trained on, and label-transfer
  prediction fails or would fail on feature mismatch.
when_to_use_negative:
- The query dataset's gene panel already matches the model's training features in
  both identity and order — intersection adds no value
- Prediction is intended across species (e.g., mouse to human neuronal subtypes);
  SIMS has only been validated within the same species because neuronal subtypes diverge
  significantly between species
- The input is a gene-by-cell matrix or a summary/feature table rather than a cell
  x gene expression matrix, which SIMS requires
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SIMS
  role: Trains the TabNet-based reference label-transfer model whose feature ordering
    defines the gene intersection target
  repo: https://github.com/braingeneers/SIMS
- name: TabNet
  role: Underlying architecture optimized for tabular data; requires query features
    to align positionally with training features
  repo: https://github.com/braingeneers/SIMS
- name: Scanpy
  role: Loads the query .h5ad AnnData and applies the preprocessing (filtering, normalize_total,
    log1p, scale) applied to training data
  repo: https://github.com/braingeneers/SIMS
- name: PyTorch Lightning
  role: Runs the SIMS prediction on the intersected, reordered query matrix
  repo: https://github.com/braingeneers/SIMS
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.xgen.2024.100581
    title: 'SIMS: A deep-learning label transfer tool for single-cell RNA sequencing
      analysis'
  source_content_hash: sha256:383e742dffc221dca541eb4683a4b88fbb42a2816c0be726c8bad64d6ff54c46
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# AnnData expression matrix subsetting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Aligns a query single-cell AnnData object's gene panel to a trained SIMS/TabNet reference model's feature ordering via set intersection, so label-transfer prediction works across datasets with different numbers of genes (e.g., PBMC68K ~20k genes vs Han landscape ~5k genes).

## When to use

Use when a query .h5ad dataset has a different gene set or gene count than the features a SIMS/TabNet reference model was trained on, and label-transfer prediction fails or would fail on feature mismatch.

## When NOT to use

- The query dataset's gene panel already matches the model's training features in both identity and order — intersection adds no value
- Prediction is intended across species (e.g., mouse to human neuronal subtypes); SIMS has only been validated within the same species because neuronal subtypes diverge significantly between species
- The input is a gene-by-cell matrix or a summary/feature table rather than a cell x gene expression matrix, which SIMS requires

## Inputs

- Query single-cell expression matrix as AnnData (.h5ad) in cell x gene orientation
- Trained SIMS TabNet model checkpoint (e.g., weights_path='myawesomemodel.ckpt') with its training gene/feature order
- Optional: gene_intersection.csv mapping of shared genes

## Outputs

- Intersected, reordered AnnData expression matrix whose columns exactly match the reference model's feature ordering
- gene_intersection.csv listing exactly the genes shared between query panel and training features
- Cell-type label predictions as a pandas DataFrame with pred_0/prob_0 (and optional top-k) columns

## How to apply

Load the query data as an AnnData object and extract the ordered gene-name vector from the reference model's training expression matrices and from adata.var. Compute the set intersection of the two gene panels, then subset and reorder the query expression matrix (cell x gene) so its columns match the reference model's feature ordering exactly — TabNet treats the input as tabular, so feature identity and position must match training. Process the intersected data the same way the training data was processed (normalize_total, log1p, scale, plus filter_cells min_genes=100 / filter_genes min_cells=3 as used for SIMS pretrained models) before calling predict. Validate by running prediction on panels of different sizes and confirming no feature-mismatch errors and that the output gene_intersection.csv contains exactly the genes shared between the query panel and the model's training features.

## Related tools

- **SIMS** (Trains the TabNet-based reference label-transfer model whose feature ordering defines the gene intersection target) — https://github.com/braingeneers/SIMS
- **TabNet** (Underlying architecture optimized for tabular data; requires query features to align positionally with training features) — https://github.com/braingeneers/SIMS
- **Scanpy** (Loads the query .h5ad AnnData and applies the preprocessing (filtering, normalize_total, log1p, scale) applied to training data) — https://github.com/braingeneers/SIMS
- **PyTorch Lightning** (Runs the SIMS prediction on the intersected, reordered query matrix) — https://github.com/braingeneers/SIMS

## Examples

```
adata = adata[:, adata.var_names.intersection(model_genes)].copy(); top1 = sims.predict(adata, top_k=1)
```

## Evaluation signals

- Prediction executes without feature-mismatch errors on both benchmark panels with different feature counts (PBMC68K ~20k genes vs Han landscape ~5k genes)
- gene_intersection.csv contains exactly the genes shared between the query panel and the model's training features — no extra or missing genes
- Column order of the subsetted query matrix matches the reference model's training feature ordering exactly (position-sensitive check)
- Output is a valid per-cell prediction DataFrame with pred_0/prob_0 columns and one row per input cell
- Query data was preprocessed identically to training data (normalize_total, log1p, scale)

## Limitations

- SIMS label transfer has only been validated within the same species; trans-species prediction is untested
- As a supervised model, SIMS inherits errors from upstream annotations (normalization, clustering, differential expression); incorrect normalization or sequencing-technology differences can corrupt the intersected features' interpretability
- Query data must be log1p normalized and preprocessed the same way as the training data or predictions will be invalid

## Evidence

- [readme] Cell-by-gene matrix required for prediction: [evidence span withheld — anchor status: external_doc]
- [readme] Query preprocessing must mirror training preprocessing: [evidence span withheld — anchor status: external_doc]
- [readme] SIMS input accepts arbitrarily many expression matrices: "SIMS takes in a list of arbitrarily many expression matrices along with their corresponding target variables."
- [methods] Reference model training call defining the feature set: "sims = SIMS(adata=adata, labels_key='class_label')"
- [discussion] TabNet handles large tabular feature spaces: "SIMS, being based on TabNet, and therefore optimized for tabular data,30 can work well with over 45,000 features"
