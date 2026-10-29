---
name: fold-change-threshold-based-differential-expression
description: Use when analyzing human blood transcriptome expression matrices (e.g.
  SummarizedExperiment objects like GSE13015/EH5429) to classify each fixed BloodGen3
  module as increased or decreased per group or per individual sample against a control
  reference.
when_to_use_negative:
- Analysis targets tissues or species other than human blood — the BloodGen3 repertoire
  is specifically designed for human blood transcriptome data
- Input is raw, unnormalized, or log2-transformed expression data without gene-symbol
  row names
- Goal is de novo module/co-expression discovery rather than profiling against the
  fixed repertoire — use WGCNA or GSEA instead
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: BloodGen3module
  role: R package providing Groupcomparison, Groupcomparisonlimma, and Individualcomparison
    functions that apply FC/DIFF/p-value thresholds and express results at the module
    level
  repo: github.com/Drinchai/BloodGen3Module
- name: R
  role: Runtime environment; package installable via devtools::install_github or Bioconductor
    BiocManager
  repo: github.com/Drinchai/BloodGen3Module
- name: BloodGen3
  role: Fixed repertoire of 382 transcriptional modules (built from 16 datasets /
    985 profiles) used to annotate module membership for threshold testing
  repo: github.com/Drinchai/BloodGen3Module
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-24584-w
    title: Development of a fixed module repertoire for the analysis and interpretation
      of blood transcriptome data
  source_content_hash: sha256:d38c49f67d36d2ec358b14d3b3874fd1a96eb1fe435666faec43fb86bd49358e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BloodGen3
  - BloodGen3module
  - R
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# fold-change-threshold-based-differential-expression

## Summary

Threshold-based transcript-level differential expression (fold change ≥ 1.5 plus absolute difference or p-value cutoffs) aggregated to the BloodGen3 module level as the percentage of constitutive genes increased or decreased, using the BloodGen3Module R package's Groupcomparison/Groupcomparisonlimma (group-level) and Individualcomparison (single-sample) functions.

## When to use

Use when analyzing human blood transcriptome expression matrices (e.g. SummarizedExperiment objects like GSE13015/EH5429) to classify each fixed BloodGen3 module as increased or decreased per group or per individual sample against a control reference.

## When NOT to use

- Analysis targets tissues or species other than human blood — the BloodGen3 repertoire is specifically designed for human blood transcriptome data
- Input is raw, unnormalized, or log2-transformed expression data without gene-symbol row names
- Goal is de novo module/co-expression discovery rather than profiling against the fixed repertoire — use WGCNA or GSEA instead

## Inputs

- Normalized expression data matrix (not Log2 transformed) with valid Gene Symbols as row names and Sample IDs as columns, e.g. SummarizedExperiment GSE13015 from ExperimentHub resource EH5429
- Sample annotation dataframe with Sample ID and group columns (e.g. 'Group_test' with values 'Sepsis' and 'Control')
- BloodGen3 module membership annotation (fixed repertoire of 382 modules)

## Outputs

- Group_df / Group_limma dataframe: per-module percentage of genes increased or decreased for group comparisons
- Individual_df dataframe: per-sample, per-module percentage of genes increased or decreased
- Fingerprint grid plot PDF (gridplot) or fingerprint heatmap PDF (fingerprintplot), written to the working directory

## How to apply

Load the normalized (non-log2) expression matrix with valid gene symbols in rows and annotate it with BloodGen3 module membership from the 382-module repertoire. For group comparisons, call Groupcomparison (t-test) or Groupcomparisonlimma with FC = 1.5, pval = 0.1, FDR = TRUE, Group_column = 'Group_test', Test_group = 'Sepsis', Ref_group = 'Control'; for individual-level stratification, call Individualcomparison with FC = 1.5 and DIFF = 10. Both functions compute, per module, the percentage of constitutive genes passing the thresholds relative to the reference group — the two-tiered output is expressed first at the module level and then aggregated (e.g. A28, A35 for neutrophil inflammation). Decision point: use Groupcomparison/limma for cohort contrasts and fingerprint grid plots; use Individualcomparison when inter-individual variability at the single-sample level is the goal, visualized with fingerprintplot at a 15% visualization cutoff. Validate that a module is only called increased or decreased when the required fraction of its constitutive genes pass the thresholds.

## Related tools

- **BloodGen3module** (R package providing Groupcomparison, Groupcomparisonlimma, and Individualcomparison functions that apply FC/DIFF/p-value thresholds and express results at the module level) — github.com/Drinchai/BloodGen3Module
- **R** (Runtime environment; package installable via devtools::install_github or Bioconductor BiocManager) — github.com/Drinchai/BloodGen3Module
- **BloodGen3** (Fixed repertoire of 382 transcriptional modules (built from 16 datasets / 985 profiles) used to annotate module membership for threshold testing) — github.com/Drinchai/BloodGen3Module

## Examples

```
Individual_df = Individualcomparison(GSE13015, sample_info = NULL, FC = 1.5, DIFF = 10, Group_column = "Group_test", Ref_group = "Control")
```

## Evaluation signals

- The resulting dataframe reports, for each module, the percentage of constitutive genes increased and decreased rather than a per-gene statistic
- With FC = 1.5, DIFF = 10, a transcript counts toward a module only if both thresholds relative to Ref_group = 'Control' are met
- gridplot/fingerprintplot PDFs are generated with modules marked increased/decreasing only where ≥ 15% (default cutoff, adjustable 0–100%) of constitutive genes pass thresholds
- Individual-level analysis of the sepsis cohort shows consistent decreases for aggregate A1 (lymphocytic cells/responses) and consistent increases for aggregate A35 (inflammatory neutrophil responses) across patients
- Duplicate handling of the 388 duplicated probes among 14,168 does not materially alter results since they concern only a small minority of genes

## Limitations

- Default individual-analysis cutoffs (FC = 1.5, DIFF = 10) and visualization cutoff (15%) are fixed defaults that can be changed within their allowed ranges (0–100% for visualization); results are threshold-sensitive
- The repertoire was constructed at the probe level from Illumina HT12 v3 BeadArray data, so 388 of 14,168 probes are duplicate genes in different modules and a handling choice is required
- Some aggregates are functionally heterogeneous, so module-level calls are not ground truth; also, the repertoire is specific to human blood and pre-dates RNA-seq

## Evidence

- [readme] The **Groupcomparison** function will perform group comparison analyses.: "The **Groupcomparison** function will perform group comparison analyses."
- [methods] Expressing results at the module level as percentage of genes increased or decreased: "3.	Expressing results “at the module level” as the percentage of genes that are increased or decreased."
- [readme] Individual comparison in reference to control: "The **Individualcomparison** function will perform an individual sample comparison analysis in reference to a control sample or group of samples."
- [readme] Default cutoffs FC=1.5 and DIFF=10 for individual analysis: "The default cut off is set at fold change (FC) =1.5 and absolute difference (DIFF) =10."
- [readme] Default visualization cutoff 15%, changeable 0–100%: "The default cut off for visualization is set at 15%; it can be changed to any value between 0 and 100%."
