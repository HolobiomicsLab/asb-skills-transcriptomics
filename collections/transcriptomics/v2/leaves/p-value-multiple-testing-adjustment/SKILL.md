---
name: p-value-multiple-testing-adjustment
description: Use when combining two independent differential-expression p-values (sender
  ligand, receiver receptor) into one LR-pair significance and needing a multiple-testing-adjusted
  p-value to filter candidate ligand-receptor associations.
when_to_use_negative:
- When LR-pair significance should incorporate pathway activity in receiver cells
  (CommPath's downstream filterLR step) rather than expression-based statistical identification
  alone
- When the input is not cluster-level differential-expression results but, e.g., already
  aggregated interaction counts or pathway-level statistics
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: CommPath
  role: R package whose findLRmarker and findLRpairs functions compute ligand/receptor
    differential expression, combine the p-values into P.val.LR, and produce P.val.adj.LR
    under the specified thresholds
  repo: https://github.com/yingyonghui/CommPath
- name: R
  role: Runtime environment in which CommPath (installed via devtools::install_github)
    is executed
  repo: https://github.com/yingyonghui/CommPath
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2022.10.028
    title: 'CommPath: An R package for inference and analysis of pathway-mediated
      cell-cell communication chain from single-cell transcriptomics'
  source_content_hash: sha256:410b28472c20218a011f27d7b287f3898255221f36d19f6ffb48f05244c93dde
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/yingyonghui/CommPath
  repo_url: https://github.com/yingyonghui/CommPath
  tools:
  - CommPath
  - R
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# p-value-multiple-testing-adjustment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A statistical skill from CommPath that combines ligand and receptor differential-expression p-values into a single LR-pair significance (P_lr,AB = 1 − (1−P_l,A)(1−P_r,B)) and reports an adjusted P value for multiple testing, enabling statistically grounded prioritization of ligand-receptor associations between cell clusters.

## When to use

Use when combining two independent differential-expression p-values (sender ligand, receiver receptor) into one LR-pair significance and needing a multiple-testing-adjusted p-value to filter candidate ligand-receptor associations.

## When NOT to use

- When LR-pair significance should incorporate pathway activity in receiver cells (CommPath's downstream filterLR step) rather than expression-based statistical identification alone
- When the input is not cluster-level differential-expression results but, e.g., already aggregated interaction counts or pathway-level statistics

## Inputs

- Differential-expression statistics for marker ligands and receptors per cell cluster (log2FC, adjusted p-values) from findLRmarker on a CommPath object
- Normalized, log-transformed gene-by-cell expression matrix (e.g. tumor.expr) with matching cell identity labels (e.g. tumor.label)
- Threshold parameters logFC.thre and p.thre (e.g. 0 and 0.01)

## Outputs

- Per-LR-pair statistics table with columns log2FC.LR (interaction intensity), P.val.LR (combined p-value), and P.val.adj.LR (adjusted p-value)
- Counts of significant LR pairs and overall interaction intensity per sender-receiver cluster pair (tumor.obj@interact[['InteractNumer']] and [['InteractGene']])

## How to apply

After running findLRmarker (e.g. method = 'wilcox.test') to obtain log2FC and adjusted p-values for ligands in the sender cluster CA and receptors in the receiver cluster CB, run findLRpairs with logFC.thre = 0 and p.thre = 0.01 to identify statistically significant LR associations. For each LR pair, CommPath computes an intensity score as the product of the two log2FC values (Intensity_lr,AB = log2FC_l,A × log2FC_r,B) and a combined p-value P_lr,AB = 1 − (1−P_l,A)(1−P_r,B). The per-pair output includes P.val.LR and its multiple-testing-adjusted counterpart P.val.adj.LR, which are used to retain only pairs passing the p.thre = 0.01 threshold. Rationale: multiplicative p-value combination captures concordant upregulation of both ligand and receptor, while adjustment controls false positives across the large number of tested LR pairs.

## Related tools

- **CommPath** (R package whose findLRmarker and findLRpairs functions compute ligand/receptor differential expression, combine the p-values into P.val.LR, and produce P.val.adj.LR under the specified thresholds) — https://github.com/yingyonghui/CommPath
- **R** (Runtime environment in which CommPath (installed via devtools::install_github) is executed) — https://github.com/yingyonghui/CommPath

## Examples

```
tumor.obj <- findLRpairs(object = tumor.obj, logFC.thre = 0, p.thre = 0.01)
```

## Evaluation signals

- Output LR-pair table exists and contains the required columns log2FC.LR, P.val.LR, and P.val.adj.LR
- All retained LR pairs pass the specified filters: log2FC.LR > logFC.thre (0) and P.val.LR (or P.val.adj.LR) ≤ p.thre (0.01)
- Combined P.val.LR values satisfy the multiplicative relationship P_lr,AB = 1 − (1−P_l,A)(1−P_r,B) given the ligand and receptor p-values
- Interaction intensity equals the product of the ligand and receptor log2FC values from the sender and receiver cluster differential tests
- Cluster-pair counts of significant pairs in tumor.obj@interact[['InteractNumer']] match the number of rows passing thresholds in the per-pair table

## Limitations

- This expression-based identification considers only intercellular signaling; CommPath's later filterLR/filterPath steps are needed to prioritize LR interactions associated with activated pathways in receiver cells
- Adjustment is over many LR-pair tests, so stringent p.thre (e.g. 0.01) may discard weak but biologically relevant associations
- FindLRmarker supports only t.test or wilcox.test for the underlying differential-expression p-values, constraining the statistical basis of the combined p-value

## Evidence

- [methods] Combined p-value formula from source card: [evidence span withheld — anchor status: not_found]
