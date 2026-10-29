---
name: cross-modal-gene-list-comparison
description: Use when a target gene's correlated-gene ranking from original spatial
  data must be compared against the same ranking computed on a GAN-mapped scRNA-seq
  output to flag spurious vs. newly discovered candidate associations.
when_to_use_negative:
- The two modalities have not been aligned by a cross-domain mapping (e.g., no scMMGAN
  `output_fake2` exists), since rankings would come from unrelated feature spaces
- The datasets have not been preprocessed identically (MAGIC dropout correction, zero-centering/unit
  scaling, 50 principal components), making correlation ranks incomparable
- The goal is differential expression or clustering rather than distinguishing spurious
  from cross-modal novel gene correlations
edam_operation: http://edamontology.org/operation_3463
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: scMMGAN
  role: Generates the cross-domain mapped data (`output_fake2`) whose gene-correlation
    ranking is compared against the original spatial transcriptomics ranking
  repo: github.com/KrishnaswamyLab/scMMGAN
- name: MAGIC
  role: Dropout (manifold smoothing) correction applied to both domains before correlation
    ranking so lists are comparable
- name: TensorFlow
  role: Graph construction and training of the scMMGAN generator/discriminator that
    produce the mapped domain
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.patter.2022.100577
    title: Single-cell multi-modal GAN reveals spatial patterns in single-cell data
      from triple-negative breast cancer
  source_content_hash: sha256:959e0bbca4edf2036e44b6b75a0933484882281cbe0bc73550619ee06881395e
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
  edam_operation: http://edamontology.org/operation_3463
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# cross-modal-gene-list-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Compares ranked gene-correlation lists computed independently in two data domains — original spatial transcriptomics and scMMGAN-generated scRNA-seq mapped into the spatial domain — to separate spurious correlations (artifacts of spatial measurement) from novel associations revealed by the cross-modal mapping.

## When to use

Use when a target gene's correlated-gene ranking from original spatial data must be compared against the same ranking computed on a GAN-mapped scRNA-seq output to flag spurious vs. newly discovered candidate associations.

## When NOT to use

- The two modalities have not been aligned by a cross-domain mapping (e.g., no scMMGAN `output_fake2` exists), since rankings would come from unrelated feature spaces
- The datasets have not been preprocessed identically (MAGIC dropout correction, zero-centering/unit scaling, 50 principal components), making correlation ranks incomparable
- The goal is differential expression or clustering rather than distinguishing spurious from cross-modal novel gene correlations

## Inputs

- Preprocessed original spatial transcriptomics dataset (MAGIC-corrected, zero-centered/unit-scaled, reduced to 50 principal components)
- scMMGAN-generated spatial-domain output (`output_fake2` mapping of the scRNA-seq domain into the spatial domain), preprocessed identically
- Target gene identifier around which correlations are ranked (e.g., SLC2A1)

## Outputs

- Ranked top-N (e.g., 700) correlated-gene list per domain with Pearson correlation coefficients
- Comparison table (e.g., slc2a1_top700_correlation_comparison.csv) with per-gene correlation values in both domains and spurious/discovered membership annotations

## How to apply

First preprocess both domains identically: correct dropout with MAGIC, zero-center and unit scale each dimension, and reduce to 50 principal components, so that each input column is mostly in the interval [-10, 10] as the scMMGAN README advises. Run scMMGAN (TensorFlow graph; generator optimized with train_op_G, then discriminator with train_op_D) and take `output_fake2`, which maps the scRNA-seq domain into the spatial domain. Compute Pearson correlations between the target gene (e.g., SLC2A1) and every other gene in both the original spatial data and the generated data, rank genes by correlation coefficient, and retain the top 700 in each domain. Compare the two ranked lists using a low-correlation cutoff of 0.1: genes correlated in the original spatial data but absent/artifactual there (e.g., FRAT2, CAMK2A, CANX, LRRC66, ZMIZ1-AS1, MTRNR2L8) are annotated as spurious, while genes surfaced by the scMMGAN alignment (e.g., AC092115.3, P2RX7, CCDC93, UTP25, BBS10) are annotated as discovered; corroborate discoveries against external biology (P2RX7 as precursor to glucose transporters). Write a table with per-gene correlation values in both domains plus spurious/discovered membership annotations, e.g., slc2a1_top700_correlation_comparison.csv.

## Related tools

- **scMMGAN** (Generates the cross-domain mapped data (`output_fake2`) whose gene-correlation ranking is compared against the original spatial transcriptomics ranking) — github.com/KrishnaswamyLab/scMMGAN
- **MAGIC** (Dropout (manifold smoothing) correction applied to both domains before correlation ranking so lists are comparable)
- **TensorFlow** (Graph construction and training of the scMMGAN generator/discriminator that produce the mapped domain)

## Examples

```
import pandas as pd; corr = pd.DataFrame(X).corrwith(pd.DataFrame(X)['SLC2A1'], method='pearson').sort_values(ascending=False).head(700)
```

## Evaluation signals

- The output file exists and contains exactly 700 ranked genes per domain (original spatial and scMMGAN-generated)
- The six spurious genes (FRAT2, CAMK2A, CANX, LRRC66, ZMIZ1-AS1, MTRNR2L8) are correctly flagged in the spurious annotation column
- The five discovered genes (AC092115.3, P2RX7, CCDC93, UTP25, BBS10) are correctly flagged in the discovered annotation column
- Per-gene Pearson correlation values are present for both domains and the low-correlation cutoff of 0.1 was applied consistently
- Both input domains used identical preprocessing (MAGIC, zero-centering/unit scaling, 50 PCs) before ranking

## Limitations

- GAN training can be unstable or suffer mode collapse; mitigation via early stopping and the geometric loss is only informally observed and effects may still be present
- Novel gene correlations found by scMMGAN are potential discoveries that need further investigation with specifically designed experiments; aligned results require continued external verification
- Preprocessing choice matters: the README notes the best results require each input column mostly in [-10, 10], and any scaling used to shape the data should be inverted to return generated data to its original form

## Evidence

- [other] With n=700 and a low-correlation cutoff of 0.1, six spurious SLC2A1 correlations (FRAT2, CAMK2A, CANX, LRRC66, ZMIZ1-AS1, MTRNR2L8) and five scMMGAN-discovered correlations (AC092115.3, P2RX7, CCDC93, UTP25, BBS10) were found, with P2RX7 corroborated as a precursor to glucose transporters.: [evidence span withheld — anchor status: not_found]
