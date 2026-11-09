---
name: single-cell-multi-omics-preprocessing
description: Use when paired single-cell modality matrices (e.g. DBIT-seq transcriptomic/proteomic
  or granulocyte scATAC-seq/scRNA-seq count matrices) must be preprocessed before
  training scMMGAN or its CycleGAN/CMAE baselines.
when_to_use_negative:
- Input data is already a preprocessed feature table in the desired scale (e.g. columns
  already mostly within [-10, 10]) — reapplying scaling and PCA would distort the
  geometry used by the correspondence loss
- The task requires raw count-level interpretation (e.g. differential expression on
  original gene counts) before mapping, since PCA reduction to 50 components discards
  per-gene resolution
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: MAGIC
  role: Manifold smoothing used to correct dropout in the single-cell modality matrices
    prior to scaling and PCA
  repo: github.com/KrishnaswamyLab/scMMGAN
- name: scMMGAN
  role: Target model whose README specifies the preprocessing goal that each input
    column be mostly in the interval [-10, 10]
  repo: github.com/KrishnaswamyLab/scMMGAN
- name: TensorFlow
  role: Framework in which the preprocessed 50-PC matrices are fed into the scMMGAN
    graph
provenance:
  source_task_ids:
  - task_001
  - task_002
  - task_004
  source_papers:
  - doi: 10.1016/j.patter.2022.100577
    title: Single-cell multi-modal GAN reveals spatial patterns in single-cell data
      from triple-negative breast cancer
  source_content_hash: sha256:c296487b76d69163de291b1b330fa1c97e96dc2669be6474bb4daecff078090b
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/KrishnaswamyLab/scMMGAN
  tools:
  - CMAE
  - CycleGAN
  - MAGIC
  - TensorFlow
  - scMMGAN
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# single-cell-multi-omics-preprocessing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Prepares paired single-cell multi-omic datasets (e.g. scRNA-seq/scATAC-seq or spatial transcriptomics/proteomics) for scMMGAN training by applying MAGIC dropout correction, per-dimension zero-centering and unit scaling, and reduction to 50 principal components, so that each input column is mostly in the interval [-10, 10].

## When to use

Use when paired single-cell modality matrices (e.g. DBIT-seq transcriptomic/proteomic or granulocyte scATAC-seq/scRNA-seq count matrices) must be preprocessed before training scMMGAN or its CycleGAN/CMAE baselines.

## When NOT to use

- Input data is already a preprocessed feature table in the desired scale (e.g. columns already mostly within [-10, 10]) — reapplying scaling and PCA would distort the geometry used by the correspondence loss
- The task requires raw count-level interpretation (e.g. differential expression on original gene counts) before mapping, since PCA reduction to 50 components discards per-gene resolution

## Inputs

- Paired single-cell count matrices from two modalities (e.g. DBIT-seq spatial transcriptomics and proteomics; paired granulocyte scATAC-seq and scRNA-seq) loaded into x1 and x2

## Outputs

- MAGIC-corrected, zero-centered, unit-scaled matrices reduced to 50 principal components per domain, ready for feeding into the neural network

## How to apply

First correct dropout in each modality matrix using the manifold smoothing method MAGIC. Then zero-center and unit scale each dimension of the corrected matrices so every input column is mostly in the interval [-10, 10], as recommended by the scMMGAN README for best results. Reduce each preprocessed matrix to 50 principal components. Downstream, if any scaling was inverted at output time (e.g. via output_fake/output_fake2 mappings), undo the preprocessing to return generated data to its original form. The same preprocessing must be applied identically to both domains X1 and X2 so correspondence losses and MMD/correlation metrics are computed on comparable scales.

## Related tools

- **MAGIC** (Manifold smoothing used to correct dropout in the single-cell modality matrices prior to scaling and PCA) — github.com/KrishnaswamyLab/scMMGAN
- **scMMGAN** (Target model whose README specifies the preprocessing goal that each input column be mostly in the interval [-10, 10]) — github.com/KrishnaswamyLab/scMMGAN
- **TensorFlow** (Framework in which the preprocessed 50-PC matrices are fed into the scMMGAN graph)

## Examples

```
import magic; X1_pc = sklearn.decomposition.PCA(n_components=50).fit_transform(scale().fit_transform(magic.MAGIC().fit_transform(x1)))
```

## Evaluation signals

- After preprocessing, each input column is mostly within the interval [-10, 10] per the scMMGAN README guidance
- Both domains X1 and X2 have been reduced to exactly 50 principal components with identical preprocessing steps applied
- Downstream scMMGAN training on the preprocessed data reproduces reported benchmarks, e.g. DBIT-seq correlations r≈0.154/0.152 for scMMGAN vs near-zero for CycleGAN/CMAE, and ATAC/RNA correlation r≈0.336
- Any scaling applied during preprocessing is correctly inverted when returning generated data (output_fake/output_fake2) to its original form

## Limitations

- The README notes that further preprocessing beyond scaling to [-10, 10] may be necessary depending on specific data properties, so the three-step recipe is not universally sufficient
- Preprocessing choices (MAGIC smoothing, 50 PCs) shape the diffusion geometry that the correspondence loss enforces; poor preprocessing will degrade alignment quality
- GAN training instability and mode collapse can still occur even with well-preprocessed inputs, per the article's discussion

## Evidence
