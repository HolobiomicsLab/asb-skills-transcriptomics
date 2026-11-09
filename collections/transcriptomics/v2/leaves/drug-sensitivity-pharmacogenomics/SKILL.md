---
name: drug-sensitivity-pharmacogenomics
description: Use when testing whether a gene-expression signature (e.g., TMG/NMG aggregate
  scores) is associated with drug response (IC50) across a cell-line panel, requiring
  per-compound Spearman correlations with FDR control.
when_to_use_negative:
- The goal is classifying individual cells as malignant vs non-malignant — that is
  PreCanCell's k-NN (k=5) ensemble classifier, not this pharmacogenomic correlation
  workflow.
- Expression and drug response are not matched per sample (no paired cell-line IC50
  data), so per-compound Spearman correlation of aggregate scores is undefined.
- The target is differential expression between malignant and non-malignant cells
  (FindMarkers/Wilcoxon, Bonferroni-corrected P < 0.05) rather than drug sensitivity
  association.
edam_operation: http://edamontology.org/operation_3465
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: R
  role: Performs log2(x+1) transformation, per-compound Spearman correlation of aggregate
    TMG/NMG scores with IC50, and Benjamini-Hochberg FDR adjustment
- name: PreCanCell
  role: Source of the 73 TMG / 186 NMG differentially expressed gene lists used as
    the correlation features
  repo: https://github.com/WangX-Lab/PreCanCell
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.csbj.2023.07.009
    title: 'PreCanCell: An ensemble learning algorithm for predicting cancer and non-cancer
      cells from single-cell transcriptomes'
  source_content_hash: sha256:a3143772056ef63a590664363cca57e8d8139bda96b7ee709aeb90bb1a6558ef
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WangX-Lab/PreCanCell
  tools:
  - PreCanCell
  - R
  edam_operation: http://edamontology.org/operation_3465
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# drug-sensitivity-pharmacogenomics

## Summary

Correlates aggregate expression of tumor marker genes (TMGs) and non-tumor marker genes (NMGs) with drug sensitivity (IC50) across cancer cell lines to test whether malignant-cell signatures predict pharmacogenomic response.

## When to use

Use when testing whether a gene-expression signature (e.g., TMG/NMG aggregate scores) is associated with drug response (IC50) across a cell-line panel, requiring per-compound Spearman correlations with FDR control.

## When NOT to use

- The goal is classifying individual cells as malignant vs non-malignant — that is PreCanCell's k-NN (k=5) ensemble classifier, not this pharmacogenomic correlation workflow.
- Expression and drug response are not matched per sample (no paired cell-line IC50 data), so per-compound Spearman correlation of aggregate scores is undefined.
- The target is differential expression between malignant and non-malignant cells (FindMarkers/Wilcoxon, Bonferroni-corrected P < 0.05) rather than drug sensitivity association.

## Inputs

- GDSC Affymetrix Human Genome U219 gene expression matrix for 962 human cancer cell lines
- GDSC drug sensitivity matrix of IC50 values (962 cell lines x 265 compounds)
- Gene lists of 73 TMGs and 186 NMGs (DEGs between malignant and non-malignant cells)

## Outputs

- Per-compound table of Spearman rho, raw P-value, Benjamini-Hochberg FDR-adjusted P-value, and significance flags for TMG-IC50 and NMG-IC50 correlations

## How to apply

Download the Affymetrix Human Genome U219 expression data for 962 human cancer cell lines and their IC50 values for 265 compounds from the Genomics of Drug Sensitivity in Cancer (GDSC) project. Log2-transform all gene expression values as log2(x + 1) before analysis. Compute per-cell-line aggregate scores for the 73 TMGs and 186 NMGs — the DEGs identified between malignant and non-malignant cells. For each of the 265 compounds, compute the Spearman correlation (rho and P-value) between the TMG aggregate score and IC50, and separately between the NMG aggregate score and IC50. Adjust P-values across compounds with the Benjamini-Hochberg FDR method and call correlations significant at FDR < 0.05. Validate by checking that the direction and coverage of significant correlations match the reported result: TMG expression significantly positively correlated with IC50 in 212 of 265 compounds, and NMG expression significantly negatively correlated in 232 of 265 compounds.

## Related tools

- **R** (Performs log2(x+1) transformation, per-compound Spearman correlation of aggregate TMG/NMG scores with IC50, and Benjamini-Hochberg FDR adjustment)
- **PreCanCell** (Source of the 73 TMG / 186 NMG differentially expressed gene lists used as the correlation features) — https://github.com/WangX-Lab/PreCanCell

## Examples

```
sapply(compounds, function(cp) cor.test(tmg_score, ic50[[cp]], method = "spearman")$estimate); fdr <- p.adjust(pvals, method = "BH")
```

## Evaluation signals

- TMG aggregate expression is significantly positively correlated with IC50 (FDR < 0.05) in 212 of 265 compounds
- NMG aggregate expression is significantly negatively correlated with IC50 (FDR < 0.05) in 232 of 265 compounds
- All expression values are log2(x + 1) transformed before aggregation and correlation
- P-values are Benjamini-Hochberg adjusted across the 265 compounds, and the output table contains rho, P-value, and significance per compound
- Aggregate scores are computed from exactly the 73 TMG and 186 NMG gene lists

## Limitations

- Correlation with IC50 does not establish a causal effect of TMG/NMG expression on drug response
- Analysis is restricted to the 265 GDSC compounds and 962 cell lines profiled on the Affymetrix Human Genome U219 microarray; direction and coverage of significant correlations may differ on other platforms or cohorts
- Aggregate TMG/NMG scores summarize bulk cell-line expression and do not resolve per-cell-type composition within samples

## Evidence

- [methods] their drug sensitivities (IC50 values) to 265 compounds from the Genomics of Drug Sensitivity in Cancer (GDSC) project: "their drug sensitivities (IC50 values) to 265 compounds from the Genomics of Drug Sensitivity in Cancer (GDSC) project"
- [results] The DEGs included 73 TMGs and 186 NMGs.: "The DEGs included 73 TMGs and 186 NMGs."
- [other] compute the Spearman correlation (rho and P-value) between TMG aggregate expression and IC50, and separately between NMG aggregate expression and IC50: [evidence span withheld — anchor status: not_found]
