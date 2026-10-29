---
name: module-percentage-heatmap-visualization
description: Use when the goal is to assess inter-individual variability in a blood
  transcriptome dataset at the module level for each individual subject — e.g., stratifying
  patients by individual molecular fingerprints rather than comparing pooled groups.
when_to_use_negative:
- The analysis goal is a pooled group-vs-group comparison rather than individual patient
  stratification — use Groupcomparison/Groupcomparisonlimma and gridplot instead
- The input expression matrix is Log2 transformed or lacks valid Gene Symbols as row
  names, violating the Individualcomparison input requirements
- The data are not human blood transcriptome profiles — the BloodGen3 repertoire is
  specifically designed for human blood and other tissues or species require separate
  frameworks
edam_operation: http://edamontology.org/operation_0531
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0203
tools:
- name: BloodGen3module
  role: R package supplying the Individualcomparison and fingerprintplot functions
    that compute per-sample module percentages and render the fingerprint heatmap
    PDF
  repo: github.com/Drinchai/BloodGen3Module
- name: R
  role: Runtime environment in which the BloodGen3module analysis workflow (ExperimentHub
    data loading through heatmap visualization) is executed
- name: ExperimentHub
  role: Source of the example GSE13015 SummarizedExperiment resource (EH5429) used
    as expression input
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-021-24584-w
    title: Development of a fixed module repertoire for the analysis and interpretation
      of blood transcriptome data
  source_content_hash: sha256:62ef1866def9e9756faf81ac5ab9f29393c68655b1108fd698b1970a82ff9c8e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BloodGen3
  - BloodGen3module
  - ExperimentHub
  - R
  edam_operation: http://edamontology.org/operation_0531
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# module-percentage-heatmap-visualization

## Summary

Performs an individual sample comparison against a reference group using the BloodGen3Module package, expressing results 'at the module level' as the percentage of constitutive genes increased or decreased per sample, and renders them as a fingerprint heatmap PDF. This enables molecular stratification of individual patients within a cohort rather than pooled group averages.

## When to use

Use when the goal is to assess inter-individual variability in a blood transcriptome dataset at the module level for each individual subject — e.g., stratifying patients by individual molecular fingerprints rather than comparing pooled groups.

## When NOT to use

- The analysis goal is a pooled group-vs-group comparison rather than individual patient stratification — use Groupcomparison/Groupcomparisonlimma and gridplot instead
- The input expression matrix is Log2 transformed or lacks valid Gene Symbols as row names, violating the Individualcomparison input requirements
- The data are not human blood transcriptome profiles — the BloodGen3 repertoire is specifically designed for human blood and other tissues or species require separate frameworks

## Inputs

- Normalized (non-Log2) gene expression matrix with valid Gene Symbols as rows and Sample IDs as columns (SummarizedExperiment or matrix)
- Sample annotation dataframe with columns for Sample ID and group annotation (e.g., Group_test with 'Sepsis'/'Control' values)
- Reference group name (e.g., 'Control') within the Group_column

## Outputs

- Individual_df dataframe: per-sample module-level percentages of genes increased or decreased
- Fingerprint heatmap PDF saved in the working directory, rows split by aggregate, with modules marked increased/decreased at the chosen cutoff (default 15%)

## How to apply

Load the normalized expression data as a SummarizedExperiment (e.g., GSE13015 via ExperimentHub resource EH5429); the matrix must have valid Gene Symbols as row names and must not be Log2 transformed. Run BloodGen3Module's Individualcomparison with FC = 1.5, DIFF = 10 (the defaults), Group_column = 'Group_test', and Ref_group = 'Control' to compute, per sample, the percentage of genes increased or decreased for each module relative to the control group. Then call fingerprintplot on the resulting Individual_df with cutoff = 15 (the visualization default, adjustable from 0–100), rowSplit = TRUE to split heatmap rows by module aggregate, Ref_group = 'Control', and optionally Aggregate = 'A28' to focus on a specific aggregate; the fingerprint heatmap is written as a PDF to the working directory. Validation rests on modules being marked increased/decreasing only where at least 15% of the module's constitutive genes pass the FC = 1.5 / DIFF = 10 thresholds relative to the reference group.

## Related tools

- **BloodGen3module** (R package supplying the Individualcomparison and fingerprintplot functions that compute per-sample module percentages and render the fingerprint heatmap PDF) — github.com/Drinchai/BloodGen3Module
- **R** (Runtime environment in which the BloodGen3module analysis workflow (ExperimentHub data loading through heatmap visualization) is executed)
- **ExperimentHub** (Source of the example GSE13015 SummarizedExperiment resource (EH5429) used as expression input)

## Examples

```
Individual_df = Individualcomparison(GSE13015, sample_info = NULL, FC = 1.5, DIFF = 10, Group_column = "Group_test", Ref_group = "Control"); fingerprintplot(Individual_df, cutoff = 15, rowSplit = TRUE, Ref_group = "Control", Aggregate = "A28")
```

## Evaluation signals

- The fingerprint heatmap PDF file is produced in the specified working directory
- Modules are marked increased/decreased only where at least 15% of the module's constitutive genes pass the FC = 1.5 / DIFF = 10 thresholds relative to the Ref_group
- Individual_df contains one entry per sample with module-level percentages, matching all Sample IDs in the input matrix
- For the sepsis reference cohort, aggregate A1 (lymphocytic cells/responses) shows consistent decreases and A35 (inflammatory neutrophil responses) shows consistent increases across patients
- Heatmap rows are split by module aggregate when rowSplit = TRUE, and the aggregate filter (e.g., 'A28') restricts display to that aggregate

## Limitations

- The default cut off is set at fold change (FC) = 1.5 and absolute difference (DIFF) = 10; changing these thresholds changes which modules are called per individual
- The visualization cutoff default is 15% but can be changed to any value between 0 and 100%, affecting heatmap display density
- The sample annotation file must be loaded using a specific name = 'sample.info' and column names for conditions must be specified exactly
- The BloodGen3 repertoire is specific to human blood transcriptome data; other tissues or species require separate frameworks
- The module repertoire is not a ground truth — some aggregates are heterogeneous in functionality

## Evidence

- [readme] The **Individualcomparison** function will perform an individual sample comparison analysis in reference to a control sample or group of samples.: "The **Individualcomparison** function will perform an individual sample comparison analysis in reference to a control sample or group of samples."
- [readme] The default cut off is set at fold change (FC) =1.5 and absolute difference (DIFF) =10.: "The default cut off is set at fold change (FC) =1.5 and absolute difference (DIFF) =10."
- [readme] The default cut off for visualization is set at 15%, it can changed to any value between 0-100%.: "The default cut off for visualization is set at 15%, it can changed to any value between 0-100%."
- [readme] The **fingerprintplot** ffunction will generate fingerprint heatmap plots as a PDF file.: "The **fingerprintplot** ffunction will generate fingerprint heatmap plots as a PDF file. The file will be saved in the working directory specified for the analysis."
- [readme] Matrix of normalized expression data (that should not be Log2 transformed). Genes should be arranged as rows and Sample ID as columns.Row names are required to be valid Gene Symbols.: [evidence span withheld — anchor status: external_doc]
- [methods] Expressing results “at the module level” as the percentage of genes that are increased or decreased.: "Expressing results “at the module level” as the percentage of genes that are increased or decreased."
