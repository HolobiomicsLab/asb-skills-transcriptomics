---
name: gene-expression-stratification-plotting
description: Use when you have scMMGAN-generated scRNA-seq profiles (output_fake2)
  from spatial transcriptomics input and need to test whether k=5 spectral k-means
  clusters or marker-gene strata (SLC2A1/CDK11A) retain the spatial organization of
  the tissue.
when_to_use_negative:
- Input is raw spatial count data that has not been mapped into the scRNA-seq domain
  by scMMGAN — the stratification is defined on generated profiles, not on native
  spatial spots
- The goal is training or tuning scMMGAN itself (loss monitoring, lambda_correspondence
  tuning) rather than downstream spatial visualization of its output
- No original spatial coordinates are available to project cluster labels back onto,
  so the spatial preservation check cannot be performed
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0092
tools:
- name: scMMGAN
  role: Generates the scRNA-seq-domain profiles from spatial transcriptomics input
    that are stratified and spatially plotted
  repo: github.com/KrishnaswamyLab/scMMGAN
- name: TensorFlow
  role: Runs the trained scMMGAN session to fetch the output_fake2 tensor of generated
    profiles
  repo: github.com/KrishnaswamyLab/scMMGAN
- name: MAGIC
  role: Dropout (manifold smoothing) correction applied to input matrices before mapping,
    ensuring profiles used for stratification are denoised
  repo: github.com/KrishnaswamyLab/scMMGAN
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.patter.2022.100577
    title: Single-cell multi-modal GAN reveals spatial patterns in single-cell data
      from triple-negative breast cancer
  source_content_hash: sha256:19d7ff080d6e11128f8b29c6a0ae8499840724137546226d1c92ed6223384f8b
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
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# gene-expression-stratification-plotting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Stratifies scMMGAN-generated single-cell profiles by marker-gene expression (SLC2A1-high vs CDK11A-high) and projects cluster labels and expression strata back onto original spatial-transcriptomics spot coordinates to render spatial maps (Figures 5B and 5C of the TNBC scMMGAN paper).

## When to use

Use when you have scMMGAN-generated scRNA-seq profiles (output_fake2) from spatial transcriptomics input and need to test whether k=5 spectral k-means clusters or marker-gene strata (SLC2A1/CDK11A) retain the spatial organization of the tissue.

## When NOT to use

- Input is raw spatial count data that has not been mapped into the scRNA-seq domain by scMMGAN — the stratification is defined on generated profiles, not on native spatial spots
- The goal is training or tuning scMMGAN itself (loss monitoring, lambda_correspondence tuning) rather than downstream spatial visualization of its output
- No original spatial coordinates are available to project cluster labels back onto, so the spatial preservation check cannot be performed

## Inputs

- Preprocessed spatial transcriptomics matrix (MAGIC-corrected, zero-centered, unit-scaled, 50 PCs) from the MDA-MB-231 xenograft
- Preprocessed MDA-MB-231 xenograft scRNA-seq matrix (same preprocessing) for the paired domain
- Trained scMMGAN TensorFlow session/graph
- Original spatial transcriptomics spot coordinates for projection
- Generated scRNA-seq profiles (output_fake2) and marker genes SLC2A1 and CDK11A

## Outputs

- Spatial cluster map of 5 spectral k-means cluster labels on original spot coordinates (Figure 5B analog)
- Spatially informed differentiation plot of SLC2A1-high vs CDK11A-high cells (Figure 5C analog)

## How to apply

First preprocess both the MDA-MB-231 xenograft scRNA-seq and spatial transcriptomics matrices with MAGIC dropout correction, zero-centering and unit scaling of each dimension, and reduction to 50 principal components, since the scMMGAN README recommends columns mostly in the interval [-10, 10]. Run the trained TensorFlow scMMGAN mapping that sends spatial data (x1) into the second (scRNA-seq) domain and collect profiles from the output_fake2 tensor. Cluster the generated profiles with spectral k-means at k=5, then project the 5 cluster labels back onto the original spatial spot coordinates to render the Figure 5B spatial cluster map. Stratify cells by SLC2A1-high versus CDK11A-high expression in the generated coordinates and plot their spatially informed differentiation as in Figure 5C. Validation is a visual expert check that the spatial map preserves the reported patterns, since no numeric criterion is reported.

## Related tools

- **scMMGAN** (Generates the scRNA-seq-domain profiles from spatial transcriptomics input that are stratified and spatially plotted) — github.com/KrishnaswamyLab/scMMGAN
- **TensorFlow** (Runs the trained scMMGAN session to fetch the output_fake2 tensor of generated profiles) — github.com/KrishnaswamyLab/scMMGAN
- **MAGIC** (Dropout (manifold smoothing) correction applied to input matrices before mapping, ensuring profiles used for stratification are denoised) — github.com/KrishnaswamyLab/scMMGAN

## Examples

```
gen = sess.run(output_fake2, feed_dict={x1: X_spatial}); labels = spectral_kmeans(gen, k=5); plot_spatial(coords, labels)
```

## Evaluation signals

- The k=5 spatial cluster map visually preserves the spatial patterns reported in Figure 5B of the paper (visual expert check; no numeric criterion is reported)
- SLC2A1-high vs CDK11A-high separation in the generated coordinates matches the spatially informed differentiation shown in Figure 5C
- Every generated profile's cluster label maps back to exactly one original spatial transcriptomics spot coordinate (label-to-spot 1:1 projection)
- Preprocessed input columns fall mostly in the interval [-10, 10] as recommended by the scMMGAN README
- Generated output is stable across training iterations before being used for stratification (README criterion: when output is stable, further training is unnecessary)

## Limitations

- Validation of the spatial preservation is a visual expert check; the paper reports no numeric criterion for cluster-map agreement
- GAN instability and mode collapse can deteriorate the model quickly across training iterations, so generated profiles used for stratification may be affected; early stopping and the geometric loss only informally mitigate these effects
- Aligned/generated results require continued evaluation through external verification measures; novel gene correlations found by scMMGAN are potential discoveries needing specifically designed experiments

## Evidence
