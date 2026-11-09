---
name: platform-lisi-batch-mixing-quantification
description: Use when you need to verify that a batch-effect removal step (e.g., SELINA's
  MADA network or autoencoder fine-tuning) actually mixes platform batches while preserving
  cell-type separation, e.g., across merged liver reference datasets or reference-query
  pairs.
when_to_use_negative:
- You want to remove batch effects themselves — this skill only quantifies mixing
  after an integration method (SELINA MADA/autoencoder or Harmony) has been run
- The reference contains a single platform and no query data, so no platform-common/platform-specific
  or QR-common/QR-specific cell groups can be defined
- Disease query data where SELINA's fine-tuning step is skipped (disease mode with
  --disease), since the reference-query LISI evaluation depends on fine-tuned reference-query
  integration
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SELINA
  role: Provides the MADA-pre-trained and autoencoder-fine-tuned embeddings whose
    batch mixing is quantified with LISI
  repo: https://github.com/SELINA-team/SELINA-reference
- name: Harmony
  role: Baseline integration method re-run on the same merged data to compute matching
    LISI scores for comparison (Figures 3D-3F)
  repo: https://github.com/immunogenomics/harmony
- name: Python
  role: Language used (with R) to compute LISI scores, write lisi_scores.csv, and
    render UMAP PNGs
- name: R
  role: Language used to run Harmony and compute the matching LISI scores on the integrated
    embeddings
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.crmeth.2023.100577
    title: Single-cell assignment using multiple-adversarial domain adaptation network
      with large-scale references
  source_content_hash: sha256:c465d893ec520ce7d99a7477edbd9a4b31b4f7a8ee60dbc7759cefa3722e4605
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/SELINA-team/SELINA.py
  repo_url: https://github.com/SELINA-team/SELINA-reference
  tools:
  - Harmony
  - Python
  - R
  - SELINA
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# platform-lisi-batch-mixing-quantification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies batch removal and cell-type preservation in integrated single-cell embeddings using LISI (Local Inverse Simpson Index) scores, computed for platform-common vs platform-specific cells in a merged reference and for reference-query common vs source-specific cells after autoencoder fine-tuning, with Harmony as the integration baseline.

## When to use

Use when you need to verify that a batch-effect removal step (e.g., SELINA's MADA network or autoencoder fine-tuning) actually mixes platform batches while preserving cell-type separation, e.g., across merged liver reference datasets or reference-query pairs.

## When NOT to use

- You want to remove batch effects themselves — this skill only quantifies mixing after an integration method (SELINA MADA/autoencoder or Harmony) has been run
- The reference contains a single platform and no query data, so no platform-common/platform-specific or QR-common/QR-specific cell groups can be defined
- Disease query data where SELINA's fine-tuning step is skipped (disease mode with --disease), since the reference-query LISI evaluation depends on fine-tuned reference-query integration

## Inputs

- Merged single-cell reference of gene-by-cell expression matrices (*_expr.txt) with per-cell metadata (*_meta.txt: cell type and sequencing platform)
- Query expression matrix (query_expr.txt) plus SELINA pre-trained model (pre-trained_params.pt)
- SELINA integrated embeddings (feature-extractor and cell-type-classifier transformations)
- Harmony-integrated embedding of the same merged data (comparator)

## Outputs

- lisi_scores.csv with platform-LISI and reference-query LISI columns for SELINA and Harmony
- UMAP PNG files of the integrated embeddings colored by platform/cell-type groups

## How to apply

Merge the multiple reference datasets (gene-by-cell *_expr.txt matrices with cell-type and sequencing-platform metadata, e.g., five liver datasets) into one combined reference and pre-train SELINA's multi-adversarial domain adaptation (MADA) network with 'selina train', which first oversamples rare cell types with SMOTE. Fine-tune the pre-trained parameters to the query distribution with 'selina predict', which runs Fine-tuning1 for 50 epochs and Fine-tuning2 for 10 epochs, then extract the integrated embeddings from both the feature-extractor and cell-type-classifier transformations. Compute LISI scores per cell group: platform-LISI for platform-common vs platform-specific cells (reference integration) and reference-query LISI for QR-common vs QR-specific cell types (fine-tuning integration); correct batch removal should increase LISI for common cells (better mixing) and decrease LISI for specific cells (preserved separation). Re-run the same merged data through Harmony and compute matching LISI scores as a comparator, then write all values to lisi_scores.csv and render embedding UMAPs to PNG files. Judge success by the expected direction of change relative to pre-integration, reproducing the Figure 3C/3G pattern where SELINA shows increased LISI for platform-common/QR-common cells and decreased LISI for platform-specific/QR-specific cells.

## Related tools

- **SELINA** (Provides the MADA-pre-trained and autoencoder-fine-tuned embeddings whose batch mixing is quantified with LISI) — https://github.com/SELINA-team/SELINA-reference
- **Harmony** (Baseline integration method re-run on the same merged data to compute matching LISI scores for comparison (Figures 3D-3F)) — https://github.com/immunogenomics/harmony
- **Python** (Language used (with R) to compute LISI scores, write lisi_scores.csv, and render UMAP PNGs)
- **R** (Language used to run Harmony and compute the matching LISI scores on the integrated embeddings)

## Examples

```
selina train --path-in ./reference_data/ --path-out ./res/pre-train/ --outprefix pre-trained && selina predict --query-expr ./res/query_expr.txt --model ./res/pre-train/pre-trained_params.pt --seurat ./res/query_res.rds --path-out ./res/predict/
```

## Evaluation signals

- lisi_scores.csv exists and contains platform-LISI and reference-query LISI columns for both SELINA and Harmony
- Relative to pre-integration, SELINA's platform-LISI increases for platform-common cells and decreases for platform-specific cells after the MADA feature-extractor and classifier transformations
- After autoencoder fine-tuning, reference-query LISI increases for QR-common cell types and decreases for QR-specific (source-specific) cell types
- Direction-of-change pattern matches the published Figure 3C/3G result and can be compared against Harmony's LISI scores on the same data
- Embedding UMAPs are rendered to PNG files showing the expected mixing of platform-common cells with preserved separation of platform-specific cells

## Limitations

- LISI is evaluated post hoc on embeddings; it does not itself remove batch effects and depends on correct platform-common vs platform-specific and QR-common vs QR-specific cell-group definitions from the metadata
- SELINA's pre-trained reference atlas is still imbalanced across tissues and cell types ('the number of datasets in different tissues and the cell-type abundance in each tissue is still imbalanced'), which can affect the cell groups being compared
- Fine-tuning behavior is only meaningful for normal-mode prediction; in disease mode the fine-tuning step is removed, so reference-query LISI after fine-tuning is not applicable

## Evidence

- [readme] Harmony serves as the comparator integration method: [evidence span withheld — anchor status: external_doc]
