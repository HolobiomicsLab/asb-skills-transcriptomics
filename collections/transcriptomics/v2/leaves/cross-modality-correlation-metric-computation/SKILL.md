---
name: cross-modality-correlation-metric-computation
description: Use when two modalities are jointly measured (e.g. DBIT-seq spatial transcriptomics
  and proteomics) and you must score whether a learned cross-domain mapping preserves
  point-wise correspondence, not just distributional similarity.
when_to_use_negative:
- The modalities are not jointly measured, so no columns are known to correspond and
  the correlation metric cannot be meaningfully interpreted.
- The goal is only distribution-level comparison; CycleGAN/CMAE-style baselines with
  distribution-level losses alone already answer the question and no correspondence
  evaluation is needed.
- The input matrices have not been preprocessed (no MAGIC dropout correction, no zero-centering/unit
  scaling, not reduced to 50 PCs), violating the [-10, 10] column-range requirement
  for feeding the network.
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scMMGAN
  role: Generates the cross-domain mappings (output_fake2, output_fake) whose point-wise
    correspondence is being scored
  repo: github.com/KrishnaswamyLab/scMMGAN
- name: TensorFlow
  role: Framework in which the generator/discriminator graph and training loop are
    built and mappings are produced via sess.run
- name: MAGIC
  role: Manifold-smoothing dropout correction applied to both modalities before metrics
    are computed
- name: CycleGAN
  role: Baseline trained with distribution-level losses only, used as the near-zero-correlation
    comparison point
- name: CMAE
  role: Baseline trained with distribution-level losses only, used as the near-zero-correlation
    comparison point
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.patter.2022.100577
    title: Single-cell multi-modal GAN reveals spatial patterns in single-cell data
      from triple-negative breast cancer
  source_content_hash: sha256:ce48f237cb2d8c62f6234b56e473b78a1a42323bf19bb7caf250612cca87d7ce
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# cross-modality-correlation-metric-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes MMD and column-wise correlation between a trained model's cross-domain mappings (e.g. scMMGAN's G(x1) into the proteomic space) and the jointly measured target modality, to quantify how well point-wise correspondence is preserved on jointly measured DBIT-seq spatial transcriptomics/proteomics data.

## When to use

Use when two modalities are jointly measured (e.g. DBIT-seq spatial transcriptomics and proteomics) and you must score whether a learned cross-domain mapping preserves point-wise correspondence, not just distributional similarity.

## When NOT to use

- The modalities are not jointly measured, so no columns are known to correspond and the correlation metric cannot be meaningfully interpreted.
- The goal is only distribution-level comparison; CycleGAN/CMAE-style baselines with distribution-level losses alone already answer the question and no correspondence evaluation is needed.
- The input matrices have not been preprocessed (no MAGIC dropout correction, no zero-centering/unit scaling, not reduced to 50 PCs), violating the [-10, 10] column-range requirement for feeding the network.

## Inputs

- DBIT-seq mouse embryo spatial transcriptomics matrix (x1) preprocessed with MAGIC, zero-centering/unit scaling, and 50-PC reduction
- DBIT-seq mouse embryo proteomics matrix (x2) preprocessed identically
- Trained model checkpoints/mappings from scMMGAN, CycleGAN, and CMAE (variables output_fake2 and output_fake for the two mapping directions)
- Definition of columns known to correspond between the two modalities (jointly measured markers/genes)

## Outputs

- Per-model MMD values for G(x1)→x2 and x1→G(x2)
- Per-model correlations between generated and measured values in corresponding columns for both directions
- table1_metrics.csv containing the four metrics (2 MMD, 2 correlation) per model

## How to apply

Preprocess each modality identically before mapping: correct dropout with MAGIC, zero-center and unit-scale each dimension, and reduce to 50 principal components, ensuring columns mostly lie in [-10, 10] as the scMMGAN README requires. Train scMMGAN in TensorFlow (generators 128/256/512, discriminators 1024/512/256, generator optimized via train_op_G then discriminator via train_op_D, lambda_correspondence=10, lambda_cycle=1, lr=0.0001, batch 256), and train CycleGAN and CMAE baselines with distribution-level losses only. Take the generated mappings from the session output: output_fake2 (x1 mapped into the second domain) and output_fake (x2 mapped into the first domain). For each of the three models compute MMD between G(x1) and x2 and between x1 and G(x2), plus the correlation between the mapped and measured values in columns known to correspond, in both directions. Correct application is indicated by similar low MMDs across models but a preserved-correlation advantage for scMMGAN (average r≈0.154/0.152) versus near-zero correlations for CycleGAN and CMAE; assemble the four metrics per model into table1_metrics.csv.

## Related tools

- **scMMGAN** (Generates the cross-domain mappings (output_fake2, output_fake) whose point-wise correspondence is being scored) — github.com/KrishnaswamyLab/scMMGAN
- **TensorFlow** (Framework in which the generator/discriminator graph and training loop are built and mappings are produced via sess.run)
- **MAGIC** (Manifold-smoothing dropout correction applied to both modalities before metrics are computed)
- **CycleGAN** (Baseline trained with distribution-level losses only, used as the near-zero-correlation comparison point)
- **CMAE** (Baseline trained with distribution-level losses only, used as the near-zero-correlation comparison point)

## Examples

```
r12 = [np.corrcoef(output_fake2[:, i], x2[:, i])[0, 1] for i in range(x2.shape[1])]
```

## Evaluation signals

- All three models yield similarly low MMD values in both directions — a large MMD indicates preprocessing or mapping failure rather than metric failure.
- scMMGAN's average correlation across the two mapping directions is approximately r=0.154/0.152 in columns known to correspond.
- CycleGAN and CMAE baselines yield near-zero correlations, so a non-zero correlation from these baselines signals a data leakage or correspondence-column definition error.
- Mapping variables match the README contract: output_fake2 must be x1 mapped into the second domain and output_fake x2 mapped into the first domain before metric computation.
- Generated values fall in a comparable range to the preprocessed targets (columns mostly within [-10, 10]) before correlation/MMD are computed.

## Limitations

- All models achieve similarly low MMDs, so MMD alone cannot distinguish correspondence-preserving models — the correlation metric is the differentiator.
- GAN training instability can make a model deteriorate from effective to ineffective quickly across training iterations, changing metric outcomes run-to-run; early stopping and the geometric loss only informally mitigate this.
- The geometry-based loss (and by extension the correspondence evaluation) is not completely 'plug-and-play' since a distance choice between data points is required.
- Aligned results should be further validated through external verification measures; novel correlations remain potential discoveries needing specifically designed experiments.

## Evidence

- [results] Metric rationale from findings: [evidence span withheld — anchor status: not_found]
