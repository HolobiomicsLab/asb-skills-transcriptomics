---
name: correlation-heatmap-visualization
description: Use when you have estimated cell type proportions for a set of bulk samples
  (e.g., MuSiC Est.prop.weighted from 105 rat renal tubule segments) and need to visually
  validate that samples within each segment/condition show concordant deconvolution
  profiles.
when_to_use_negative:
- Cell type proportions cannot be estimated with TPM as the input — correlations should
  be computed on proportions derived from raw read counts, not TPM-normalized bulk
  data
- You only have single-sample or single-condition bulk data, since pairwise sample-to-sample
  correlation requires multiple samples per segment/condition to show clustering
- The goal is absolute cell type quantification rather than cross-sample concordance,
  since MuSiC recovers trends of proportion change across subjects, not absolute proportions
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0092
tools:
- name: MuSiC
  role: Produces the estimated cell type proportions (music_prop / music_prop.cluster,
    Est.prop.weighted) whose pairwise sample correlations are visualized
  repo: https://github.com/xuranw/MuSiC
- name: ggplot2
  role: R package used to render the pairwise correlation heatmap and save it as rat_segment_proportion_heatmap.png
- name: R
  role: Statistical environment in which the correlation computation and heatmap rendering
    are implemented
  repo: https://github.com/xuranw/MuSiC
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-018-08023-x
    title: Bulk tissue cell type deconvolution with multi-subject single-cell expression
      reference
  source_content_hash: sha256:b028e8c037f2b292e3c6d13b28c9718ce3360f1be1a85991faa717c8aeda6c69
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xuranw/MuSiC
  tools:
  - Biobase
  - MuSiC
  - R
  - ggplot2
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0092
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# correlation-heatmap-visualization

## Summary

Compute pairwise sample-to-sample correlations of MuSiC-estimated cell type proportions across bulk RNA-seq samples and render them as a heatmap (ggplot2/heatmap functions) to verify that samples from the same biological segment cluster together.

## When to use

Use when you have estimated cell type proportions for a set of bulk samples (e.g., MuSiC Est.prop.weighted from 105 rat renal tubule segments) and need to visually validate that samples within each segment/condition show concordant deconvolution profiles.

## When NOT to use

- Cell type proportions cannot be estimated with TPM as the input — correlations should be computed on proportions derived from raw read counts, not TPM-normalized bulk data
- You only have single-sample or single-condition bulk data, since pairwise sample-to-sample correlation requires multiple samples per segment/condition to show clustering
- The goal is absolute cell type quantification rather than cross-sample concordance, since MuSiC recovers trends of proportion change across subjects, not absolute proportions

## Inputs

- Est.prop.weighted proportion matrix (samples x cell types) from MuSiC music_prop or music_prop.cluster
- Sample-to-segment annotation (e.g., the 105 rat micro-dissected tubule bulk samples from GSE56743 labeled by renal segment)

## Outputs

- Pairwise sample-to-sample correlation matrix of estimated cell type proportions
- Correlation heatmap image file (e.g., rat_segment_proportion_heatmap.png) rendered with ggplot2/heatmap functions

## How to apply

After estimating cell type proportions for all bulk samples with MuSiC's music_prop (or music_prop.cluster with clusters.type), take the Est.prop.weighted matrix (samples x cell types) and compute all pairwise Pearson correlations between samples. Render the correlation matrix as a heatmap using ggplot2 or base R heatmap functions, saving the figure with a descriptive name such as rat_segment_proportion_heatmap.png. Rationale: samples from the same anatomical segment should share similar dominant cell type proportions (e.g., DCT segments dominated by DCT cells), so they should cluster together with high within-segment correlation, providing a visual validation of the deconvolution against each segment's mapped position along the nephron. Interpret the heatmap jointly with the dominant recovered cell type per segment to confirm anatomical agreement.

## Related tools

- **MuSiC** (Produces the estimated cell type proportions (music_prop / music_prop.cluster, Est.prop.weighted) whose pairwise sample correlations are visualized) — https://github.com/xuranw/MuSiC
- **ggplot2** (R package used to render the pairwise correlation heatmap and save it as rat_segment_proportion_heatmap.png)
- **R** (Statistical environment in which the correlation computation and heatmap rendering are implemented) — https://github.com/xuranw/MuSiC

## Examples

```
library(MuSiC); est <- music_prop(bulk.eset = rat.eset, sc.eset = mouse.eset, clusters = 'cellType', samples = 'sampleID'); cor.mat <- cor(t(est$Est.prop.weighted)); pheatmap(cor.mat, filename = 'rat_segment_proportion_heatmap.png')
```

## Evaluation signals

- Samples from each renal segment cluster together in the heatmap (high within-segment correlation, visible block structure along the diagonal)
- The dominant deconvolved cell type per segment agrees with the segment's anatomical position along the nephron (e.g., DCT cells recovered from the DCT segment)
- The saved figure exists as rat_segment_proportion_heatmap.png and matches the block/cluster structure of the reported figure
- The proportion matrix fed to the correlation step is Est.prop.weighted from music_prop, with columns summing to 1 per sample (proportions, not TPM)
- High sample-to-sample correlation within segments, consistent with the paper's report of high within-segment sample correlation

## Limitations

- MuSiC recovers trends of cell type proportion change across subjects, not absolute cell type proportions, so heatmap clustering reflects relative concordance only
- Collinearity among closely related cell types can make estimated proportions unreliable, especially for rare cell types, distorting correlation structure; grouping cell types hierarchically mitigates this
- Most scRNA-seq references do not reflect true cell type proportions in intact tissues, so correlations validate deconvolution consistency, not true in-tissue composition

## Evidence

- [methods] Compute pairwise sample-to-sample correlations of the estimated proportions and render the correlation heatmap with ggplot2/heatmap functions: [evidence span withheld — anchor status: not_found]
- [methods] Estimate cell type proportions for all 105 rat bulk samples with music_prop: [evidence span withheld — anchor status: not_found]
- [methods] heatmap must show that samples from each renal segment cluster together: [evidence span withheld — anchor status: not_found]
- [discussion] Collinearity among closely related cell types: "Correlation in gene expression can lead to collinearity, making it difficult to reliably estimate cell type proportions, especially for less frequent and rare cell types."
- [discussion] TPM limitation: "However, cell type proportions cannot be estimated with TPM as the input."
