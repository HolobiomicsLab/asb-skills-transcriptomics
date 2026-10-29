---
name: module-repertoire-abundance-quantification
description: Use when analyzing normalized human blood transcriptome matrices against
  the fixed BloodGen3 repertoire to score module-level increase/decrease percentages
  per sample or group, e.g., to detect individual subjects carrying an interferon
  signature.
when_to_use_negative:
- Input is raw counts or already Log2-transformed expression data — the functions
  require non-Log2-transformed normalized data and perform the transformation themselves
- Analyzing non-blood tissue or non-human species — the BloodGen3 repertoire is specifically
  designed for human blood transcriptome data and such analyses would require a separate
  framework
- The goal is de novo module discovery from a single dataset rather than interpretation
  against a fixed, reusable repertoire
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: BloodGen3Module (R package)
  role: Provides Groupcomparison, Groupcomparisonlimma, Individualcomparison, gridplot
    and fingerprintplot functions that annotate the expression matrix with module
    membership, quantify increased/decreased gene percentages per module, and render
    fingerprint plots
  repo: github.com/Drinchai/BloodGen3Module
- name: BloodGen3
  role: The fixed third-generation repertoire of 382 transcriptional modules (16 aggregates)
    against which abundance is quantified
  repo: https://www.nature.com/articles/s41467-021-24584-w.pdf
- name: ExperimentHub
  role: Source of example/reference normalized expression data (e.g., query for 'GSE13015'
    and retrieve res[['EH5429']])
- name: SummarizedExperiment
  role: Container used to load ExperimentHub expression datasets as SummarizedExperiment
    objects for input to the module analysis
- name: R
  role: Runtime environment in which the BloodGen3Module workflows are executed
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-24584-w
    title: Development of a fixed module repertoire for the analysis and interpretation
      of blood transcriptome data
  source_content_hash: sha256:2034cc74abae6926589f701664729a95646aff74fc2b541dbcd673b7e32f7825
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
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# module-repertoire-abundance-quantification

## Summary

Quantifies transcript abundance at the fixed BloodGen3 module level, expressed as the percentage of constitutive genes in each of the 382 modules that are increased or decreased, for both group comparisons and single-sample (individual) analyses against a reference group. It underlies fingerprint heatmaps and aggregate-level stratification such as the A28 interferon signature.

## When to use

Use when analyzing normalized human blood transcriptome matrices against the fixed BloodGen3 repertoire to score module-level increase/decrease percentages per sample or group, e.g., to detect individual subjects carrying an interferon signature.

## When NOT to use

- Input is raw counts or already Log2-transformed expression data — the functions require non-Log2-transformed normalized data and perform the transformation themselves
- Analyzing non-blood tissue or non-human species — the BloodGen3 repertoire is specifically designed for human blood transcriptome data and such analyses would require a separate framework
- The goal is de novo module discovery from a single dataset rather than interpretation against a fixed, reusable repertoire

## Inputs

- Normalized (non-Log2-transformed) gene expression matrix with valid HGNC gene symbols as row names and Sample IDs as columns
- Sample annotation dataframe ('sample.info') with one Sample ID column (matching data.matrix column names) and one group annotation column
- BloodGen3 fixed module repertoire membership (Gen3_ann / Module_listGen3) shipped with the BloodGen3Module package
- Analysis parameters: FC and DIFF cutoffs, Group_column, Test_group, Ref_group, visualization cutoff

## Outputs

- Group_df or Individual_df dataframe: percentage of increased/decreased constitutive genes per module per group or sample
- Fingerprint gridplot PDF (group comparisons) or fingerprint heatmap PDF (individual comparisons)
- Per-cohort stratification counts of subjects with increased abundance in a target aggregate (e.g., A28 interferon modules)

## How to apply

