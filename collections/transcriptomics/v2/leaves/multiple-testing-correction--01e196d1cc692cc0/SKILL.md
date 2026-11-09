---
name: multiple-testing-correction
description: Use when calling significant hits across thousands of parallel gene-level
  tests (DGE or DUL) in scRNA-seq pseudobulk comparisons, or when testing set overlaps
  (chi-square independence) where unadjusted p-values would inflate false positives.
when_to_use_negative:
- When only a small predefined number of hypotheses is tested (e.g., a single gene
  of interest), where an FDR correction over thousands of tests is unnecessary and
  over-conservative
- When the goal is exploratory ranking of genes by raw effect size rather than defining
  significant hit sets
edam_operation: http://edamontology.org/operation_0566
edam_topics:
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_0203
tools:
- name: scUTRboot
  role: Statistical testing package whose WUI tests produce per-gene p-values that
    are multiple-testing corrected to q < 0.05 for calling differentially used 3'UTR
    isoforms
  repo: https://github.com/mfansler/scutrboot
- name: scUTRquant
  role: Generates the gene-level and 3'UTR isoform count matrices (SingleCellExperiment)
    on which the corrected significance tests are performed
  repo: https://github.com/Mayrlab/scUTRquant
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-48254-9
    title: Quantifying 3′UTR length from scRNA-seq data reveals changes independent
      of gene expression
  source_content_hash: sha256:ef89be1dd31edf358d1cd55c61d92a92d8e1a3f250acf9d7d3f8266aa7de314a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mfansler/scutrboot
  tools:
  - scUTRboot
  - scUTRquant
  edam_operation: http://edamontology.org/operation_0566
  edam_topics:
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# multiple-testing-correction

## Summary

Statistical practice of correcting p-values for many simultaneous hypothesis tests (e.g., thousands of genes tested per cell-type pair), so that significant hits are called at a controlled false-discovery rate rather than by raw p-value cutoffs. In this workflow it is applied to both differential gene expression (Welch t-tests) and differential 3'UTR usage (scUTRboot WUI tests) before downstream overlap analysis.

## When to use

Use when calling significant hits across thousands of parallel gene-level tests (DGE or DUL) in scRNA-seq pseudobulk comparisons, or when testing set overlaps (chi-square independence) where unadjusted p-values would inflate false positives.

## When NOT to use

- When only a small predefined number of hypotheses is tested (e.g., a single gene of interest), where an FDR correction over thousands of tests is unnecessary and over-conservative
- When the goal is exploratory ranking of genes by raw effect size rather than defining significant hit sets

## Inputs

- Per-gene Welch t-test p-values from pseudobulk gene-level count comparisons (scUTRquant SingleCellExperiment gene matrix)
- Per-gene WUI test p-values from scUTRboot on multi-UTR genes
- Chi-square test p-values from DGE-DUL overlap independence tests

## Outputs

- Adjusted p-values (q-values) per gene for DGE and DUL calls
- Filtered hit lists: DGE genes with |fold change| > 1.5 and q < 0.05; DUL genes with |dWUI| > 0.10 and q < 0.05
- Statistically validated overlap fractions (e.g., 0-7.4% range) with chi-square independence results

## How to apply

For each cell-type pair, run Welch t-tests on gene-level counts to call differentially expressed genes and run scUTRboot weighted UTR isoform (WUI) tests on multi-UTR genes to call differentially used 3' ends; in both cases retain hits only after multiple-testing correction at q < 0.05, combined with effect-size filters (|fold change| > 1.5 for DGE, |dWUI| > 0.10 for DUL). The rationale is that with thousands of genes tested per pair, raw p-value thresholds would yield mostly false positives; FDR correction enforces a controlled false-discovery rate so the resulting DGE and DUL gene sets are trustworthy inputs for overlap analysis. Multiple-testing correction is likewise relevant when testing DGE-DUL overlap against chance using chi-square tests of independence across the 17 cell-type pairs.

## Related tools

- **scUTRboot** (Statistical testing package whose WUI tests produce per-gene p-values that are multiple-testing corrected to q < 0.05 for calling differentially used 3'UTR isoforms) — https://github.com/mfansler/scutrboot
- **scUTRquant** (Generates the gene-level and 3'UTR isoform count matrices (SingleCellExperiment) on which the corrected significance tests are performed) — https://github.com/Mayrlab/scUTRquant

## Examples

```
dge_hits <- subset(dge_res, abs(fold_change) > 1.5 & q_value < 0.05)
```

## Evaluation signals

- All retained DGE hits satisfy both |fold change| > 1.5 and q < 0.05 after multiple-testing correction
- All retained DUL hits satisfy both |dWUI| > 0.10 and q < 0.05 after correction
- Per-pair DGE-DUL overlap fractions fall in the expected 0-7.4% range, consistent with statistical independence in 14/17 sample pairs
- Chi-square tests of independence are reported for each pair alongside the overlap fractions in dge_dul_overlap_fractions.tsv

## Limitations

- Applying correction over all genes rather than the restricted multi-UTR, co-expressed gene set changes the FDR denominator and hence the hit calls
- FDR-controlled hit sets are pair-specific; correction does not remove the need for effect-size thresholds (|fold change| > 1.5, |dWUI| > 0.10) to ensure biologically meaningful changes

## Evidence

- [results] DGE and DUL hit thresholds with FDR correction: [evidence span withheld — anchor status: not_found]
- [results] DUL test thresholds: [evidence span withheld — anchor status: not_found]
- [results] chi-square independence testing of overlaps: [evidence span withheld — anchor status: not_found]
- [abstract] expected overlap range supporting the corrected hit sets: "we discover extensive 3'UTR length changes across cell types that are as widespread and coordinately regulated as gene expression changes but affect mostly different genes"
