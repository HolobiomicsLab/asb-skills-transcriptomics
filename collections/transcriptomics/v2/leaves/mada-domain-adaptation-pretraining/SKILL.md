---
name: mada-domain-adaptation-pretraining
description: Use when merging multiple scRNA-seq reference datasets sequenced on different
  platforms and batch effects must be removed before training a cell-type annotation
  model or fine-tuning on query data.
when_to_use_negative:
- Query data is from a disease context — the fine-tuning step is removed in disease
  mode ('--disease') since disease expression profiles deviate substantially from
  normal references
- Reference consists of a single dataset with no batch/platform structure to remove
  — MADA domain adaptation needs multiple domains
- Input is raw unprocessed counts without normalization and gene-symbol conversion
  to hg38 via 'selina preprocess'
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: SELINA
  role: Provides the selina train CLI that pre-trains the MADA network with SMOTE
    cell-type balancing on merged reference data
  repo: https://github.com/SELINA-team/SELINA.py
- name: Harmony
  role: Batch-integration benchmark run on the same merged data to compare LISI scores
    against SELINA's MADA integration (Figures 3D-3F)
  repo: immunogenomics/harmony
- name: Python
  role: Compute LISI scores and write lisi_scores.csv and UMAP PNGs from the integrated
    embeddings
- name: R
  role: Run Harmony comparison and supporting analysis of the merged reference
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.crmeth.2023.100577
    title: Single-cell assignment using multiple-adversarial domain adaptation network
      with large-scale references
  source_content_hash: sha256:79c21d8af275224220929c73939d852859288d9381de2b7d772460488221346e
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/SELINA-team/SELINA.py
  repo_url: https://github.com/SELINA-team/SELINA.py
  tools:
  - Harmony
  - Python
  - R
  - SELINA
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 14
---

# mada-domain-adaptation-pretraining

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Pre-train SELINA's multi-adversarial domain adaptation (MADA) network on a merged multi-dataset reference (e.g., five liver datasets) so the feature extractor and cell-type discriminator remove platform batch effects, evaluated with LISI scores before label transfer.

## When to use

Use when merging multiple scRNA-seq reference datasets sequenced on different platforms and batch effects must be removed before training a cell-type annotation model or fine-tuning on query data.

## When NOT to use

- Query data is from a disease context — the fine-tuning step is removed in disease mode ('--disease') since disease expression profiles deviate substantially from normal references
- Reference consists of a single dataset with no batch/platform structure to remove — MADA domain adaptation needs multiple domains
- Input is raw unprocessed counts without normalization and gene-symbol conversion to hg38 via 'selina preprocess'

## Inputs

- Merged reference directory with paired *_expr.txt (gene-by-cell expression matrix) and *_meta.txt (cell type and sequencing platform) files
- Multiple single-dataset liver (or other tissue) scRNA-seq references from different sequencing platforms
- Query expression matrix (query_expr.txt) for downstream fine-tuning

## Outputs

- pre-trained_params.pt (trained MADA model parameters)
- pre-trained_meta.pkl (reference cell types and genes)
- Integrated embeddings for LISI computation
- lisi_scores.csv with platform-LISI and reference-query-LISI columns for SELINA and Harmony
- UMAP plots of embeddings written to PNG files

## How to apply

Merge the paired *_expr.txt (gene-by-cell expression) and *_meta.txt (cell type, sequencing platform) files into a single combined reference directory. First balance cell types by oversampling rare cell types with SMOTE, then run 'selina train --path-in ./reference_data/ --path-out ./res/pre-train/' to pre-train the MADA network, which adversarially removes dataset/platform batch effects. To verify batch removal, extract the integrated embeddings and compute platform-LISI scores for platform-common and platform-specific cells: expect increased LISI for common cells and decreased LISI for specific cells, mirroring the Figure 3C pattern. Benchmark against Harmony on the same merged data (Figures 3D-3F). The output pre-trained_params.pt and pre-trained_meta.pkl then feed into 'selina predict', where Fine-tuning1 (50 epochs) and Fine-tuning2 (10 epochs) adapt parameters to the query distribution, again verified by reference-query LISI for QR-common and QR-specific cell types.

## Related tools

- **SELINA** (Provides the selina train CLI that pre-trains the MADA network with SMOTE cell-type balancing on merged reference data) — https://github.com/SELINA-team/SELINA.py
- **Harmony** (Batch-integration benchmark run on the same merged data to compare LISI scores against SELINA's MADA integration (Figures 3D-3F)) — immunogenomics/harmony
- **Python** (Compute LISI scores and write lisi_scores.csv and UMAP PNGs from the integrated embeddings)
- **R** (Run Harmony comparison and supporting analysis of the merged reference)

## Examples

```
selina train --path-in ./reference_data/ --path-out ./res/pre-train/ --outprefix pre-trained
```

## Evaluation signals

- lisi_scores.csv exists and contains platform-LISI and reference-query-LISI columns for both SELINA and Harmony
- Platform-LISI increases for platform-common cells and decreases for platform-specific cells after both the feature extractor and cell-type classifier transformations (Figure 3C pattern)
- Fine-tuning analogously increases LISI for reference-query (QR) common cell types and decreases it for QR-specific cell types (Figure 3G pattern)
- selina train produces both pre-trained_params.pt and pre-trained_meta.pkl in the output directory
- SELINA's LISI profile on the merged liver data matches or improves on the Harmony benchmark computed on the same merged data

## Limitations

- Disease query data requires skipping fine-tuning (--disease flag) because 'the expression profiles of disease data may deviate a lot from normal data', so the reference-query batch-removal evaluation applies to normal-mode prediction
- The reference atlas is still imbalanced: 'the number of datasets in different tissues and the cell-type abundance in each tissue is still imbalanced'
- Pre-trained disease models cover only non-small-cell lung carcinoma, type 2 diabetes, and Alzheimer's disease; other conditions require training with user's own data

## Evidence

- [readme] Disease mode skips fine-tuning: [evidence span withheld — anchor status: external_doc]
