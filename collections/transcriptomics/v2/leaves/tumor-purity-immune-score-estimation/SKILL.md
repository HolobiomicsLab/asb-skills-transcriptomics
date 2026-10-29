---
name: tumor-purity-immune-score-estimation
description: Use when you need to quantify immune infiltration or tumor purity from
  bulk RNA-seq (e.g., TCGA RSEM-normalized expression) and test whether marker gene-set
  expression correlates with microenvironment features.
when_to_use_negative:
- Input is single-cell transcriptome data rather than bulk tumor expression — use
  PreCanCell cell classification instead
- The analysis goal is survival stratification or tumor-vs-normal differential expression
  rather than microenvironment/purity estimation
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2640
- http://edamontology.org/topic_2229
tools:
- name: ESTIMATE
  role: Estimates immune score, stromal score, and tumor purity from bulk RNA-seq
    expression
- name: R
  role: Environment for log2(x+1) transformation, Spearman correlation computation,
    and ESTIMATE execution
- name: PreCanCell
  role: Upstream algorithm that identifies the 73 TMGs and 186 NMGs whose bulk expression
    is correlated with immune score and purity
  repo: https://github.com/WangX-Lab/PreCanCell
- name: ssGSEA
  role: Gene-set enrichment scoring used alongside ESTIMATE for molecular-feature
    correlation analyses
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.csbj.2023.07.009
    title: 'PreCanCell: An ensemble learning algorithm for predicting cancer and non-cancer
      cells from single-cell transcriptomes'
  source_content_hash: sha256:f76bced32a3620b24e03f0526110ea404324ae69c7673b6e1efcbb4a766d421f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WangX-Lab/PreCanCell
  tools:
  - ESTIMATE
  - PreCanCell
  - R
  - Student's t-test
  - ssGSEA
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2640
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# tumor-purity-immune-score-estimation

## Summary

Estimate immune score, stromal score, and tumor purity for bulk tumor transcriptomes (e.g., TCGA RSEM RNA-seq) with the ESTIMATE algorithm, then relate these microenvironment metrics to gene-set expression (TMGs/NMGs) to validate single-cell-derived marker genes at the bulk level.

## When to use

Use when you need to quantify immune infiltration or tumor purity from bulk RNA-seq (e.g., TCGA RSEM-normalized expression) and test whether marker gene-set expression correlates with microenvironment features.

## When NOT to use

- Input is single-cell transcriptome data rather than bulk tumor expression — use PreCanCell cell classification instead
- The analysis goal is survival stratification or tumor-vs-normal differential expression rather than microenvironment/purity estimation

## Inputs

- RSEM-normalized RNA-seq gene expression matrix for TCGA samples (33 cancer types) from the GDC portal
- TCGA clinical data from the GDC portal
- Gene set lists (e.g., 73 TMGs and 186 NMGs from single-cell DEG analysis)

## Outputs

- Per-sample ESTIMATE immune score, stromal score, and tumor purity values
- Per-sample mean expression of TMG and NMG gene sets (log2(x+1)-transformed)
- Spearman correlation table (rho and P-values) between gene-set expression and immune score, stromal score, and tumor purity

## How to apply

Download RSEM-normalized RNA-seq expression and clinical data for the TCGA cancer types of interest from the GDC portal and log2(x+1)-transform expression values. Run the ESTIMATE algorithm to compute per-sample immune score, stromal score, and tumor purity. Compute per-sample mean expression of the gene sets of interest (here the 73 TMGs and 186 NMGs identified from single-cell data) and correlate them with each microenvironment metric using Spearman correlation (rho and P-values). Validate the result against expected anchors from the PreCanCell pan-cancer analysis: strong positive NMG-immune score correlations (rho >= 0.89) and strong negative NMG-purity correlations (rho < -0.70). This links single-cell tumor/non-tumor markers to bulk tumor microenvironment properties.

## Related tools

- **ESTIMATE** (Estimates immune score, stromal score, and tumor purity from bulk RNA-seq expression)
- **R** (Environment for log2(x+1) transformation, Spearman correlation computation, and ESTIMATE execution)
- **PreCanCell** (Upstream algorithm that identifies the 73 TMGs and 186 NMGs whose bulk expression is correlated with immune score and purity) — https://github.com/WangX-Lab/PreCanCell
- **ssGSEA** (Gene-set enrichment scoring used alongside ESTIMATE for molecular-feature correlation analyses)

## Examples

```
library(ESTIMATE); est <- estimateScore(expr_matrix_log2cp1, "estimate_out.gct", platform="illumina"); cor(t(mean_expr), est_scores, method="spearman")
```

## Evaluation signals

- Spearman rho between NMG mean expression and immune score is >= 0.89 across TCGA samples
- Spearman rho between NMG mean expression and tumor purity is < -0.70
- Expression values are log2(x+1)-transformed before correlation (matching GDC RSEM normalization handling)
- Correlation table includes both rho and P-values for each of the three metrics (immune score, stromal score, tumor purity)
- Immune score, stromal score, and tumor purity are computed for every sample in the 33 TCGA cancer types

## Limitations

- Correlation anchors (rho >= 0.89 for NMG-immune, rho < -0.70 for NMG-purity) are specific to the 73 TMG/186 NMG sets and TCGA pan-cancer data; other gene sets will yield different expected ranges
- ESTIMATE purity is an expression-based proxy, not pathologist-estimated purity, so it may diverge for highly infiltrated or necrotic tumors

## Evidence

- [methods] ESTIMATE-based microenvironment metrics anchored: [evidence span withheld — anchor status: not_found]
- [methods] data transformation detail: [evidence span withheld — anchor status: not_found]
- [results] validation thresholds: [evidence span withheld — anchor status: not_found]
- [results] gene set origin: "The DEGs included 73 TMGs and 186 NMGs."
- [methods] TCGA data provenance: "we downloaded RNA-seq gene expression profiling (RSEM normalized) and clinical data for 33 TCGA cancer types"
