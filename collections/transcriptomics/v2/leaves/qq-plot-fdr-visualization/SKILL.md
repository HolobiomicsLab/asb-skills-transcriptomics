---
name: qq-plot-fdr-visualization
description: Use when a scTenifoldNet diffRegulation table (gene, distance, Z, FC,
  p.value, p.adj) needs visual goodness-of-fit assessment against chi-square(1) expectations,
  e.g. to reproduce the Figure 2C / readmeExample.png style H0-vs-HA comparison.
when_to_use_negative:
- Input is raw counts or unranked gene expression matrices — QQ plotting requires
  the dRegulation FC/p.adj output, not the raw data
- A ranked gene table alone is the deliverable — the plot summarizes fit visually
  but does not replace the BH-adjusted p-value ranking for downstream enrichment
- The comparison statistic is not expected to follow chi-square(1) under the null
  (e.g. untransformed distances) — the reference quantiles would be invalid
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scTenifoldNet
  role: Produces the diffRegulation data frame (FC, p.adj) that the QQ plot visualizes
  repo: https://github.com/cailab-tamu/scTenifoldNet
- name: R
  role: Base graphics (qqplot, rchisq, par/ifelse coloring) used to render the FDR-highlighted
    QQ plots
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.patter.2020.100139
    title: 'scTenifoldNet: A Machine Learning Workflow for Constructing and Comparing
      Transcriptome-wide Gene Regulatory Networks from Single-Cell Data'
  source_content_hash: sha256:9a7513317f000029932603203f5cd398b8fde4dbd30d0a6050165e76c9bce981
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/cailab-tamu/scTenifoldNet
  repo_url: https://github.com/cailab-tamu/scTenifoldNet
  tools:
  - R
  - igraph
  - scTenifoldNet
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# QQ-plot FDR visualization of differential regulation results

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Summarizes scTenifoldNet differential regulation output by plotting the FC statistic of each gene against chi-square(1) quantiles in a quantile-quantile plot, highlighting genes with Benjamini-Hochberg FDR < 0.1 in red so null vs perturbed behavior is visible at a glance.

## When to use

Use when a scTenifoldNet diffRegulation table (gene, distance, Z, FC, p.value, p.adj) needs visual goodness-of-fit assessment against chi-square(1) expectations, e.g. to reproduce the Figure 2C / readmeExample.png style H0-vs-HA comparison.

## When NOT to use

- Input is raw counts or unranked gene expression matrices — QQ plotting requires the dRegulation FC/p.adj output, not the raw data
- A ranked gene table alone is the deliverable — the plot summarizes fit visually but does not replace the BH-adjusted p-value ranking for downstream enrichment
- The comparison statistic is not expected to follow chi-square(1) under the null (e.g. untransformed distances) — the reference quantiles would be invalid

## Inputs

- scTenifoldNet diffRegulation data frame with columns gene, distance, Z, FC, p.value, p.adj (Benjamini-Hochberg adjusted)
- Both outputH0 and outputHA result lists for the paired null/alternative panels

## Outputs

- Two-panel QQ-plot of FC against chi-square(1) quantiles (e.g. readmeExample.png) with FDR < 0.1 genes highlighted in red
- Gene color vector derived from the p.adj < 0.1 threshold

## How to apply

Take the FC column of the diffRegulation data frame from scTenifoldNet (dRegulation step applies a Box-Cox power transformation and chi-square statistics to manifold-alignment distances). Draw a reference sample of 100 chi-square(1) deviates with rchisq(100, 1) and call base-R qqplot() with pch = 16, coloring each gene via rev(ifelse(diffRegulation$p.adj < 0.1, 'red', 'black')). Arrange H0 (X vs X) and HA (X vs Y) panels together with par(mfrow = c(2, 1)) so null behavior and perturbation signal can be compared. Rationale: under H0 no gene should attain FDR < 0.1 (no red points, FC quantiles track the chi-square line), while under HA only truly perturbed genes depart from the chi-square(1) quantiles and turn red.

## Related tools

- **scTenifoldNet** (Produces the diffRegulation data frame (FC, p.adj) that the QQ plot visualizes) — https://github.com/cailab-tamu/scTenifoldNet
- **R** (Base graphics (qqplot, rchisq, par/ifelse coloring) used to render the FDR-highlighted QQ plots)

## Examples

```
par(mfrow=c(2,1), mar=c(3,3,1,1), mgp=c(1.5,0.5,0)); set.seed(1); qChisq <- rchisq(100, 1); geneColor <- rev(ifelse(outputH0$diffRegulation$p.adj < 0.1, 'red', 'black')); qqplot(qChisq, outputH0$diffRegulation$FC, pch=16, col=geneColor)
```

## Evaluation signals

- Under H0 (X vs X) no gene is red: minimum p.adj = 0.6411916, so all p.adj > 0.1 and no points are highlighted
- Under HA the red points are exactly the six swapped genes ng2, ng50, ng11, ng3, ng10, ng5, all with p.adj < 0.1 (e.g. ng2 p.adj = 0.02414332)
- Null (black) FC quantiles track the chi-square(1) reference quantiles; only significant genes depart from the line
- Color vector is reproducible: rev(ifelse(output$diffRegulation$p.adj < 0.1, 'red', 'black')) with set.seed(1) for the rchisq reference sample
- Resulting figure matches the layout of the repository's readmeExample.png (H0 panel above HA panel)

## Limitations

- The chi-square(1) reference is valid only because dRegulation applies a Box-Cox power transformation to the manifold-alignment distances; other statistics would need different reference quantiles
- The red-highlight threshold is fixed at FDR < 0.1; genes just above the cutoff are indistinguishable from null points in the plot
- QQ plots are a summary device — they do not identify which genes are significant without consulting the ranked p.adj table

## Evidence
