---
name: spearman-correlation-analysis
description: Use when testing whether a gene-expression signature (e.g., aggregate
  TMG or NMG scores from the 73/186 DEGs) is associated with a continuous drug-response
  phenotype such as GDSC IC50 across cancer cell lines.
when_to_use_negative:
- The goal is classifying individual cells as malignant vs non-malignant — that is
  PreCanCell_classifier()'s k-NN (k = 5) ensemble task, not phenotype correlation.
- The endpoint is time-to-event survival rather than a continuous drug-response metric
  — the article handles survival with survfit() from the R survival package instead.
- 'The gene lists still need to be derived: differential expression between malignant
  and non-malignant cells uses Seurat FindMarkers() with Bonferroni-adjusted Wilcoxon
  P < 0.05, upstream of this skill.'
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0202
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3308
tools:
- name: R
  role: Runtime for the log2(x + 1) transformation, per-cell-line TMG/NMG score aggregation,
    per-compound Spearman correlation, and Benjamini-Hochberg FDR adjustment
- name: PreCanCell
  role: Source of the 73 TMG and 186 NMG gene lists (common DEGs between malignant
    and non-malignant cells) whose aggregate expression scores are correlated with
    IC50
  repo: https://github.com/WangX-Lab/PreCanCell
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.csbj.2023.07.009
    title: 'PreCanCell: An ensemble learning algorithm for predicting cancer and non-cancer
      cells from single-cell transcriptomes'
  source_content_hash: sha256:af11a6e1294d986ef5f3018e47bf45a2d62e16e56a15bbb92dcdbd784bb5a699
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WangX-Lab/PreCanCell
  tools:
  - PreCanCell
  - R
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0202
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# spearman-correlation-analysis

## Summary

Computes per-compound Spearman rank correlations between aggregate tumor marker gene (TMG) and non-tumor marker gene (NMG) expression scores and GDSC IC50 values across 962 cancer cell lines, testing whether the 73 TMG / 186 NMG malignant-vs-non-malignant DEG signatures track drug sensitivity or resistance.

## When to use

Use when testing whether a gene-expression signature (e.g., aggregate TMG or NMG scores from the 73/186 DEGs) is associated with a continuous drug-response phenotype such as GDSC IC50 across cancer cell lines.

## When NOT to use

- The goal is classifying individual cells as malignant vs non-malignant — that is PreCanCell_classifier()'s k-NN (k = 5) ensemble task, not phenotype correlation.
- The endpoint is time-to-event survival rather than a continuous drug-response metric — the article handles survival with survfit() from the R survival package instead.
- The gene lists still need to be derived: differential expression between malignant and non-malignant cells uses Seurat FindMarkers() with Bonferroni-adjusted Wilcoxon P < 0.05, upstream of this skill.

## Inputs

- Affymetrix Human Genome U219 microarray expression matrix for 962 human cancer cell lines (from the GDSC project)
- GDSC IC50 drug-sensitivity values for 265 compounds across the same cell lines
- Gene lists: 73 TMGs and 186 NMGs (common DEGs between malignant and non-malignant cells)

## Outputs

- Per-compound table (265 rows) of Spearman rho, raw P-value, and BH-adjusted FDR for TMG aggregate expression vs IC50
- Per-compound table (265 rows) of Spearman rho, raw P-value, and BH-adjusted FDR for NMG aggregate expression vs IC50
- Per-cell-line aggregate TMG and NMG expression scores computed on log2(x + 1)-transformed values

## How to apply

Download the Affymetrix Human Genome U219 microarray expression data for 962 human cancer cell lines plus IC50 values for 265 compounds from the GDSC project, and log2-transform expression as log2(x + 1) before any analysis. Aggregate the 73 TMGs and 186 NMGs (the DEGs between malignant and non-malignant cells from the PreCanCell study) into per-cell-line TMG and NMG expression scores. For each of the 265 compounds, compute the Spearman rho and P-value between the TMG score and IC50 and, separately, between the NMG score and IC50; the rank-based statistic is robust to the skewed IC50 distributions typical of drug screens. Adjust per-compound P-values with the Benjamini-Hochberg FDR method and call significance at FDR < 0.05, tabulating per-compound rho, raw P-value, adjusted P-value, and direction. Validate by checking that TMG expression is significantly positively correlated with IC50 in ~212 of 265 compounds (80%) and NMG expression is significantly negatively correlated in ~232 of 265 compounds (88%).

## Related tools

- **R** (Runtime for the log2(x + 1) transformation, per-cell-line TMG/NMG score aggregation, per-compound Spearman correlation, and Benjamini-Hochberg FDR adjustment)
- **PreCanCell** (Source of the 73 TMG and 186 NMG gene lists (common DEGs between malignant and non-malignant cells) whose aggregate expression scores are correlated with IC50) — https://github.com/WangX-Lab/PreCanCell

## Examples

```
ct <- cor.test(tmg_score, ic50, method = "spearman", exact = FALSE); padj <- p.adjust(pvals, method = "BH")
```

## Evaluation signals

- Directional coverage reproduces the report: TMG score significantly positively correlated with IC50 in ~212 of 265 compounds and NMG score significantly negatively correlated in ~232 of 265 compounds at FDR < 0.05
- Each correlation table contains exactly 265 rows (one per GDSC compound), each with a rho, a raw P-value, and a BH-adjusted FDR
- All expression values are log2(x + 1)-transformed before aggregation, and TMG/NMG scores are computed for all 962 cell lines with no missing compounds
- Sign pattern is coherent: FDR-significant rho values are predominantly positive for TMG and predominantly negative for NMG
- BH-adjusted P-values are monotonically consistent with raw P-values, and significance calls use the FDR < 0.05 threshold

## Limitations

- The 73 TMG / 186 NMG lists come from DEG selection in the same five scRNA-seq datasets used as PreCanCell training sets, so the gene lists may embed feature-selection bias
- Correlations use bulk Affymetrix U219 microarray data while the gene sets derive from single-cell transcriptomes; gene-identifier matching across platforms can reduce effective gene-set coverage
- Spearman rho detects monotonic association only, and the significance counts are specific to the 265 GDSC compounds — they do not automatically transfer to other drug panels or in vivo response

## Evidence

- [intro] We also analyzed correlations of these DEGs with various cancer-related molecular and clinical features in pan-cancer: "We also analyzed correlations of these DEGs with various cancer-related molecular and clinical features in pan-cancer"
- [methods] their drug sensitivities (IC50 values) to 265 compounds from the Genomics of Drug Sensitivity in Cancer (GDSC) project: "their drug sensitivities (IC50 values) to 265 compounds from the Genomics of Drug Sensitivity in Cancer (GDSC) project"
- [results] The DEGs included 73 TMGs and 186 NMGs.: "The DEGs included 73 TMGs and 186 NMGs."
- [other] For each of the 265 compounds, compute the Spearman correlation (rho and P-value) between TMG aggregate expression and IC50: [evidence span withheld — anchor status: not_found]
- [other] TMG expression correlated significantly positively with IC50 values in 212 (80%) of 265 compounds: [evidence span withheld — anchor status: not_found]
