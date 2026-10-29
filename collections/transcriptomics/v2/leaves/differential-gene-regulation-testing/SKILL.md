---
name: differential-gene-regulation-testing
description: Use when you have manifold-aligned network coordinates for genes from
  two single-cell conditions (scTenifoldNet output) and need to call significantly
  differentially regulated genes with p-values and FDR.
when_to_use_negative:
- You have raw or normalized expression matrices but no constructed/aligned gene regulatory
  networks — run the full scTenifoldNet pipeline (scQC through manifoldAlignment)
  first
- You need differential expression at the expression level rather than differential
  regulation at the network/regulatory level — scTenifoldNet compares constructed
  scGRNs, not expression fold changes
- The two samples are highly dissimilar (e.g., different cell types) — the article
  notes scTenifoldNet is more suitable for comparing highly similar samples, such
  as two populations of cells of the same type
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: scTenifoldNet
  role: R package whose dRegulation function implements the Box-Cox transformation
    and chi-square-based differential regulation testing (step 6 of the pipeline)
  repo: github.com/cailab-tamu/scTenifoldNet
- name: R
  role: Runtime environment; stats::pchisq and stats::p.adjust (method = 'BH') provide
    the chi-square tail probabilities and FDR correction
- name: MASS
  role: MASS::boxcox provides the Box-Cox power transformation of the per-gene distance
    distribution used to derive Z-scores
- name: scTenifoldpy
  role: Python implementation of the scTenifold suite including the cross-condition
    GRN comparison (scTenifoldNet) workflow
  repo: github.com/qwerty239qwe/scTenifoldpy
provenance:
  source_task_ids:
  - task_006
  - task_004
  - task_007
  source_papers:
  - doi: 10.1016/j.patter.2020.100139
    title: 'scTenifoldNet: A Machine Learning Workflow for Constructing and Comparing
      Transcriptome-wide Gene Regulatory Networks from Single-Cell Data'
  source_content_hash: sha256:d0896e47e45198e396dc5292323016d194ffdb26afde3a9035f6ceed7ec12596
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/cailab-tamu/scTenifoldNet
  tools:
  - MASS
  - R
  - igraph
  - scTenifoldNet
  - scTenifoldpy
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# differential-gene-regulation-testing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Statistical testing of per-gene differential regulation in scTenifoldNet: manifold-alignment distances are Box-Cox transformed to Z-scores, scaled fold changes are tested against a chi-square distribution with 1 degree of freedom, and Benjamini-Hochberg FDR correction yields a ranked list of differentially regulated genes.

## When to use

Use when you have manifold-aligned network coordinates for genes from two single-cell conditions (scTenifoldNet output) and need to call significantly differentially regulated genes with p-values and FDR.

## When NOT to use

- You have raw or normalized expression matrices but no constructed/aligned gene regulatory networks — run the full scTenifoldNet pipeline (scQC through manifoldAlignment) first
- You need differential expression at the expression level rather than differential regulation at the network/regulatory level — scTenifoldNet compares constructed scGRNs, not expression fold changes
- The two samples are highly dissimilar (e.g., different cell types) — the article notes scTenifoldNet is more suitable for comparing highly similar samples, such as two populations of cells of the same type

## Inputs

- manifoldAlignment data frame from scTenifoldNet: 2 x n gene rows, d = 30 low-dimensional columns
- gene identifiers matching the rows of the input count matrices

## Outputs

- diffRegulation data frame with six columns: gene, distance, Z, FC, p.value, p.adj
- ranked gene list (sorted by differential regulation level) suitable for functional enrichment/GSEA

## How to apply

Load the manifoldAlignment data frame (2 x n genes rows x d = 30 columns) produced by scTenifoldNet's non-linear manifold alignment step. For each gene, compute the Euclidean distance between its coordinates under the two conditions across the aligned dimensions, then apply a Box-Cox power transformation (MASS::boxcox) to the distance vector and derive each gene's Z-score. Compute the fold change FC as the scaled quantity dj^2 * n / d_bar^2 and obtain upper-tail p-values from the chi-square distribution with one degree of freedom (stats::pchisq, lower.tail = FALSE). Adjust p-values with the Benjamini & Hochberg FDR procedure (stats::p.adjust, method = 'BH') and assemble the six-column table (gene, distance, Z, FC, p.value, p.adj). This step is implemented by the exported dRegulation function; significance calls are then made at an FDR cutoff (5% in the morphine-vs-mock application; 10% in the simulated example).

## Related tools

- **scTenifoldNet** (R package whose dRegulation function implements the Box-Cox transformation and chi-square-based differential regulation testing (step 6 of the pipeline)) — github.com/cailab-tamu/scTenifoldNet
- **R** (Runtime environment; stats::pchisq and stats::p.adjust (method = 'BH') provide the chi-square tail probabilities and FDR correction)
- **MASS** (MASS::boxcox provides the Box-Cox power transformation of the per-gene distance distribution used to derive Z-scores)
- **scTenifoldpy** (Python implementation of the scTenifold suite including the cross-condition GRN comparison (scTenifoldNet) workflow) — github.com/qwerty239qwe/scTenifoldpy

## Examples

```
outputHA <- scTenifoldNet(X = X, Y = Y, nc_nNet = 10, nc_nCells = 500, td_K = 3, qc_minLibSize = 30); head(outputHA$diffRegulation)
```

## Evaluation signals

- Output table has exactly six columns named gene, distance, Z, FC, p.value, p.adj
- Under H0 (identical input networks), no gene reaches significance at FDR < 0.1 (e.g., top gene p.adj = 0.6411916 in the README example)
- Under HA, perturbed genes are flagged: the six swapped genes (ng2, ng50, ng11, ng3, ng10, ng5) rank at the top with p.adj < 0.1
- FC statistics follow a chi-square(1) null — quantile-quantile plots of FC against chi-square(1) quantiles should fall on the diagonal for non-significant genes
- p.adj values equal Benjamini-Hochberg adjusted p.value values and genes are sorted by differential regulation level

## Limitations

- The chi-square approximation relies on the Box-Cox-transformed distances; validity depends on the distance distribution of the input manifold alignment
- scTenifoldNet (and hence this test) is more suitable for comparing highly similar samples, such as two populations of cells of the same type, restricting applicability
- Technical noise in scRNA-seq data is difficult to control, which can affect regulatory relationships inferred upstream and thus the test input
- Causal factors and their target genes remain unknown in many biological systems, limiting ground-truth validation of the significant-gene calls

## Evidence

- [methods] The DR test computes a p value for each gene using a χ2 test, adjusts the p values with a multiple testing correction, and selects significant differentially regulated genes using a 5% false discovery rate cutoff.: [evidence span withheld — anchor status: not_found]
