---
name: fdr-multiple-testing-correction
description: Use when performing thousands of per-gene group comparisons (e.g. t-tests
  across all probes in a blood expression matrix) where module-level hits would otherwise
  be inflated, and a BH-FDR-adjusted p-value cutoff must be applied instead of raw
  p-values.
when_to_use_negative:
- Input analysis uses the Individualcomparison single-sample function, which uses
  fold change (FC = 1.5) and absolute difference (DIFF = 10) cutoffs and does not
  involve group-comparison p-values to FDR-correct
- The expression matrix is log2 transformed — the BloodGen3module functions expect
  non-log2 normalized data and transform internally
- Goal is gene-level differential expression reporting rather than module-level percentage
  summaries; FDR here is a parameter of the module repertoire workflow, not a standalone
  gene-level testing tool
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3053
tools:
- name: BloodGen3module (R package)
  role: Provides the Groupcomparison (t-test) and Groupcomparisonlimma functions whose
    FDR argument applies Benjamini-Hochberg correction to p-values before module-level
    aggregation
  repo: github.com/Drinchai/BloodGen3Module
- name: R
  role: Runtime environment in which the BloodGen3module workflows for group comparison
    and molecular fingerprinting are executed
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-24584-w
    title: Development of a fixed module repertoire for the analysis and interpretation
      of blood transcriptome data
  source_content_hash: sha256:b943d55ebccfd49da4b4409a2d29ac7786e78c0b950ad5e7a32c2619ce9ab02e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BloodGen3
  - BloodGen3module (R package)
  - R
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3053
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# fdr-multiple-testing-correction

## Summary

Apply Benjamini-Hochberg false discovery rate correction to p-values when computing module-level differential expression in blood transcriptome group comparisons, as implemented in the BloodGen3module Groupcomparison function (FDR = TRUE). This ensures that the percentage of genes increased or decreased per module is not inflated by spurious per-transcript hits across thousands of tested probes.

## When to use

Use when performing thousands of per-gene group comparisons (e.g. t-tests across all probes in a blood expression matrix) where module-level hits would otherwise be inflated, and a BH-FDR-adjusted p-value cutoff must be applied instead of raw p-values.

## When NOT to use

- Input analysis uses the Individualcomparison single-sample function, which uses fold change (FC = 1.5) and absolute difference (DIFF = 10) cutoffs and does not involve group-comparison p-values to FDR-correct
- The expression matrix is log2 transformed — the BloodGen3module functions expect non-log2 normalized data and transform internally
- Goal is gene-level differential expression reporting rather than module-level percentage summaries; FDR here is a parameter of the module repertoire workflow, not a standalone gene-level testing tool

## Inputs

- Normalized expression data matrix (genes as rows, Sample ID as columns, valid Gene Symbols as row names, NOT log2 transformed)
- Sample annotation dataframe with a Sample ID column and a group annotation column (e.g. Group_test with values SLE and Control)

## Outputs

- Group_df dataframe from Groupcomparison containing per-module percentages of increased/decreased genes computed against the FDR-adjusted p-value threshold
- Fingerprint grid plot (PDF from gridplot) whose shaded modules reflect FDR-filtered results at a 15% visualization cutoff

## How to apply

Run the BloodGen3module Groupcomparison function on a normalized (non-log2-transformed) expression matrix with sample annotation, using a t-test, fold-change cutoff FC = 1.5, and set FDR = TRUE so that p-values are corrected by the Benjamini-Hochberg method before comparison against the cutoff pval = 0.1. The function tests each transcript for increase/decrease relative to the Ref_group (e.g. Control) versus the Test_group (e.g. SLE), and FDR correction is applied because thousands of probes are tested simultaneously. Results are then aggregated 'at the module level' as the percentage of constitutive genes increased or decreased per module, which feeds the gridplot visualization at a 15% visualization cutoff. The same FDR option is available for the limma variant (Groupcomparisonlimma) when a linear-model framework is preferred over the t-test.

## Related tools

- **BloodGen3module (R package)** (Provides the Groupcomparison (t-test) and Groupcomparisonlimma functions whose FDR argument applies Benjamini-Hochberg correction to p-values before module-level aggregation) — github.com/Drinchai/BloodGen3Module
- **R** (Runtime environment in which the BloodGen3module workflows for group comparison and molecular fingerprinting are executed)

## Examples

```
Group_df <- Groupcomparison(GSE13015, sample_info = NULL, FC = 1.5, pval = 0.1, FDR = TRUE, Group_column = "Group_test", Test_group = "SLE", Ref_group = "Control")
```

## Evaluation signals

- The Groupcomparison call returns without error and includes FDR = TRUE with pval = 0.1 and FC = 1.5
- Output Group_df expresses results at the module level as percentages of genes increased or decreased per module (aggregates A1–A38)
- Fingerprint grid PDF exists in the working directory and only modules meeting or exceeding the 15% visualization cutoff are shaded
- Rows of the grid correspond to the fixed BloodGen3 module aggregates, confirming module membership annotation was applied before the FDR-filtered comparison
- Rerunning with FDR = FALSE at the same pval yields a superset of module hits, confirming the correction is actually restricting differential calls

## Limitations

- FDR correction is applied at the transcript level within the module repertoire framework; 388 of 14,168 probes are duplicate genes found in different modules, a small minority whose handling has little impact on results
- The default pval = 0.1 with FDR = TRUE is relatively permissive even after correction; results should be interpreted as module-level enrichment signals rather than individually validated gene calls
- The repertoire was built at the probe level on Illumina HT12 v3 BeadArrays (pre-dating RNA-seq), so FDR-filtered results on other platforms depend on gene-symbol mapping quality

## Evidence

- [readme] pval             Numeric value specifying the p-value cut off or False discovery rate when FDR = TRUE: [evidence span withheld — anchor status: external_doc]
- [readme] FDR              Logical operator (TRUE/FALSE) to specify whether False discovery rate cut off (using BH-method) should be used.: [evidence span withheld — anchor status: external_doc]
- [methods] 2.	Running statistical tests to determine the proportion of constitutive genes that are differentially expressed for each module.: "2.	Running statistical tests to determine the proportion of constitutive genes that are differentially expressed for each module."
- [methods] 3.	Expressing results “at the module level” as the percentage of genes that are increased or decreased.: "3.	Expressing results “at the module level” as the percentage of genes that are increased or decreased."
- [readme] pval = 0.1 ,
                            FDR = TRUE,: "pval = 0.1 ,
                            FDR = TRUE,"
