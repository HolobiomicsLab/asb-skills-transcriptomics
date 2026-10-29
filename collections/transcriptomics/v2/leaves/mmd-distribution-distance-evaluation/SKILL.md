---
name: mmd-distribution-distance-evaluation
description: Use when you have trained a cross-modality mapping model (e.g. scMMGAN)
  between two domains and need to quantify both distributional fidelity (MMD) and
  point-wise correspondence preservation (correlation of columns known to correspond).
when_to_use_negative:
- You have not yet trained a mapping model — MMD/correlation require generated G(x1)
  and G(x2) outputs; they cannot evaluate a single-domain dataset alone.
- Your two input domains were preprocessed with different pipelines (e.g. only one
  MAGIC-corrected or scaled to a different interval than [-10, 10]) — the distribution-
  and correlation-based metrics become incomparable across models.
- You only need to measure distributional similarity and do not care about point-wise
  correspondence — in that case MMD alone suffices, since all models achieve similarly
  low MMDs and correlation adds the discriminating signal.
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3308
tools:
- name: scMMGAN
  role: Source of the trained cross-domain mappings G(x1)/G(x2) whose outputs (output_fake2,
    output_fake) are fed into the MMD and correlation evaluation; the benchmark distinguishes
    its correspondence-preserving loss from distribution-only baselines
  repo: github.com/KrishnaswamyLab/scMMGAN
- name: TensorFlow
  role: Framework in which the generators/discriminators are built and trained and
    from which output_fake2/output_fake are generated for metric computation
- name: MAGIC
  role: Manifold-smoothing dropout correction applied to both input matrices before
    training so the evaluated mappings are computed on comparable preprocessed data
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.patter.2022.100577
    title: Single-cell multi-modal GAN reveals spatial patterns in single-cell data
      from triple-negative breast cancer
  source_content_hash: sha256:159393825604ff1f2d22b40f7394b9484fe14a9c5d82c0ebdf9c3c7fc2d0444f
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
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# mmd-distribution-distance-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark cross-modality GAN mappings (scMMGAN vs CycleGAN vs CMAE) by computing maximum mean discrepancy (MMD) and correlation between generated mappings G(x1)/G(x2) and the opposite modality, as demonstrated on DBIT-seq mouse embryo spatial transcriptomics/proteomics data.

## When to use

Use when you have trained a cross-modality mapping model (e.g. scMMGAN) between two domains and need to quantify both distributional fidelity (MMD) and point-wise correspondence preservation (correlation of columns known to correspond).

## When NOT to use

- You have not yet trained a mapping model — MMD/correlation require generated G(x1) and G(x2) outputs; they cannot evaluate a single-domain dataset alone.
- Your two input domains were preprocessed with different pipelines (e.g. only one MAGIC-corrected or scaled to a different interval than [-10, 10]) — the distribution- and correlation-based metrics become incomparable across models.
- You only need to measure distributional similarity and do not care about point-wise correspondence — in that case MMD alone suffices, since all models achieve similarly low MMDs and correlation adds the discriminating signal.

## Inputs

- Preprocessed DBIT-seq mouse embryo spatial transcriptomics matrix (MAGIC-corrected, zero-centered/unit-scaled, reduced to 50 PCs)
- Preprocessed DBIT-seq mouse embryo proteomics matrix (same preprocessing pipeline)
- Trained cross-modality mappings G(x1) and G(x2) from scMMGAN, CycleGAN, and CMAE (output_fake2 / output_fake in the scMMGAN code)

## Outputs

- table1_metrics.csv with MMD (both directions) and correlation (both directions) per model
- Comparative assessment of correspondence preservation: scMMGAN (r≈0.154/0.152) vs CycleGAN and CMAE (near zero)

## How to apply

First preprocess both input matrices identically — MAGIC dropout correction, zero-center and unit scale each dimension (columns mostly within [-10, 10] per the README), and reduction to 50 principal components. Train scMMGAN (generators with 128/256/512-neuron layers, discriminators with 1,024/512/256 neurons, generator optimized before discriminator each step, lambda_correspondence=10, lambda_cycle=1, learning rate 0.0001, batch size 256) and the CycleGAN/CMAE baselines using distribution-level losses only (no correspondence loss). Generate the cross-domain mappings: output_fake2 is G(x1) (mapping x1 into the second domain) and output_fake is G(x2). Compute MMD between G(x1) and x2 and between x1 and G(x2) for each model, plus the correlation between G(x1) and x2 and between x1 and G(x2) on columns known to correspond. Assemble the four metrics per model into table1_metrics.csv; expect all models to show similarly low MMDs, while scMMGAN alone preserves correspondence (average r≈0.154/0.152, baselines near zero) — MMD alone cannot distinguish correspondence-preserving mappings from distribution-matching ones.

## Related tools

- **scMMGAN** (Source of the trained cross-domain mappings G(x1)/G(x2) whose outputs (output_fake2, output_fake) are fed into the MMD and correlation evaluation; the benchmark distinguishes its correspondence-preserving loss from distribution-only baselines) — github.com/KrishnaswamyLab/scMMGAN
- **TensorFlow** (Framework in which the generators/discriminators are built and trained and from which output_fake2/output_fake are generated for metric computation)
- **MAGIC** (Manifold-smoothing dropout correction applied to both input matrices before training so the evaluated mappings are computed on comparable preprocessed data)

## Examples

```
import numpy as np; Gx1 = sess.run(output_fake2); Gx2 = sess.run(output_fake); r = np.corrcoef(Gx1.ravel(), x2.ravel())[0, 1]
```

## Evaluation signals

- All three models (scMMGAN, CycleGAN, CMAE) should achieve similarly low MMD values in both directions (G(x1) vs x2 and x1 vs G(x2)) — a high MMD indicates preprocessing or training failure.
- Correlation between corresponding columns should be r≈0.154/0.152 for scMMGAN across the two directions; CycleGAN and CMAE should yield near-zero correlation.
- Both directions must be evaluated: G(x1)→x2 and x1→G(x2) (output_fake2 and output_fake); reporting only one direction is incomplete.
- Input preprocessing sanity check: each input column should be mostly within the interval [-10, 10] before training/evaluation.
- All four metrics per model must be assembled into a single table1_metrics.csv before comparison.

## Limitations

- MMD alone is not discriminating: all models reach similarly low MMDs, so distributional match does not imply point-wise correspondence preservation; correlation on known corresponding columns is needed.
- GAN training is unstable and can deteriorate from effective to ineffective quickly across iterations, and may suffer mode collapse; early stopping and the geometric loss only informally mitigate these effects.
- The geometry-based loss is not 'plug-and-play' — a choice of distance between data points is still required, and the correspondence loss is only enforced on the first 10–20 eigenvectors.
- Preprocessing matters: the README notes further preprocessing beyond the [-10, 10] scaling may be necessary depending on specific data properties, and evaluation results depend on the MAGIC + scaling + 50-PC pipeline.

## Evidence
