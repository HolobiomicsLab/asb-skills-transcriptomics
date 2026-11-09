---
name: gene-expression-correlation-analysis
description: Use when comparing gene-gene correlation landscapes between an original
  measurement domain and a cross-modality mapping (e.g., scMMGAN spatial output),
  to flag spurious correlations and surface novel candidate associations like SLC2A1–P2RX7.
when_to_use_negative:
- Input data has not been through the required preprocessing chain (MAGIC dropout
  correction, zero-centering/unit scaling, 50-PC reduction) — raw count matrices will
  yield incomparable correlation rankings across domains
- The two domains were not aligned by a cross-modality mapping (e.g., scMMGAN), since
  the spurious-vs-discovered comparison assumes a paired original/mapped pair of domains
- The goal is differential expression between conditions rather than correlation-landscape
  comparison against a mapped domain
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scMMGAN
  role: Generates the cross-domain output (output_fake2) whose SLC2A1 correlation
    landscape is compared against the original spatial data to distinguish spurious
    from discovered correlations
  repo: github.com/KrishnaswamyLab/scMMGAN
- name: MAGIC
  role: Corrects for dropout in both input domains before correlation ranking, per
    the article's preprocessing chain
- name: TensorFlow
  role: Framework used for the scMMGAN graph construction and training that produces
    the generated data being correlated
  repo: github.com/KrishnaswamyLab/scMMGAN
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.patter.2022.100577
    title: Single-cell multi-modal GAN reveals spatial patterns in single-cell data
      from triple-negative breast cancer
  source_content_hash: sha256:341a3d8abbd85e3ce6995e927059bf7bf8fd072b33120d2fdda7aaeeb1dc4a6f
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
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# gene-expression-correlation-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Rank all genes by Pearson correlation to a gene of interest (e.g., SLC2A1) in paired domains — original spatial transcriptomics data versus scMMGAN-generated spatial-domain scRNA-seq (output_fake2) — to separate spurious spatial-measurement correlations from novel cross-domain associations.

## When to use

Use when comparing gene-gene correlation landscapes between an original measurement domain and a cross-modality mapping (e.g., scMMGAN spatial output), to flag spurious correlations and surface novel candidate associations like SLC2A1–P2RX7.

## When NOT to use

- Input data has not been through the required preprocessing chain (MAGIC dropout correction, zero-centering/unit scaling, 50-PC reduction) — raw count matrices will yield incomparable correlation rankings across domains
- The two domains were not aligned by a cross-modality mapping (e.g., scMMGAN), since the spurious-vs-discovered comparison assumes a paired original/mapped pair of domains
- The goal is differential expression between conditions rather than correlation-landscape comparison against a mapped domain

## Inputs

- Preprocessed original spatial transcriptomics dataset (MAGIC-corrected, zero-centered/unit-scaled, 50 PCs)
- scMMGAN-generated spatial-domain scRNA-seq data (output_fake2 from the KrishnaswamyLab/scMMGAN pipeline)
- Seed gene identifier (SLC2A1); reference lists of candidate spurious and discovered genes (FRAT2, CAMK2A, CANX, LRRC66, ZMIZ1-AS1, MTRNR2L8; AC092115.3, P2RX7, CCDC93, UTP25, BBS10)

## Outputs

- Ranked top-700 gene list by Pearson correlation to SLC2A1 per domain
- slc2a1_top700_correlation_comparison.csv with per-gene correlation values in both domains and spurious/discovered membership annotation columns

## How to apply

Load both preprocessed datasets: the original spatial transcriptomics data and the scMMGAN-generated spatial-domain output (the `output_fake2` mapping of the scRNA-seq domain), each prepared with MAGIC dropout correction, zero-centering/unit scaling, and reduction to 50 principal components. Compute Pearson correlations between the seed gene (SLC2A1) and every other gene in the original spatial data, rank by correlation coefficient, and retain the top 700 (cutoff of 0.1 on correlation). Repeat the identical ranking on the scMMGAN-generated data. Compare the two ranked lists: genes correlated in the original but absent in the mapped domain are annotated as spurious artifacts (e.g., FRAT2, CAMK2A, CANX, LRRC66, ZMIZ1-AS1, MTRNR2L8), while genes surfaced by the alignment are annotated as discovered candidates (e.g., AC092115.3, P2RX7, CCDC93, UTP25, BBS10). Write a per-gene table with correlation values in both domains and spurious/discovered membership annotations.

## Related tools

- **scMMGAN** (Generates the cross-domain output (output_fake2) whose SLC2A1 correlation landscape is compared against the original spatial data to distinguish spurious from discovered correlations) — github.com/KrishnaswamyLab/scMMGAN
- **MAGIC** (Corrects for dropout in both input domains before correlation ranking, per the article's preprocessing chain)
- **TensorFlow** (Framework used for the scMMGAN graph construction and training that produces the generated data being correlated) — github.com/KrishnaswamyLab/scMMGAN

## Examples

```
from scipy.stats import pearsonr; rho, p = pearsonr(slc2a1_expr, gene_expr)
```

## Evaluation signals

- Output file slc2a1_top700_correlation_comparison.csv exists and contains 700 ranked genes per domain
- The six spurious genes (FRAT2, CAMK2A, CANX, LRRC66, ZMIZ1-AS1, MTRNR2L8) are correctly flagged in the spurious annotation column with their original-domain correlation values
- The five discovered genes (AC092115.3, P2RX7, CCDC93, UTP25, BBS10) are correctly flagged in the discovered annotation column
- Low-correlation cutoff of 0.1 applied; per-gene Pearson values present for both original and scMMGAN-generated domains
- Novel associations (e.g., SLC2A1–P2RX7) are corroborated against prior biological knowledge (P2RX7 as a precursor to glucose transporters) before being reported as discoveries

## Limitations

- Novel gene correlations found by scMMGAN are potential discoveries that need further investigation with specifically designed experiments; they should not be treated as validated interactions
- scMMGAN alignments require continued evaluation through external verification measures
- Correlation ranking quality depends on the preprocessing choices (dropout correction, scaling, 50-PC reduction); the spurious genes are artifacts of spatial measurement and only meaningful relative to the original domain

## Evidence