Load the normalized, non-Log2-transformed expression matrix (gene symbols as rows, Sample IDs as columns) and a sample annotation table with columns for Sample ID and group names (e.g., Group_test with 'Sepsis'/'Control'). Run Groupcomparison (or Groupcomparisonlimma) for group analyses with FC = 1.5, pval = 0.1, FDR = TRUE, or Individualcomparison for single-sample analyses against the Ref_group using the defaults FC = 1.5 and DIFF = 10. Both functions express results at the module level as the percentage of constitutive genes increased or decreased per module. Visualize with gridplot (group results) or fingerprintplot (individual heatmaps), using the default visualization cutoff of 15% (range 0–100%), rowSplit = TRUE, and optionally restrict to one aggregate such as Aggregate = 'A28' for the six interferon modules. Validate by confirming the PDF plots exist in the working directory and that stratification counts (e.g., 3/22 melanoma, 11/40 JDM subjects with increased A28 module abundance) reproduce reported figures.

## Related tools

- **BloodGen3Module (R package)** (Provides Groupcomparison, Groupcomparisonlimma, Individualcomparison, gridplot and fingerprintplot functions that annotate the expression matrix with module membership, quantify increased/decreased gene percentages per module, and render fingerprint plots) — github.com/Drinchai/BloodGen3Module
- **BloodGen3** (The fixed third-generation repertoire of 382 transcriptional modules (16 aggregates) against which abundance is quantified) — https://www.nature.com/articles/s41467-021-24584-w.pdf
- **ExperimentHub** (Source of example/reference normalized expression data (e.g., query for 'GSE13015' and retrieve res[['EH5429']]))
- **SummarizedExperiment** (Container used to load ExperimentHub expression datasets as SummarizedExperiment objects for input to the module analysis)
- **R** (Runtime environment in which the BloodGen3Module workflows are executed)

## Examples

```
Individual_df = Individualcomparison(GSE13015, sample_info=NULL, FC=1.5, DIFF=10, Group_column="Group_test", Ref_group="Control")
```

## Evaluation signals

- PDF fingerprint outputs (gridplot/heatmap) exist in the specified working directory with default visualization cutoff of 15%
- Individualcomparison runs with the default cutoffs FC = 1.5 and DIFF = 10; group analyses use FC = 1.5 and FDR = TRUE with pval = 0.1
- Per-cohort stratification counts match published values: 3/22 melanoma and 11/40 juvenile dermatomyositis subjects classified with increased A28 interferon module abundance
- Module-level result values are interpretable as percentages of constitutive genes increased/decreased (0–100%)
- Sample IDs in the annotation table exactly match the column names of the expression matrix

## Limitations

- The BloodGen3 repertoire was constructed at the probe level (Illumina HT12 v3 BeadArrays); 388 of 14,168 probes are duplicate genes found in different modules, requiring a handling choice, though the impact is small
- Some module aggregates are heterogeneous in function and annotation titles are subjective; the repertoire is not a ground truth
- Visualization and analysis cutoffs (15% visualization default; FC = 1.5, DIFF = 10 for individual analysis) are tunable, so choices must be reported consistently to reproduce published stratifications

## Evidence

- [readme] The **Individualcomparison** function will perform an individual sample comparison analysis in reference to a control sample or group of samples.: "The **Individualcomparison** function will perform an individual sample comparison analysis in reference to a control sample or group of samples."
- [readme] The default cut off for visualization is set at 15%, it can changed to any value between 0-100%.: "The default cut off for visualization is set at 15%, it can changed to any value between 0-100%."
- [readme] The default cut off is set at fold change (FC) =1.5 and absolute difference (DIFF) =10.: "The default cut off is set at fold change (FC) =1.5 and absolute difference (DIFF) =10."
- [methods] Running statistical tests to determine the proportion of constitutive genes that are differentially expressed for each module: "Running statistical tests to determine the proportion of constitutive genes that are differentially expressed for each module"
- [abstract] the development of a new fixed repertoire of transcriptional modules, BloodGen3, that is designed to serve as a stable reusable framework: "the development of a new fixed repertoire of transcriptional modules, BloodGen3, that is designed to serve as a stable reusable framework"
