---
name: gene-panel-intersection-across-datasets
description: Use when a query scRNA-seq AnnData has a different gene count or ordering
  than the SIMS TabNet reference model's training features, and cell-type label prediction
  must proceed without feature-mismatch errors.
when_to_use_negative:
- The query dataset already contains exactly the reference model's training genes
  in the same order — no intersection is needed, predict directly
- Cross-species label transfer is the goal — the article restricts SIMS to samples
  within the same species because neuronal subtypes diverge significantly between
  species
- Gene symbols use inconsistent naming conventions across labs — the paper notes lack
  of naming conventions makes harmonization manually intensive and ontology-based
  harmonization is only a proposed future direction
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3474
tools:
- name: SIMS
  role: Low-code pipeline whose predict() consumes the intersected, reordered query
    matrix to transfer cell-type labels from a trained checkpoint
  repo: https://github.com/braingeneers/SIMS
- name: TabNet
  role: Underlying model architecture SIMS is based on; feature-order-sensitive tabular
    model requiring exact gene-panel alignment, works well with over 45,000 features
  repo: https://github.com/braingeneers/SIMS
- name: PyTorch Lightning
  role: Implementation framework for SIMS training and inference (pl.Trainer, EarlyStopping,
    LearningRateMonitor callbacks)
- name: Scanpy
  role: Reads the query .h5ad AnnData and applies the preprocessing steps (filter_cells
    min_genes=100, filter_genes min_cells=3, normalize_total, log1p, scale) applied
    before intersection and prediction
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.xgen.2024.100581
    title: 'SIMS: A deep-learning label transfer tool for single-cell RNA sequencing
      analysis'
  source_content_hash: sha256:3c9b4445d0fcd02bdcd96ded5ba93dc7c46f9d1e642d2eb00fa15bee17cb4c54
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
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# gene-panel-intersection-across-datasets

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Automated gene-set alignment for cross-dataset label transfer with SIMS/TabNet: the query AnnData's gene panel is intersected with and reordered to match the trained reference model's feature panel, so prediction can run on datasets with a different number of genes (e.g. PBMC68K ~20k vs Han landscape ~5k).

## When to use

Use when a query scRNA-seq AnnData has a different gene count or ordering than the SIMS TabNet reference model's training features, and cell-type label prediction must proceed without feature-mismatch errors.

## When NOT to use

- The query dataset already contains exactly the reference model's training genes in the same order — no intersection is needed, predict directly
- Cross-species label transfer is the goal — the article restricts SIMS to samples within the same species because neuronal subtypes diverge significantly between species
- Gene symbols use inconsistent naming conventions across labs — the paper notes lack of naming conventions makes harmonization manually intensive and ontology-based harmonization is only a proposed future direction

## Inputs

- Query single-cell dataset as AnnData (.h5ad) in cell x gene format, preprocessed identically to training (normalize_total, log1p, scale)
- Trained SIMS TabNet model checkpoint (e.g. myawesomemodel.ckpt) with its training gene panel / label file
- Ordered gene-name vectors from the reference model's training expression matrices

## Outputs

- Gene-intersected, reordered query expression matrix matching the reference model's feature ordering
- gene_intersection.csv containing exactly the genes shared between the query panel and the model's training features
- Cell-type label predictions (pandas DataFrame with pred_0/prob_0 columns for top_k=1)

## How to apply

Load the query dataset as an AnnData object (cell x gene, log1p-normalized the same way as training data: filter_cells min_genes=100, filter_genes min_cells=3, normalize_total, log1p, scale) and load the trained SIMS checkpoint (e.g. trained via SIMS(adata=adata, labels_key='class_label'), setup_model(n_a=64, n_d=64), 100 epochs). Extract the ordered gene-name vectors from the reference model's training expression matrices and from the query AnnData var namespace, then compute the set intersection of the two panels. Subset and reorder the query expression matrix so its feature ordering exactly matches the reference model's, since TabNet is feature-order sensitive for tabular input. Run sims.predict on the intersected, reordered matrix to obtain cell-type labels. Validate by repeating the intersection-prediction cycle across benchmark panels of different sizes (PBMC68K ~20k genes vs Han landscape ~5k genes) and checking that prediction executes without feature-mismatch errors.

## Related tools

- **SIMS** (Low-code pipeline whose predict() consumes the intersected, reordered query matrix to transfer cell-type labels from a trained checkpoint) — https://github.com/braingeneers/SIMS
- **TabNet** (Underlying model architecture SIMS is based on; feature-order-sensitive tabular model requiring exact gene-panel alignment, works well with over 45,000 features) — https://github.com/braingeneers/SIMS
- **PyTorch Lightning** (Implementation framework for SIMS training and inference (pl.Trainer, EarlyStopping, LearningRateMonitor callbacks))
- **Scanpy** (Reads the query .h5ad AnnData and applies the preprocessing steps (filter_cells min_genes=100, filter_genes min_cells=3, normalize_total, log1p, scale) applied before intersection and prediction)

## Examples

```
sims = SIMS(weights_path='myawesomemodel.ckpt'); top1 = sims.predict(unlabeled_data, top_k=1)
```

## Evaluation signals

- Prediction executes without feature-mismatch errors on both benchmark panels (PBMC68K ~20k genes vs Han landscape ~5k genes)
- gene_intersection.csv contains exactly the genes shared between the query panel and the model's training features — no extra or missing genes
- Column order of the query expression matrix equals the reference model's training feature ordering after reordering (vector equality check on gene names)
- predict() returns a pandas DataFrame with one row per input cell and canonical pred_0/prob_0 columns (not the deprecated first_pred/first_prob names)
- Query data was preprocessed identically to training (filter_cells min_genes=100, filter_genes min_cells=3, normalize_total, log1p, scale) before intersection

## Limitations

- SIMS label transfer has only been validated within the same species; trans-species prediction remains untested
- Technical factors such as sequencing technology or incorrect normalization may affect downstream differential expression and lead to misnamed clusters in the reference annotations
- TabNet/SIMS accuracy and feature handling are bounded by the tabular architecture; scBERT-style transformer approaches handle up to 16,000 genes differently, and panel size differences beyond the benchmarked ~5k–20k range are unvalidated

## Evidence

- [methods] Model initialization used for the reference checkpoint: [evidence span withheld — anchor status: not_found]
- [readme] Inference on unlabeled query data with a checkpoint: [evidence span withheld — anchor status: external_doc]
- [readme] Query preprocessing must match training preprocessing: [evidence span withheld — anchor status: external_doc]
