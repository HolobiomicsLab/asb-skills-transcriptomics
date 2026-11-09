---
name: cancer-cell-line-expression-data-harmonization
description: Use when a bulk cancer cell-line expression matrix (GDSC Affymetrix U219
  microarray, CCLE RSEM RNA-seq) must be log2(x+1)-normalized and aggregated over
  TMG/NMG gene sets for correlation with IC50 drug sensitivity.
when_to_use_negative:
- Input is single-cell transcriptome data that needs malignant vs non-malignant cell
  labels — that is PreCanCell_classifier()'s classification task, not cell-line expression
  harmonization
- You lack the 73 TMG / 186 NMG DEG lists or a per-cell-line IC50 phenotype; the aggregation
  and correlation steps are then undefined
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: R
  role: Analysis environment for log2(x + 1) transformation, per-cell-line TMG/NMG
    aggregation, Spearman correlation against IC50, and Benjamini-Hochberg FDR adjustment
- name: PreCanCell
  role: Source of the 73 TMG and 186 NMG DEG sets (malignant vs non-malignant cells)
    that are aggregated over the cell-line expression matrix
  repo: https://github.com/WangX-Lab/PreCanCell
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.csbj.2023.07.009
    title: 'PreCanCell: An ensemble learning algorithm for predicting cancer and non-cancer
      cells from single-cell transcriptomes'
  source_content_hash: sha256:fb1206fef2cee5d409478a990edeaa5806b030e5f770c8cf10f498a169f8a884
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WangX-Lab/PreCanCell
  tools:
  - PreCanCell
  - R
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# cancer-cell-line-expression-data-harmonization

## Summary

Harmonizes bulk cancer cell-line expression data (GDSC Affymetrix Human Genome U219 microarray for 962 cell lines, CCLE RSEM-normalized RNA-seq) with the single-cell-derived 73 TMG / 186 NMG signatures via log2(x + 1) transformation and per-line gene-set aggregation, enabling Spearman correlation against drug IC50 values.

## When to use

Use when a bulk cancer cell-line expression matrix (GDSC Affymetrix U219 microarray, CCLE RSEM RNA-seq) must be log2(x+1)-normalized and aggregated over TMG/NMG gene sets for correlation with IC50 drug sensitivity.

## When NOT to use

- Input is single-cell transcriptome data that needs malignant vs non-malignant cell labels — that is PreCanCell_classifier()'s classification task, not cell-line expression harmonization
- You lack the 73 TMG / 186 NMG DEG lists or a per-cell-line IC50 phenotype; the aggregation and correlation steps are then undefined

## Inputs

- GDSC Affymetrix Human Genome U219 microarray expression matrix for 962 human cancer cell lines (genes x cell lines)
- GDSC IC50 drug sensitivity values for 265 compounds
- Gene lists of the 73 tumor marker genes (TMGs) and 186 non-tumor marker genes (NMGs), the DEGs between malignant and non-malignant cells

## Outputs

- Log2(x + 1)-transformed cell-line expression matrix
- Per-cell-line aggregate TMG and NMG expression scores for the 962 cell lines
- Per-compound table (265 compounds) of Spearman rho, raw P-value, and Benjamini-Hochberg FDR-adjusted significance for TMG-IC50 and NMG-IC50 correlations

## How to apply

Download the GDSC Affymetrix Human Genome U219 microarray expression matrix for 962 human cancer cell lines together with IC50 values for 265 compounds from the Genomics of Drug Sensitivity in Cancer (GDSC) project. Log2-transform all gene expression values as log2(x + 1) before any downstream computation so microarray (GDSC U219) and RNA-seq (CCLE RSEM-normalized) inputs are comparable. Load the 73 TMGs and 186 NMGs — the DEGs between malignant and non-malignant cells identified across five single-cell datasets (Bonferroni-adjusted Wilcoxon P < 0.05) — and compute a per-cell-line aggregate TMG score and NMG score over the transformed matrix. For each of the 265 compounds, compute the Spearman correlation (rho and P-value) between the TMG aggregate expression and IC50, and separately between the NMG aggregate expression and IC50. Adjust compound-level P-values with the Benjamini-Hochberg FDR method and call significance at FDR < 0.05; the harmonization is correct if direction and coverage match the published result (TMG expression significantly positively correlated with IC50 in 212/265 compounds; NMG significantly negatively in 232/265).

## Related tools

- **R** (Analysis environment for log2(x + 1) transformation, per-cell-line TMG/NMG aggregation, Spearman correlation against IC50, and Benjamini-Hochberg FDR adjustment)
- **PreCanCell** (Source of the 73 TMG and 186 NMG DEG sets (malignant vs non-malignant cells) that are aggregated over the cell-line expression matrix) — https://github.com/WangX-Lab/PreCanCell

## Examples

```
expr <- log2(read.table("gdsc_u219_expression.txt", sep = "\t", header = TRUE, row.names = 1) + 1)
```

## Evaluation signals

- Significance coverage reproduces the published pattern: TMG aggregate expression significantly positively correlates with IC50 in 212 of 265 compounds and NMG significantly negatively in 232 of 265 (FDR < 0.05)
- Transformed matrix satisfies the log2(x + 1) invariant: no negative values and zero expression maps to zero
- Aggregate scores are computed for all 962 GDSC cell lines over exactly 73 TMGs and 186 NMGs
- Benjamini-Hochberg FDR-adjusted P-values across the 265 compounds are monotone with raw P-values and flagged at FDR < 0.05
- Correlation directions skew as reported: TMG-IC50 rho predominantly positive, NMG-IC50 rho predominantly negative

## Limitations

- The 73 TMG / 186 NMG signatures derive from five cancer single-cell datasets (RCC, HNSCC, melanoma, LUAD, BC); transfer to cell lines of other lineages is untested
- GDSC expression is Affymetrix U219 microarray while CCLE/TCGA are RSEM-normalized RNA-seq, so cross-platform use relies on the log2(x + 1) transform and matched-gene selection; residual platform effects are not corrected
- PreCanCell's reported training-set performance may be overestimated because the predicted sets were involved in feature selection, so the DEG signatures may be overfitted to those five cancer types

## Evidence

- [methods] their drug sensitivities (IC50 values) to 265 compounds from the Genomics of Drug Sensitivity in Cancer (GDSC) project: "their drug sensitivities (IC50 values) to 265 compounds from the Genomics of Drug Sensitivity in Cancer (GDSC) project"
- [methods] RNA-seq gene expression profiling (RSEM normalized) in 1033 cancer cell lines and 51 normal cell lines from the Cancer Cell Line Encyclopedia (CCLE) project: "RNA-seq gene expression profiling (RSEM normalized) in 1033 cancer cell lines and 51 normal cell lines from the Cancer Cell Line Encyclopedia (CCLE) project"
- [results] The DEGs included 73 TMGs and 186 NMGs.: "The DEGs included 73 TMGs and 186 NMGs."
- [other] Log2-transform all gene expression values as log2(x + 1) before analysis.: [evidence span withheld — anchor status: not_found]
- [other] In cancer cell lines, TMG expression correlated significantly positively with IC50 values in 212 (80%) of 265 compounds: [evidence span withheld — anchor status: not_found]
- [readme] The input data can be any library-depth normalization (e.g. TPM, CPM).: "The input data can be any library-depth normalization (e.g. TPM, CPM)."
