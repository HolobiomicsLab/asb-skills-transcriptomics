---
name: point-wise-correlation-benchmarking
description: Use when two paired single-cell modalities (e.g., scRNA-seq and scATAC-seq
  of the same granulocyte cells) have been mapped between domains by scMMGAN, CycleGAN,
  or CMAE and you must verify that predictions retain point-wise correspondence with
  ground truth rather than only matching the marginal.
when_to_use_negative:
- Only unpaired modalities with no ground-truth measurements of the target modality
  — point-wise correlation cannot be computed without per-cell paired truth
- You only need distribution-level agreement between generated and true populations;
  MMD alone suffices and point-wise correlation would conflate alignment quality with
  correspondence
- The mapping has not yet been trained/converged — GAN training is unstable and the
  model can deteriorate from effective to ineffective across iterations, invalidating
  the comparison
edam_operation: http://edamontology.org/operation_0306
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scMMGAN
  role: Method under test whose geometry-preserving diffusion loss should yield point-wise
    correlation r≈0.336 on the granulocyte ATAC/RNA pair
  repo: github.com/KrishnaswamyLab/scMMGAN
- name: CycleGAN
  role: Baseline trained with the same architecture and cycle-loss coefficient of
    1 but without the correspondence loss; expected to be essentially uncorrelated
    with ground truth
- name: CMAE
  role: Baseline with separate encoders/decoders mapping into a shared space; expected
    to be essentially uncorrelated with ground truth
- name: MAGIC
  role: Dropout/manifold smoothing preprocessing applied to both domains before mapping
    so predicted and true values are comparable
- name: TensorFlow
  role: Framework in which scMMGAN and the baselines are trained and from which output_fake/output_fake2
    predictions are extracted
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.patter.2022.100577
    title: Single-cell multi-modal GAN reveals spatial patterns in single-cell data
      from triple-negative breast cancer
  source_content_hash: sha256:4aae746088cfb27159da178fd849fa69869cadef9601e8661e6236c9aac628e1
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
  edam_operation: http://edamontology.org/operation_0306
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# point-wise-correlation-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark for judging whether cross-modal single-cell mappings (e.g., ATAC-seq to RNA-seq) preserve point-wise correspondence: after generating predictions with scMMGAN, CycleGAN, and CMAE, compute the correlation between each predicted value and the ground-truth measurement of the target modality. A high point-wise correlation (scMMGAN r≈0.336 on the granulocyte pair) distinguishes geometry-preserving mappings from distribution-matching ones that are essentially uncorrelated with the truth.

## When to use

Use when two paired single-cell modalities (e.g., scRNA-seq and scATAC-seq of the same granulocyte cells) have been mapped between domains by scMMGAN, CycleGAN, or CMAE and you must verify that predictions retain point-wise correspondence with ground truth rather than only matching the marginal distribution.

## When NOT to use

- Only unpaired modalities with no ground-truth measurements of the target modality — point-wise correlation cannot be computed without per-cell paired truth
- You only need distribution-level agreement between generated and true populations; MMD alone suffices and point-wise correlation would conflate alignment quality with correspondence
- The mapping has not yet been trained/converged — GAN training is unstable and the model can deteriorate from effective to ineffective across iterations, invalidating the comparison

## Inputs

- Paired single-cell multi-modal datasets (domains X1 and X2, e.g., paired granulocyte scATAC-seq and scRNA-seq) preprocessed with MAGIC, zero-centering, unit scaling, and reduction to 50 principal components
- Trained cross-modal models: scMMGAN (correspondence loss coefficient 10, cycle-loss coefficient 1, lr 0.0001, batch size 256), CycleGAN, and CMAE predictions
- Generated predictions output_fake2 and output_fake from each model, with preprocessing inverted

## Outputs

- Point-wise correlation coefficient (Pearson r) between predicted and measured values of the target modality for each method
- Maximum mean discrepancy (MMD) between generated and true distributions for each method
- table2_metrics.csv summarizing per-method benchmark metrics

## How to apply

First preprocess each domain identically — MAGIC dropout correction, zero-centering and unit scaling of each dimension, and reduction to 50 principal components — so predicted and true values are directly comparable. Generate cross-domain predictions from each trained model using output_fake2 (mapping X1 into the second domain) and output_fake (mapping X2 into the first domain), inverting any preprocessing such as scaling. For each method, compute the point-wise correlation between predicted and measured values of the target modality, and optionally the maximum mean discrepancy (MMD) between generated and true distributions as a complementary distribution-level metric. Write the per-method results to table2_metrics.csv to reproduce the benchmark table. Validate: scMMGAN's recomputed point-wise correlation should fall within the reported ~0.313–0.358 range (average r=0.336), while CycleGAN and CMAE predictions should remain essentially uncorrelated with ground truth. The rationale is that the diffusion-geometry correspondence loss (inverse-distance kernel with k-NN adaptive bandwidth, row-normalized transition matrices, loss on the first 10–20 diffusion eigenvectors with sign correction at εr=-0.5) is what constrains the over-parameterized GAN to preserve per-cell correspondence, so only methods with such a loss should pass.

## Related tools

- **scMMGAN** (Method under test whose geometry-preserving diffusion loss should yield point-wise correlation r≈0.336 on the granulocyte ATAC/RNA pair) — github.com/KrishnaswamyLab/scMMGAN
- **CycleGAN** (Baseline trained with the same architecture and cycle-loss coefficient of 1 but without the correspondence loss; expected to be essentially uncorrelated with ground truth)
- **CMAE** (Baseline with separate encoders/decoders mapping into a shared space; expected to be essentially uncorrelated with ground truth)
- **MAGIC** (Dropout/manifold smoothing preprocessing applied to both domains before mapping so predicted and true values are comparable)
- **TensorFlow** (Framework in which scMMGAN and the baselines are trained and from which output_fake/output_fake2 predictions are extracted)

## Examples

```
import numpy as np; r = np.corrcoef(pred.ravel(), truth.ravel())[0,1]; metrics.to_csv('table2_metrics.csv')
```

## Evaluation signals

- Recomputed point-wise correlation for scMMGAN on the granulocyte ATAC/RNA pair falls within the reported ~0.313–0.358 range (average r=0.336)
- CycleGAN and CMAE point-wise correlations remain essentially uncorrelated with ground truth (near zero)
- MMD between generated and true distributions is computed and reported alongside correlation for each method
- Results are written to table2_metrics.csv matching the structure of the benchmark Table 2
- Predictions were de-preprocessed (e.g., scaling inverted) before correlation was computed, per the README's output-inversion guidance

## Limitations

- GAN training is unstable: the model can deteriorate from effective to ineffective quickly across training iterations, so the benchmark result depends on training trajectory and early stopping
- Distribution-level metrics do not penalize mode collapse; a model can match the marginal distribution while failing point-wise correspondence — hence both MMD and correlation are needed
- The geometry-based loss is not completely plug-and-play; it requires a choice of distance between data points, affecting reproducibility
- Aligned results require continued evaluation through external verification measures; benchmark correlations alone do not confirm biological validity

## Evidence
