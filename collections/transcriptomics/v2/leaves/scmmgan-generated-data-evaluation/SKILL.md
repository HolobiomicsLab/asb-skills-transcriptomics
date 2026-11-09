---
name: scmmgan-generated-data-evaluation
description: Use when scMMGAN-generated data (e.g. `output_fake2` mapping scRNA-seq
  into the spatial domain) must be biologically validated by comparing gene-gene correlation
  rankings against the original modality to separate spurious artifacts from novel
  associations.
when_to_use_negative:
- When the goal is differential expression or clustering within a single modality
  rather than cross-modality validation of GAN-generated correlations
- When the scMMGAN output has not been preprocessed with the same pipeline (MAGIC,
  zero-centering/unit scaling, 50 PCs) as the original data — rankings will not be
  comparable
- When no biologically meaningful hub gene (like SLC2A1) is available to anchor the
  correlation comparison
tools:
- name: scMMGAN
  role: Generates the cross-modality data (`output_fake2`, scRNA-seq mapped into the
    spatial domain) whose gene correlations are evaluated against the original spatial
    data
  repo: github.com/KrishnaswamyLab/scMMGAN
- name: MAGIC
  role: Corrects dropout in both input domains so correlation rankings are computed
    on comparable, smoothed expression values
- name: TensorFlow
  role: Runs the scMMGAN graph construction and training (`train_op_G` then `train_op_D`)
    that produce the generated output being evaluated
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.patter.2022.100577
    title: Single-cell multi-modal GAN reveals spatial patterns in single-cell data
      from triple-negative breast cancer
  source_content_hash: sha256:a9ccd4b2a92891cdf42dc782af23436171712b275bdc282e76d3d4954fce4d38
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/KrishnaswamyLab/scMMGAN
  tools:
  - MAGIC
  - TensorFlow
  - scMMGAN
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# scmmgan-generated-data-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluates scMMGAN-generated single-cell data (the `output_fake2` mapping of scRNA-seq into the spatial domain) by comparing per-gene Pearson correlation rankings (e.g. SLC2A1 top-700, low-correlation cutoff 0.1) against the original spatial transcriptomics data, to flag spurious artifacts and novel candidate gene associations.

## When to use

Use when scMMGAN-generated data (e.g. `output_fake2` mapping scRNA-seq into the spatial domain) must be biologically validated by comparing gene-gene correlation rankings against the original modality to separate spurious artifacts from novel associations.

## When NOT to use

- When the goal is differential expression or clustering within a single modality rather than cross-modality validation of GAN-generated correlations
- When the scMMGAN output has not been preprocessed with the same pipeline (MAGIC, zero-centering/unit scaling, 50 PCs) as the original data — rankings will not be comparable
- When no biologically meaningful hub gene (like SLC2A1) is available to anchor the correlation comparison

## Inputs

- Preprocessed original spatial transcriptomics gene-expression matrix (MAGIC-corrected, zero-centered/unit-scaled, 50 PCs)
- scMMGAN-generated spatial-domain output (`output_fake2` mapping of the scRNA-seq domain into the spatial domain)
- Hub gene identifier for correlation analysis (e.g. SLC2A1)

## Outputs

- slc2a1_top700_correlation_comparison.csv with per-gene correlation values in both domains and spurious/discovered membership annotations
- Ranked top-700 gene correlation list per domain (original spatial vs scMMGAN-generated)

## How to apply

Preprocess both domains identically — MAGIC dropout correction, zero-centering/unit scaling, and reduction to 50 principal components — so the original spatial transcriptomics data and the scMMGAN output are comparable. Compute Pearson correlations between a hub gene (SLC2A1) and every other gene in the original spatial data, rank genes by correlation coefficient, and retain the top 700; repeat the identical ranking on the scMMGAN-generated data. Compare the two ranked lists with a low-correlation cutoff of 0.1 to annotate membership of spurious genes present only in the original spatial correlations (FRAT2, CAMK2A, CANX, LRRC66, ZMIZ1-AS1, MTRNR2L8) versus discovered genes surfaced by the scMMGAN alignment (AC092115.3, P2RX7, CCDC93, UTP25, BBS10). Write per-gene correlation values in both domains plus spurious/discovered annotations to a CSV (e.g. slc2a1_top700_correlation_comparison.csv). Validate the comparison by cross-checking discovered candidates against external biology (e.g. P2RX7 as a precursor to glucose transporters), since novel scMMGAN correlations remain potential discoveries needing specifically designed experiments.

## Related tools

- **scMMGAN** (Generates the cross-modality data (`output_fake2`, scRNA-seq mapped into the spatial domain) whose gene correlations are evaluated against the original spatial data) — github.com/KrishnaswamyLab/scMMGAN
- **MAGIC** (Corrects dropout in both input domains so correlation rankings are computed on comparable, smoothed expression values)
- **TensorFlow** (Runs the scMMGAN graph construction and training (`train_op_G` then `train_op_D`) that produce the generated output being evaluated)

## Examples

```
import pandas as pd; corr = df.corr(method='pearson')['SLC2A1'].sort_values(ascending=False).head(700)
```

## Evaluation signals

- Output CSV exists and contains 700 ranked genes per domain (original spatial and scMMGAN-generated)
- The six spurious genes (FRAT2, CAMK2A, CANX, LRRC66, ZMIZ1-AS1, MTRNR2L8) are correctly flagged in the spurious annotation column and appear only in original spatial correlations
- The five discovered genes (AC092115.3, P2RX7, CCDC93, UTP25, BBS10) are correctly flagged in the discovered annotation column and pass the low-correlation cutoff of 0.1
- Per-gene correlation values are present for both domains for every ranked gene
- Discovered candidates are corroborated by external biology (e.g. P2RX7 as a precursor to glucose transporters)

## Limitations

- Novel gene correlations found by scMMGAN are potential discoveries that should be further investigated with experiments specifically designed for this aim; external verification measures are required
- GAN training instability and mode collapse may still be present in some contexts despite early stopping and the geometric loss, potentially affecting the generated data being evaluated
- Preprocessing must keep each input column mostly in the interval [-10, 10] for best scMMGAN results; inadequate preprocessing can degrade the alignment and thus the correlation comparison

## Evidence
