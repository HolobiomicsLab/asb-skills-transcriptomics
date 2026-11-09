---
name: spectral-clustering-of-generated-expression-profiles
description: Use when you have GAN-generated single-cell expression profiles produced
  by mapping spatial transcriptomics data (x1) into the scRNA-seq domain, and you
  need to verify or derive spatially informed structure at single-cell resolution.
when_to_use_negative:
- The input is raw spatial-transcriptomics data that has not yet been mapped into
  the scRNA-seq domain by scMMGAN — clustering raw spots does not yield the single-cell-resolution
  profiles this skill targets
- Expression profiles have not undergone the paper's preprocessing (MAGIC dropout
  correction, zero-centering/unit scaling, 50-PC reduction), which the README states
  is needed for best results
- The biological question calls for a different number of clusters justified by the
  data rather than the k=5 used to reproduce Figure 5B
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scMMGAN
  role: Produces the generated scRNA-seq profiles (output_fake2) from spatial data
    x1 that are then spectrally clustered
  repo: github.com/KrishnaswamyLab/scMMGAN
- name: TensorFlow
  role: Runs the trained scMMGAN graph; sess.run on output_fake2 collects the generated
    profiles feeding the clustering
- name: MAGIC
  role: Upstream dropout (manifold smoothing) correction of the expression matrices
    before clustering
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.patter.2022.100577
    title: Single-cell multi-modal GAN reveals spatial patterns in single-cell data
      from triple-negative breast cancer
  source_content_hash: sha256:a100fb138509a5b3224173378f592f1183468e58057c2c6b2962042999d883d6
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# spectral-clustering-of-generated-expression-profiles

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Cluster scMMGAN-generated scRNA-seq profiles (mapped from spatial transcriptomics via output_fake2) with spectral k-means at k=5, then project cluster labels back onto original spatial-transcriptomics spot coordinates to test whether single-cell-resolution clusters retain the tissue's spatial organization (Figure 5B/5C of the TNBC scMMGAN paper).

## When to use

Use when you have GAN-generated single-cell expression profiles produced by mapping spatial transcriptomics data (x1) into the scRNA-seq domain, and you need to verify or derive spatially informed structure at single-cell resolution.

## When NOT to use

- The input is raw spatial-transcriptomics data that has not yet been mapped into the scRNA-seq domain by scMMGAN — clustering raw spots does not yield the single-cell-resolution profiles this skill targets
- Expression profiles have not undergone the paper's preprocessing (MAGIC dropout correction, zero-centering/unit scaling, 50-PC reduction), which the README states is needed for best results
- The biological question calls for a different number of clusters justified by the data rather than the k=5 used to reproduce Figure 5B

## Inputs

- Preprocessed spatial transcriptomics matrix (x1) from the MDA-MB-231 xenograft TNBC dataset (MAGIC-corrected, zero-centered, unit-scaled, 50 PCs)
- Preprocessed MDA-MB-231 xenograft scRNA-seq matrix (x2) with the same preprocessing
- Trained scMMGAN model (TensorFlow session with output_fake2 tensor)

## Outputs

- Cluster labels (k=5) for each generated scRNA-seq profile
- Spatial cluster map: the 5 labels plotted on the original spatial-transcriptomics spot coordinates (Figure 5B analog)
- Spatially informed differentiation plot stratifying SLC2A1-high vs CDK11A-high cells (Figure 5C analog)

## How to apply

First preprocess both input domains as the paper prescribes: MAGIC dropout correction, zero-centering and unit scaling of each dimension (README recommends columns mostly within [-10, 10]), and reduction to 50 principal components. Run the trained scMMGAN TensorFlow session to map spatial data x1 into the scRNA-seq domain, collecting generated profiles from the output_fake2 tensor. Cluster these generated profiles with spectral k-means using k=5 clusters. Project the 5 cluster labels back onto the original spatial-transcriptomics spot coordinates and render the spatial cluster map corresponding to Figure 5B; additionally stratify cells by SLC2A1-high versus CDK11A-high expression in the generated coordinates to reproduce the spatial differentiation of Figure 5C. Validation is visual-expert: the cluster map must preserve the reported spatial patterns of Figure 5B and the SLC2A1-high/CDK11A-high separation must match Figure 5C, as no numeric criterion is reported.

## Related tools

- **scMMGAN** (Produces the generated scRNA-seq profiles (output_fake2) from spatial data x1 that are then spectrally clustered) — github.com/KrishnaswamyLab/scMMGAN
- **TensorFlow** (Runs the trained scMMGAN graph; sess.run on output_fake2 collects the generated profiles feeding the clustering)
- **MAGIC** (Upstream dropout (manifold smoothing) correction of the expression matrices before clustering)

## Examples

```
from sklearn.cluster import SpectralClustering; labels = SpectralClustering(n_clusters=5, assign_labels='kmeans').fit_predict(generatedProfiles)
```

## Evaluation signals

- The spatial cluster map of the 5 spectral k-means labels preserves the spatial patterns reported in Figure 5B (visual expert check; no numeric criterion reported)
- SLC2A1-high versus CDK11A-high separation in the generated coordinates matches the differentiation shown in Figure 5C
- Generated profiles are collected from output_fake2 (x1 mapped into the second domain), not output_fake (x2 into the first domain)
- Preprocessing invariants hold: MAGIC applied, each dimension zero-centered and unit-scaled, columns mostly within [-10, 10], data reduced to 50 principal components
- Exactly k=5 clusters are returned and each original spatial spot receives exactly one projected label

## Limitations

- Validation of the spatial-pattern preservation is a visual expert check; no numeric criterion is reported
- GAN training instability and mode collapse may still be present in some contexts despite early stopping and the geometric loss, affecting the generated profiles being clustered
- The README notes further preprocessing beyond zero-centering/unit scaling may be necessary depending on specific data properties, so cluster quality is sensitive to preprocessing choices

## Evidence

- [results] Clustering the scMMGAN-generated scRNA-seq profiles with spectral k-means at k=5 and plotting clusters on their original spatial coordinates shows that the clusters preserve the spatial patterns of the tissue: [evidence span withheld — anchor status: not_found]
