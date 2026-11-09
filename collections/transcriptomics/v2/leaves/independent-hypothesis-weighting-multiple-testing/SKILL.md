---
name: independent-hypothesis-weighting-multiple-testing
description: Use when you have a large family of test p-values (e.g., ~20k genes from
  edgeR or 50 camera gene-set tests) plus an independent continuous covariate like
  average expression, and you need FDR-controlled calls (FDR < 0.1 or < 0.05) with
  maximal power.
when_to_use_negative:
- No independent covariate is available for the tested hypotheses (IHW's power gain
  depends on an informative, null-independent covariate such as average gene expression)
- The p-value family is small or has a simple flat structure where plain Benjamini–Hochberg
  FDR control is sufficient
- The covariate is not independent of the test statistic under the null hypothesis
  (e.g., it is derived from the same counts driving the p-values directly)
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: IHW
  role: Performs covariate-weighted multiple-testing correction of edgeR gene-level
    p-values and camera gene-set p-values
- name: edgeR
  role: Produces the two-sided quasi-likelihood F-test p-values and log2-fold-changes
    that IHW corrects
- name: limma
  role: camera two-sided test on 50 MSigDB Hallmark gene sets; its gene-set p-values
    are then IHW-adjusted
- name: Molecular Signatures Database (MsigDB)
  role: Source of the 50 Hallmark gene sets tested by camera prior to IHW adjustment
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-022-28845-0
    title: MQuad enables clonal substructure discovery using single cell mitochondrial
      variants
  source_content_hash: sha256:30afab5e09ea8adce15254ae3408d645f8a453420d97a37f8a27ad6f30a587d2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - IHW
  - Molecular Signatures Database (MsigDB)
  - edgeR
  - limma
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 8
---

# independent-hypothesis-weighting-multiple-testing

## Summary

Applies Independent Hypothesis Weighting (IHW) to control FDR with improved power by exploiting a continuous independent covariate (here, average gene expression) when correcting gene-level and gene-set p-values from single-cell clonal differential expression analyses.

## When to use

Use when you have a large family of test p-values (e.g., ~20k genes from edgeR or 50 camera gene-set tests) plus an independent continuous covariate like average expression, and you need FDR-controlled calls (FDR < 0.1 or < 0.05) with maximal power.

## When NOT to use

- No independent covariate is available for the tested hypotheses (IHW's power gain depends on an informative, null-independent covariate such as average gene expression)
- The p-value family is small or has a simple flat structure where plain Benjamini–Hochberg FDR control is sufficient
- The covariate is not independent of the test statistic under the null hypothesis (e.g., it is derived from the same counts driving the p-values directly)

## Inputs

- Vector of two-sided p-values from an edgeR quasi-likelihood F-test (gene-level DE, e.g., de_gene_table.csv)
- Vector of average gene expression values used as the independent IHW covariate
- Vector of gene-set p-values from the limma camera test on 50 MSigDB Hallmark gene sets (gene_set_enrichment_table.csv)
- edgeR log2-fold-changes used as camera ranking statistic

## Outputs

- IHW-adjusted FDR values per gene; significant DE gene set at FDR < 0.1 (847 genes reported for clone1.MT0 vs clone1.MT1)
- IHW-adjusted FDR values per Hallmark gene set; significant sets flagged at FDR < 0.05 (MYC targets, E2F targets, G2M checkpoint, ROS Pathway)
- de_gene_table.csv with a gene-level statistics column containing FDR values
- gene_set_enrichment_table.csv with significance flags

## How to apply

After fitting an edgeR GLM on Smart-seq2 single-cell counts with cellular detection rate, plate, and mtDNA clone assignment as predictors and running a two-sided quasi-likelihood F-test (e.g., clone1.MT0 vs clone1.MT1), collect the two-sided p-values and use each gene's average expression as the independent covariate for IHW, retaining DE genes at FDR < 0.1 (the paper reports 847 DE genes). For gene-set analysis, run the two-sided camera test from limma on 50 MSigDB Hallmark gene sets using the edgeR log2-fold-changes as input, then adjust gene-set p-values with IHW the same way and flag enrichments at FDR < 0.05 (expected hits: MYC targets, E2F targets, G2M checkpoint, ROS Pathway). The rationale is that weighting hypotheses by a covariate informative of significance, while independent of the p-values under the null, increases rejections relative to plain Benjamini–Hochberg at the same FDR level.

## Related tools

- **IHW** (Performs covariate-weighted multiple-testing correction of edgeR gene-level p-values and camera gene-set p-values)
- **edgeR** (Produces the two-sided quasi-likelihood F-test p-values and log2-fold-changes that IHW corrects)
- **limma** (camera two-sided test on 50 MSigDB Hallmark gene sets; its gene-set p-values are then IHW-adjusted)
- **Molecular Signatures Database (MsigDB)** (Source of the 50 Hallmark gene sets tested by camera prior to IHW adjustment)

## Examples

```
library(IHW); ihw_res <- ihw(p_value, covariate = average_expression, alpha = 0.1); adj_fdr <- adj_pvalues(ihw_res)
```

## Evaluation signals

- de_gene_table.csv contains a gene-level statistics column with FDR values, and the count of significant DE genes at FDR < 0.1 is consistent with the paper's 847 genes
- gene_set_enrichment_table.csv flags exactly the four expected Hallmark sets (MYC targets, E2F targets, G2M checkpoint, ROS Pathway) as significant at FDR < 0.05
- Adjusted FDR values are monotone with p-values within covariate strata and bounded by the nominal alpha (0.1 gene-level, 0.05 gene-set level)
- The IHW covariate (average gene expression) is confirmed independent of p-values under the null before weighting

## Limitations

- Power gain depends on the informativeness of the independent covariate; with a weak covariate, IHW reduces to standard FDR correction
- Results depend on the upstream edgeR model (cellular detection rate, plate, mtDNA clone as predictors); mis-specification of predictors propagates into the corrected calls
- The reported significance counts (847 DE genes; four Hallmark sets) are specific to the fibroblast Smart-seq2 dataset with clone1.MT0 vs clone1.MT1 and may not generalize to other subclone comparisons

## Evidence

- [methods] IHW correction of DE p-values using expression covariate: [evidence span withheld — anchor status: not_found]
- [methods] IHW correction of camera gene-set p-values: [evidence span withheld — anchor status: not_found]
- [discussion] camera test on Hallmark sets: "Using 50 Hallmark gene sets from Molecular Signatures Database (MsigDB)43"
