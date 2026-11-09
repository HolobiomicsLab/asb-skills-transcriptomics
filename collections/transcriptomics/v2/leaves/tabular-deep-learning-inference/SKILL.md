---
name: tabular-deep-learning-inference
description: Use when a query AnnData (.h5ad) dataset has a different number/order
  of genes than the reference model's training panel, and you need cell-type label
  predictions from a pretrained SIMS TabNet checkpoint without feature-mismatch errors.
when_to_use_negative:
- The query dataset is a different species — SIMS has only been validated for label
  transfer within the same species because 'neuronal subtypes diverge significantly
  between species'
- The model was trained on differently normalized data — preprocessing (normalize_total,
  log1p, scale) must match training exactly or predictions will be invalid
- You need fully unsupervised cell-type discovery with no reference annotations —
  SIMS is a supervised label-transfer tool requiring pre-existing expert annotations
  for training
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3474
tools:
- name: SIMS
  role: Low-code pipeline providing the SIMS class whose predict() method runs TabNet
    label-transfer inference on query AnnData after automated gene-panel alignment
  repo: https://github.com/braingeneers/SIMS
- name: TabNet
  role: Underlying deep-learning architecture optimized for large-scale tabular data
    (>45,000 features) on which the SIMS classifier is built
- name: PyTorch Lightning
  role: Implementation framework in which SIMS is written, handling checkpoint loading
    and trainer setup for inference
- name: Scanpy
  role: Used to read .h5ad AnnData objects and apply the required preprocessing (filter,
    normalize_total, log1p, scale) before prediction
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.xgen.2024.100581
    title: 'SIMS: A deep-learning label transfer tool for single-cell RNA sequencing
      analysis'
  source_content_hash: sha256:4ce4210af82d95c4ecd5c581dac30b85c8e79c5cd474699c4b9e77bbe9f8677c
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
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# tabular-deep-learning-inference

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Run label-transfer inference on single-cell RNA-seq query datasets using a trained SIMS/TabNet classifier, including automatic alignment of the query gene panel to the model's training features via set intersection so cross-technology prediction works even when panels differ in size (e.g., PBMC68K ~20k genes vs Han landscape ~5k genes).

## When to use

Use when a query AnnData (.h5ad) dataset has a different number/order of genes than the reference model's training panel, and you need cell-type label predictions from a pretrained SIMS TabNet checkpoint without feature-mismatch errors.

## When NOT to use

- The query dataset is a different species — SIMS has only been validated for label transfer within the same species because 'neuronal subtypes diverge significantly between species'
- The model was trained on differently normalized data — preprocessing (normalize_total, log1p, scale) must match training exactly or predictions will be invalid
- You need fully unsupervised cell-type discovery with no reference annotations — SIMS is a supervised label-transfer tool requiring pre-existing expert annotations for training

## Inputs

- Query dataset as AnnData (.h5ad) in cell x gene orientation, log1p-normalized and scaled
- Trained SIMS/TabNet model checkpoint (e.g., 'myawesomemodel.ckpt')
- Label file (class_label column) used to originally train the model

## Outputs

- pandas DataFrame of per-cell predictions (pred_0..pred_n labels and prob_0..prob_n softmax probabilities)
- gene_intersection.csv listing exactly the genes shared between query panel and model training features
- Optionally obs columns sims_label and sims_confidence written back to the AnnData object

## How to apply

Load the query data as an AnnData cell-by-gene matrix and preprocess it exactly as the training data (filter_cells min_genes=100, filter_genes min_cells=3, normalize_total, log1p, scale). Load the pretrained SIMS model from its checkpoint (e.g., SIMS(weights_path='model.ckpt'), moving weights to CPU if trained on GPU). Extract the ordered gene-name vectors from the model's training expression matrices and the query var namespace, compute their set intersection, and subset/reorder the query expression matrix to the model's exact feature ordering. Then call sims.predict(unlabeled_data, top_k=3) to get a DataFrame with pred_0..pred_2 labels and prob_0..prob_2 softmax probabilities, or top_k=1 for the single best label with confidence. Validate that prediction runs without feature-mismatch errors and that the generated gene_intersection.csv contains exactly the genes shared between query and training panels.

## Related tools

- **SIMS** (Low-code pipeline providing the SIMS class whose predict() method runs TabNet label-transfer inference on query AnnData after automated gene-panel alignment) — https://github.com/braingeneers/SIMS
- **TabNet** (Underlying deep-learning architecture optimized for large-scale tabular data (>45,000 features) on which the SIMS classifier is built)
- **PyTorch Lightning** (Implementation framework in which SIMS is written, handling checkpoint loading and trainer setup for inference)
- **Scanpy** (Used to read .h5ad AnnData objects and apply the required preprocessing (filter, normalize_total, log1p, scale) before prediction)

## Examples

```
sims = SIMS(weights_path='myawesomemodel.ckpt'); top1 = sims.predict(unlabeled_data, top_k=1); unlabeled_data.obs['sims_label'] = top1['pred_0'].values
```

## Evaluation signals

- Prediction executes without feature-mismatch errors on both benchmark panels (PBMC68K ~20k genes and Han landscape ~5k genes)
- gene_intersection.csv contains exactly the genes shared between the query panel and the model's training features
- Output DataFrame uses canonical pred_0/prob_0 column names with one row per input cell
- Query data underwent the identical preprocessing chain as training (min_genes=100, min_cells=3, normalize_total, log1p, scale)
- prob_0 softmax confidence values fall in (0, 1] for every predicted cell

## Limitations

- SIMS has only been validated for label transfer within the same species; trans-species prediction remains untested
- As a supervised model, SIMS relies on pre-existing annotations (normalization, clustering, differential expression, expert annotation); technical factors such as sequencing technology or incorrect normalization can propagate errors
- For pretrained GPU-trained checkpoints, weights must be explicitly moved to CPU (map_location=torch.device('cpu')) before CPU inference

## Evidence

- [readme] Gene alignment via preprocessing and prediction call: [evidence span withheld — anchor status: external_doc]
- [readme] Predict output schema: [evidence span withheld — anchor status: external_doc]
- [readme] Checkpoint loading: [evidence span withheld — anchor status: external_doc]
