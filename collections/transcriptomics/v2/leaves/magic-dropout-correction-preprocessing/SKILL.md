---
name: magic-dropout-correction-preprocessing
description: Use when raw scRNA-seq or spatial transcriptomics count matrices contain
  dropout artifacts and must be smoothed prior to GAN-based cross-domain alignment
  or clustering pipelines.
when_to_use_negative:
- Data is already imputed or the analysis does not rely on diffusion-geometry preservation
  (e.g., downstream methods sensitive to over-smoothed expression)
- Input is already a PCA-reduced feature table or a pre-aligned embedding rather than
  a gene-by-cell count matrix
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: MAGIC
  role: Manifold-smoothing method used to correct dropout in scRNA-seq and spatial
    transcriptomics matrices before scMMGAN alignment
  repo: github.com/KrishnaswamyLab/scMMGAN
- name: Chromium 10X
  role: Technology that produced the scRNA-seq data on which dropout correction is
    applied
- name: TensorFlow
  role: Framework hosting the downstream scMMGAN graph that consumes the MAGIC-corrected,
    scaled, 50-PC inputs
provenance:
  source_task_ids:
  - task_006
  - task_003
  source_papers:
  - doi: 10.1016/j.patter.2022.100577
    title: Single-cell multi-modal GAN reveals spatial patterns in single-cell data
      from triple-negative breast cancer
  source_content_hash: sha256:088665e3bd3166a1ff3ec02261b91130a4c0943cef5dd0386f4622963c2f6150
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
  - Chromium 10X
  - CycleGAN
  - MAGIC
  - TensorFlow
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# magic-dropout-correction-preprocessing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Applies MAGIC manifold smoothing to correct dropout in single-cell datasets (e.g., MDA-MB-231 xenograft scRNA-seq and TNBC spatial transcriptomics) before zero-centering, unit scaling, 50-component PCA reduction, and downstream scMMGAN alignment.

## When to use

Use when raw scRNA-seq or spatial transcriptomics count matrices contain dropout artifacts and must be smoothed prior to GAN-based cross-domain alignment or clustering pipelines.

## When NOT to use

- Data is already imputed or the analysis does not rely on diffusion-geometry preservation (e.g., downstream methods sensitive to over-smoothed expression)
- Input is already a PCA-reduced feature table or a pre-aligned embedding rather than a gene-by-cell count matrix

## Inputs

- Raw or count-level MDA-MB-231 xenograft scRNA-seq matrix (Chromium 10X)
- TNBC spatial transcriptomics spot-by-gene matrix

## Outputs

- Dropout-corrected, MAGIC-smoothed expression matrices
- Scaled (zero-centered, unit-variance) and 50-PC-reduced matrices ready as x1/x2 inputs to scMMGAN

## How to apply

Run MAGIC on the preprocessed MDA-MB-231 (Chromium 10X) scRNA-seq and spatial transcriptomics matrices to correct for dropout via manifold smoothing, so that the gene-expression manifold geometry used by scMMGAN's diffusion-geometry correspondence loss is reliable. After MAGIC, zero-center and unit-scale each dimension — the scMMGAN README notes best results when each input column is mostly in the interval [-10, 10] — then reduce to 50 principal components before feeding the data as x1/x2 into the TensorFlow graph. MAGIC is the first filter step in the standard pipeline (task_003 and task_006 workflows), preceding any PCA, correspondence-loss eigenvector construction (first 10–20 eigenvectors), or spectral k-means clustering (k=5) of generated profiles. Invert any scaling applied during preprocessing when converting scMMGAN outputs (output_fake, output_fake2) back to the original expression space.

## Related tools

- **MAGIC** (Manifold-smoothing method used to correct dropout in scRNA-seq and spatial transcriptomics matrices before scMMGAN alignment) — github.com/KrishnaswamyLab/scMMGAN
- **Chromium 10X** (Technology that produced the scRNA-seq data on which dropout correction is applied)
- **TensorFlow** (Framework hosting the downstream scMMGAN graph that consumes the MAGIC-corrected, scaled, 50-PC inputs)

## Examples

```
import magic; X_magic = magic.MAGIC().fit_transform(X_raw); X_scaled = (X_magic - X_magic.mean(0)) / X_magic.std(0)
```

## Evaluation signals

- Each input column of the MAGIC-corrected and scaled matrix lies mostly in the interval [-10, 10] per the scMMGAN README
- Downstream alignment reproduces scMMGAN's per-sample MSE of ~0.87–1.03 (vs ~1.6–2.1 for CycleGAN/CMAE), indicating preprocessing preserved geometry suitable for the correspondence loss
- Spectral k-means (k=5) clusters of scMMGAN-generated profiles, plotted back on spatial coordinates, retain the tissue's spatial patterns (Figure 5B)
- MAGIC runs before zero-centering/unit scaling and 50-PC reduction; ordering is preserved in the pipeline

## Limitations

- Preprocessing choices interact with GAN training stability: GANs are difficult to train due to adversarial losses and can suffer mode collapse; MAGIC-induced smoothing is part of the geometry the correspondence loss constrains, so aggressive correction may propagate
- The geometry-based downstream loss is not completely plug-and-play and still requires a choice of distance between data points
- Aligned/imputed results require continued evaluation through external verification measures

## Evidence
