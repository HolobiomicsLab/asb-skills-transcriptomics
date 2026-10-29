---
name: gene-signature-aggregate-scoring
description: Use when you have a gene set (TMGs/NMGs or other DEG-derived signature)
  and a matched expression-plus-phenotype matrix (e.g., 962 GDSC cancer cell lines
  x 265 compound IC50 values) and need a single signature-level score per sample to
  correlate with the phenotype.
when_to_use_negative:
- The task is classifying individual single cells as malignant vs non-malignant —
  that requires PreCanCell_classifier(), not signature aggregate scoring
- Gene-level (per-gene) correlation output is required rather than a single signature-level
  score per sample
- The expression data are single-cell and unlabeled for cell type — PreCanCell_data()/PreCanCell_classifier()
  is the intended pipeline, not direct signature scoring
edam_operation: http://edamontology.org/operation_3465
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: R
  role: Environment used to compute per-cell-line TMG/NMG aggregate scores, per-compound
    Spearman correlations with IC50, and Benjamini-Hochberg FDR adjustment
- name: PreCanCell
  role: Source of the TMG/NMG signatures (DEGs between malignant and non-malignant
    cells identified across five cancer single-cell transcriptome training datasets)
  repo: https://github.com/WangX-Lab/PreCanCell
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.csbj.2023.07.009
    title: 'PreCanCell: An ensemble learning algorithm for predicting cancer and non-cancer
      cells from single-cell transcriptomes'
  source_content_hash: sha256:45cd6936780c2474946892b1451756acedae3aff541dd90e1142081863780022
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
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# gene-signature-aggregate-scoring

## Summary

Computes per-sample aggregate expression scores for curated gene signatures (e.g., 73 tumor marker genes and 186 non-tumor marker genes derived as DEGs between malignant and non-malignant cells) and tests them against continuous phenotypes such as GDSC drug IC50 values via Spearman correlation with Benjamini-Hochberg FDR correction.

## When to use

Use when you have a gene set (TMGs/NMGs or other DEG-derived signature) and a matched expression-plus-phenotype matrix (e.g., 962 GDSC cancer cell lines x 265 compound IC50 values) and need a single signature-level score per sample to correlate with the phenotype.

## When NOT to use

- The task is classifying individual single cells as malignant vs non-malignant — that requires PreCanCell_classifier(), not signature aggregate scoring
- Gene-level (per-gene) correlation output is required rather than a single signature-level score per sample
- The expression data are single-cell and unlabeled for cell type — PreCanCell_data()/PreCanCell_classifier() is the intended pipeline, not direct signature scoring

## Inputs

- GDSC Affymetrix Human Genome U219 microarray expression matrix for 962 human cancer cell lines
- GDSC IC50 values for 265 compounds per cell line
- Gene signature lists: 73 tumor marker genes (TMGs) and 186 non-tumor marker genes (NMGs), the DEGs between malignant and non-malignant cells

## Outputs

- Per-cell-line aggregate TMG expression score and aggregate NMG expression score
- Per-compound table of Spearman rho, raw P-value, and BH-adjusted FDR for TMG-IC50 and NMG-IC50 correlations

## How to apply

Load the curated signature gene lists — the DEGs between malignant and non-malignant cells, comprising 73 TMGs and 186 NMGs. Download the Affymetrix Human Genome U219 microarray expression data for 962 human cancer cell lines and their IC50 values for 265 compounds from GDSC, then log2-transform expression values as log2(x + 1). Compute a per-cell-line aggregate score for the TMG set and separately for the NMG set. For each of the 265 compounds, compute the Spearman correlation (rho and P-value) between the TMG aggregate score and IC50, and separately between the NMG aggregate score and IC50. Adjust P-values across compounds using Benjamini-Hochberg FDR; the rationale is to collapse many correlated marker genes into one robust per-sample statistic so the signature-level drug-sensitivity association can be tested consistently across all compounds.

## Related tools

- **R** (Environment used to compute per-cell-line TMG/NMG aggregate scores, per-compound Spearman correlations with IC50, and Benjamini-Hochberg FDR adjustment)
- **PreCanCell** (Source of the TMG/NMG signatures (DEGs between malignant and non-malignant cells identified across five cancer single-cell transcriptome training datasets)) — https://github.com/WangX-Lab/PreCanCell

## Examples

```
rho <- cor.test(log2(tmg_score + 1), log2(ic50 + 1), method="spearman"); p_adj <- p.adjust(pvals, method="BH")
```

## Evaluation signals

- Direction check: TMG aggregate expression should correlate significantly positively with IC50 in 212 of 265 compounds (80%), and NMG aggregate expression significantly negatively in 232 of 265 compounds (88%), at FDR < 0.05
- Coverage check: correlations computed for all 265 GDSC compounds and all 962 cell lines with non-missing IC50
- Preprocessing invariant: expression values are log2-transformed as log2(x + 1) before scoring
- Multiple-testing check: reported P-values are Benjamini-Hochberg adjusted across the 265 compounds

## Limitations

- Aggregate scoring masks individual gene contributions; the correlation is a signature-level association, not evidence for any single TMG or NMG driving IC50
- Spearman correlation with IC50 does not establish causality between TMG/NMG expression and drug resistance
- The GDSC microarray platform (Affymetrix Human Genome U219) differs from the single-cell RNA-seq data used to define the TMG/NMG DEGs, so cross-platform scaling of the aggregate score should be checked

## Evidence

- [results] The DEGs included 73 TMGs and 186 NMGs.: "The DEGs included 73 TMGs and 186 NMGs."
- [methods] GDSC dataset and IC50 values: "their drug sensitivities (IC50 values) to 265 compounds from the Genomics of Drug Sensitivity in Cancer (GDSC) project"
- [methods] TMG definition as signature: "this gene set was defined as tumor marker genes"
- [intro] Correlation with cancer features objective: "We also analyzed correlations of these DEGs with various cancer-related molecular and clinical features in pan-cancer"
