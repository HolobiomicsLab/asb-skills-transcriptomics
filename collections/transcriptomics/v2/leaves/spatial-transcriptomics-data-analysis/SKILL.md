---
name: spatial-transcriptomics-data-analysis
description: Use when paired spatial-transcriptomics and scRNA-seq datasets (e.g.
  MDA-MB-231-GFP xenograft spatial spots plus Chromium 10X scRNA-seq) must be integrated
  to infer spatial patterns or cross-domain gene correlations at single-cell resolution.
when_to_use_negative:
- The two modalities are already simultaneously measured (paired data) — scMMGAN targets
  prediction of missing or non-simultaneous modalities, so a jointly measured dataset
  needs no cross-domain generation
- Only one modality is available and no second domain exists to align to — scMMGAN
  maps between two inferred domains (x1, x2)
- The goal is simple expression quantification on the original spatial data without
  cross-domain inference — direct analysis on the raw spatial matrix is more appropriate
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: scMMGAN
  role: Multi-modal GAN model that maps spatial-transcriptomics features (x1) into
    the scRNA-seq domain (output_fake2) to generate single-cell-resolved profiles
    on spatial coordinates
  repo: https://github.com/KrishnaswamyLab/scMMGAN
- name: TensorFlow
  role: Framework for graph construction, training loop (train_op_G then train_op_D),
    and sess.run execution of the output_fake2 mapping
- name: MAGIC
  role: Manifold-smoothing method used to correct dropout in both spatial and scRNA-seq
    inputs before 50-PC reduction
- name: Chromium 10X
  role: Technology used to generate the scRNA-seq target domain from FACS-sorted GFP+
    tumor cells
provenance:
  source_task_ids:
  - task_005
  - task_006
  source_papers:
  - doi: 10.1016/j.patter.2022.100577
    title: Single-cell multi-modal GAN reveals spatial patterns in single-cell data
      from triple-negative breast cancer
  source_content_hash: sha256:e0101f93f592549c245615df2b90e209381a16011753085819568e691d8917f9
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/KrishnaswamyLab/scMMGAN
  repo_url: https://github.com/KrishnaswamyLab/scMMGAN
  tools:
  - Chromium 10X
  - MAGIC
  - TensorFlow
  - scMMGAN
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# spatial-transcriptomics-data-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Skill for analyzing spatial transcriptomics data by mapping it into the single-cell RNA-seq domain with scMMGAN, enabling single-cell-resolution spatial clustering and cross-domain gene-correlation discovery on triple-negative breast cancer xenograft data.

## When to use

Use when paired spatial-transcriptomics and scRNA-seq datasets (e.g. MDA-MB-231-GFP xenograft spatial spots plus Chromium 10X scRNA-seq) must be integrated to infer spatial patterns or cross-domain gene correlations at single-cell resolution.

## When NOT to use

- The two modalities are already simultaneously measured (paired data) — scMMGAN targets prediction of missing or non-simultaneous modalities, so a jointly measured dataset needs no cross-domain generation
- Only one modality is available and no second domain exists to align to — scMMGAN maps between two inferred domains (x1, x2)
- The goal is simple expression quantification on the original spatial data without cross-domain inference — direct analysis on the raw spatial matrix is more appropriate

## Inputs

- Spatial transcriptomics matrix from TNBC xenograft tissue (spot-by-gene, with spot coordinates)
- scRNA-seq matrix from Chromium 10X (cell-by-gene) of the corresponding MDA-MB-231-GFP or HCC38 xenograft cells
- Preprocessed feature matrices: MAGIC-corrected, zero-centered/unit-scaled, reduced to 50 principal components
- Trained scMMGAN TensorFlow session with output_fake2 tensor available

## Outputs

- Generated single-cell-resolution profiles on spatial coordinates (output_fake2 mapping of x1 into the scRNA-seq domain)
- Spatial cluster map of spectral k-means (k=5) labels projected onto original tissue coordinates (Figure 5B equivalent)
- SLC2A1-high vs CDK11A-high spatially informed differentiation plot (Figure 5C equivalent)
- slc2a1_top700_correlation_comparison.csv: 700 ranked genes per domain with per-gene correlation values and spurious/discovered annotations

## How to apply

Preprocess both domains identically: correct dropout with MAGIC manifold smoothing, zero-center and unit scale each dimension, and reduce to 50 principal components so input columns are mostly in [-10, 10] (per the scMMGAN README). Load the preprocessed matrices as x1 (spatial) and x2 (scRNA-seq), build the TensorFlow graph through the sess.run call, and train with the generator optimized first (train_op_G) then the discriminator (train_op_D), tuning lambda_correspondence if nearby points are not nearby in the aligned space. Collect the generated profiles from the output_fake2 tensor, which maps x1 into the scRNA-seq domain. For spatial pattern analysis, cluster the generated profiles with spectral k-means at k=5 and project the labels back onto the original spatial spot coordinates (Figure 5B), then stratify SLC2A1-high versus CDK11A-high cells (Figure 5C). For gene-correlation discovery, compute Pearson correlations between SLC2A1 and every other gene in both domains, retain the top 700, and use a low-correlation cutoff of 0.1 to flag spurious artifacts (FRAT2, CAMK2A, CANX, LRRC66, ZMIZ1-AS1, MTRNR2L8) versus scMMGAN-discovered associations (AC092115.3, P2RX7, CCDC93, UTP25, BBS10).

## Related tools

- **scMMGAN** (Multi-modal GAN model that maps spatial-transcriptomics features (x1) into the scRNA-seq domain (output_fake2) to generate single-cell-resolved profiles on spatial coordinates) — https://github.com/KrishnaswamyLab/scMMGAN
- **TensorFlow** (Framework for graph construction, training loop (train_op_G then train_op_D), and sess.run execution of the output_fake2 mapping)
- **MAGIC** (Manifold-smoothing method used to correct dropout in both spatial and scRNA-seq inputs before 50-PC reduction)
- **Chromium 10X** (Technology used to generate the scRNA-seq target domain from FACS-sorted GFP+ tumor cells)

## Examples

```
gen = sess.run(output_fake2, feed_dict={x1: spatial_pcs})  # map MAGIC-corrected, 50-PC spatial data into the scRNA-seq domain
```

## Evaluation signals

- The comparison CSV contains 700 ranked genes per domain, with the six spurious genes (FRAT2, CAMK2A, CANX, LRRC66, ZMIZ1-AS1, MTRNR2L8) flagged in the spatial domain and the five discovered genes (AC092115.3, P2RX7, CCDC93, UTP25, BBS10) flagged in the generated domain
- The k=5 spatial cluster map visually preserves the spatial patterns of the original tissue (Figure 5B expert check; no numeric criterion reported)
- SLC2A1-high vs CDK11A-high cells show the spatial differentiation reported in Figure 5C
- Preprocessed input columns are mostly in the interval [-10, 10] as recommended by the scMMGAN README
- Discovered correlations are biologically plausible — e.g. P2RX7 corroborated as a precursor to glucose transporters — and novel findings are submitted to external verification measures

## Limitations

- GANs are difficult to train due to adversarial losses, which can lead to instability, and the model can deteriorate from effective to ineffective quickly across training iterations; mode collapse may occur because distribution-level losses are absent
- Mitigation via early stopping and the geometric loss is only informally observed — instability and mode collapse may still be present in some contexts
- The geometry-based loss is not completely plug-and-play: it requires a choice of distance between data points, and the number of generators grows quadratically with the number of input modalities
- Novel gene correlations found by scMMGAN are potential discoveries that require specifically designed follow-up experiments; aligned results need continued external verification

## Evidence
