---
name: interferon-module-subject-stratification
description: Use when the goal is to detect individual subjects with increased abundance
  of interferon (A28) modules in blood expression matrices, especially when group-mean
  comparisons show no significant overall change.
when_to_use_negative:
- Input expression data is already Log2-transformed — the functions expect non-Log2
  data and perform the transformation themselves
- The analysis targets non-blood tissues or non-human species — the BloodGen3 repertoire
  is specifically designed for human blood transcriptome data
- The question is whether a cohort differs on average from controls — use Groupcomparison/Groupcomparisonlimma
  and gridplot instead of individual stratification
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_3170
tools:
- name: BloodGen3Module (R package)
  role: Provides Individualcomparison and fingerprintplot functions to compute per-sample
    module-level increase/decrease percentages and render A28-restricted fingerprint
    heatmaps
  repo: github.com/Drinchai/BloodGen3Module
- name: R
  role: Runtime environment in which the package is installed (devtools::install_github
    or BiocManager) and the workflow is executed
  repo: https://github.com/Drinchai/BloodGen3Module
- name: ExperimentHub
  role: Downloads reference cohort expression data (e.g., GSE13015 as EH5429) for
    the analysis
- name: SummarizedExperiment
  role: Container class used to load and handle the expression data retrieved via
    ExperimentHub
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-24584-w
    title: Development of a fixed module repertoire for the analysis and interpretation
      of blood transcriptome data
  source_content_hash: sha256:d6d4baa2a0f052527bf2721597022353421711d6209cdf00ee7275f1d1d4de3d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BloodGen3
  - BloodGen3Module (R package)
  - BloodGen3module (R package)
  - ExperimentHub
  - R
  - SummarizedExperiment
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# interferon-module-subject-stratification

## Summary

Performs individual-level (single-sample) blood transcriptome fingerprinting restricted to the A28 interferon module aggregate using the BloodGen3Module R package, to identify subjects carrying interferon signatures in cohorts where group-level comparisons detect no overall change (e.g., 3/22 melanoma and 11/40 JDM subjects).

## When to use

Use when the goal is to detect individual subjects with increased abundance of interferon (A28) modules in blood expression matrices, especially when group-mean comparisons show no significant overall change.

## When NOT to use

- Input expression data is already Log2-transformed — the functions expect non-Log2 data and perform the transformation themselves
- The analysis targets non-blood tissues or non-human species — the BloodGen3 repertoire is specifically designed for human blood transcriptome data
- The question is whether a cohort differs on average from controls — use Groupcomparison/Groupcomparisonlimma and gridplot instead of individual stratification

## Inputs

- Normalized (non-Log2-transformed) blood gene expression matrix with genes as rows (valid Gene Symbols) and Sample IDs as columns
- Sample annotation dataframe with a Sample ID column matching data.matrix columns and a group annotation column (Group_column)
- Reference/control group definition (Ref_group) and Group_column name

## Outputs

- Individual_df dataframe of module-level percentages of increased/decreased constitutive genes per sample
- Fingerprint heatmap PDF restricted to the A28 aggregate (fingerprintplot output, e.g., 28 x 17 inches default)
- Per-cohort counts of subjects with increased A28 interferon module abundance

## How to apply

Load normalized (non-Log2-transformed) expression matrices and sample annotations from GEO/ExperimentHub into R as SummarizedExperiment objects (e.g., GSE13015 via res[["EH5429"]]). Run the Individualcomparison function against the matched control group with fold-change cutoff FC = 1.5 and absolute-difference cutoff DIFF = 10, expressing results at the module level as the percentage of constitutive genes increased or decreased. Pass the resulting Individual_df to fingerprintplot with Aggregate = "A28" to restrict the heatmap to the six interferon modules, using the default visualization cutoff of 15%, rowSplit = TRUE grouped by Group_column, and Ref_group set to the control group. Count, per cohort, the subjects classified as having increased A28 module abundance and compare against reported stratification (e.g., 3/22 melanoma, 11/40 JDM).

## Related tools

- **BloodGen3Module (R package)** (Provides Individualcomparison and fingerprintplot functions to compute per-sample module-level increase/decrease percentages and render A28-restricted fingerprint heatmaps) — github.com/Drinchai/BloodGen3Module
- **R** (Runtime environment in which the package is installed (devtools::install_github or BiocManager) and the workflow is executed) — https://github.com/Drinchai/BloodGen3Module
- **ExperimentHub** (Downloads reference cohort expression data (e.g., GSE13015 as EH5429) for the analysis)
- **SummarizedExperiment** (Container class used to load and handle the expression data retrieved via ExperimentHub)

## Examples

```
Individual_df = Individualcomparison(GSE13015, sample_info=NULL, FC=1.5, DIFF=10, Group_column="Group_test", Ref_group="Control"); fingerprintplot(Individual_df, cutoff=15, rowSplit=TRUE, Group_column="Group_test", Ref_group="Control", Aggregate="A28", filename=tempfile())
```

## Evaluation signals

- Fingerprint heatmap PDF files for each cohort are generated in the specified working directory
- Per-cohort counts of subjects with increased A28 module abundance match reported values (3/22 melanoma, 11/40 JDM)
- Individualcomparison was run with FC = 1.5 and DIFF = 10 against the matched control group
- fingerprintplot restricted output to Aggregate = "A28" (six interferon modules) with the default 15% visualization cutoff
- Expression matrix was supplied non-Log2-transformed with row names as valid Gene Symbols

## Limitations

- Duplicate genes (388 of 14,168 probes) appear in different modules; disregarding duplicates has little impact but requires a handling choice
- The module repertoire is not a ground truth; some aggregates, including interferon modules, are heterogeneous in functionality
- The repertoire was constructed at the probe level on Illumina HT12 v3 BeadArrays, which pre-dates RNA-seq
- A dominant aggregate signature may reflect cell-composition effects (e.g., erythroid A36–A38) not captured by common deconvolution panels

## Evidence

- [readme] The **Individualcomparison** function will perform an individual sample comparison analysis in reference to a control sample or group of samples.: "The **Individualcomparison** function will perform an individual sample comparison analysis in reference to a control sample or group of samples."
- [readme] The default cut off is set at fold change (FC) =1.5 and absolute difference (DIFF) =10.: "The default cut off is set at fold change (FC) =1.5 and absolute difference (DIFF) =10."
- [readme] Matrix of normalized expression data (that should not be Log2 transformed). Genes should be arranged as rows and Sample ID as columns.: [evidence span withheld — anchor status: external_doc]
- [methods] Expressing results “at the module level” as the percentage of genes that are increased or decreased.: "Expressing results “at the module level” as the percentage of genes that are increased or decreased."
- [readme] The default cut off for visualization is set at 15%; it can be changed to any value between 0 and 100%.: "The default cut off for visualization is set at 15%; it can be changed to any value between 0 and 100%."
- [discussion] they concern only a small minority of genes (388 out of 14,168 probes): "they concern only a small minority of genes (388 out of 14,168 probes)"
