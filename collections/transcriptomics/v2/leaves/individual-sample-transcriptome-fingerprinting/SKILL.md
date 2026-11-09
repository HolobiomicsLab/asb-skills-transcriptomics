---
name: individual-sample-transcriptome-fingerprinting
description: Use when the goal is to stratify individual subjects within a cohort
  by their module-level expression fingerprint — e.g., detecting which patients carry
  an interferon (A28) signature when group-level tests show no overall change.
when_to_use_negative:
- Input data are from non-blood tissues or other species — the BloodGen3 repertoire
  is specifically designed for human blood transcriptome data
- The goal is a group-level comparison (case vs. control differential expression)
  — use Groupcomparison/Groupcomparisonlimma with gridplot instead
- The expression matrix is Log2-transformed — the functions expect non-Log2-transformed
  data and perform the transformation themselves
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0203
tools:
- name: BloodGen3Module
  role: R package providing Individualcomparison (per-sample module-level percent
    increased/decreased with FC=1.5, DIFF=10) and fingerprintplot (heatmap PDF, cutoff=15%,
    rowSplit, Aggregate='A28')
  repo: github.com/Drinchai/BloodGen3Module
- name: BloodGen3
  role: Fixed repertoire of 382 transcriptional modules (module + aggregate two-tiered
    structure, e.g., A28 interferon) against which individual fingerprints are computed
  repo: github.com/Drinchai/BloodGen3Module
- name: R
  role: Runtime environment for the BloodGen3Module package and SummarizedExperiment/ExperimentHub
    data loading
- name: ExperimentHub
  role: Source of example/reference cohort data, e.g., GSE13015 via resource EH5429
- name: SummarizedExperiment
  role: Container format expected for the expression matrix and sample annotation
    input
provenance:
  source_task_ids:
  - task_005
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-24584-w
    title: Development of a fixed module repertoire for the analysis and interpretation
      of blood transcriptome data
  source_content_hash: sha256:11207b7748c7718c60ba4bf0f146c925a0988eb3bccf69dcc427f4140feed45e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BloodGen3
  - BloodGen3Module
  - BloodGen3module (R package)
  - ExperimentHub
  - R
  - SummarizedExperiment
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# individual-sample-transcriptome-fingerprinting

## Summary

Per-subject blood transcriptome fingerprinting using the BloodGen3 fixed module repertoire: each sample is compared against a control reference via the Individualcomparison function (FC = 1.5, DIFF = 10) and visualized as a fingerprint heatmap with fingerprintplot, revealing subjects carrying molecular signatures (e.g., A28 interferon) that group-level comparisons miss.

## When to use

Use when the goal is to stratify individual subjects within a cohort by their module-level expression fingerprint — e.g., detecting which patients carry an interferon (A28) signature when group-level tests show no overall change.

## When NOT to use

- Input data are from non-blood tissues or other species — the BloodGen3 repertoire is specifically designed for human blood transcriptome data
- The goal is a group-level comparison (case vs. control differential expression) — use Groupcomparison/Groupcomparisonlimma with gridplot instead
- The expression matrix is Log2-transformed — the functions expect non-Log2-transformed data and perform the transformation themselves

## Inputs

- Normalized blood transcriptome expression matrix (not Log2 transformed), genes as rows (valid Gene Symbols), samples as columns
- Sample annotation table with columns for Sample ID (matching data.matrix) and group names (e.g., 'Group_test' with 'Sepsis'/'Control' values)
- SummarizedExperiment object, e.g., GSE13015 retrieved from ExperimentHub (EH5429)

## Outputs

- Individual_df dataframe: per-sample percentage of genes increased or decreased at the module level for each of the 382 modules
- Fingerprint heatmap PDF saved in the working directory (via fingerprintplot), optionally restricted to a module aggregate such as A28
- Per-cohort stratification counts of subjects with increased module abundance (e.g., 3/22 melanoma, 11/40 JDM with interferon signatures)

## How to apply

Load a normalized (non-Log2-transformed) SummarizedExperiment expression matrix, e.g., GSE13015 from ExperimentHub resource EH5429, with valid Gene Symbols as row names. Run Individualcomparison against a control reference group with the default cutoffs FC = 1.5 and absolute difference DIFF = 10, specifying Group_column (e.g., 'Group_test') and Ref_group (e.g., 'Control'), to compute per sample the percentage of constitutive genes increased or decreased at the module level. Feed the resulting Individual_df into fingerprintplot with visualization cutoff = 15 (percent of module genes passing thresholds), rowSplit = TRUE, and optionally Aggregate = 'A28' to restrict to the six interferon modules, generating a fingerprint heatmap PDF. Rationale: the two-tiered module/aggregate design of the fixed 382-module repertoire allows inter-individual variability to be summarized as increased/decreased percentages, exposing heterogeneity that group-averaging obscures (e.g., 3/22 melanoma and 11/40 JDM subjects with increased A28 abundance).

## Related tools

- **BloodGen3Module** (R package providing Individualcomparison (per-sample module-level percent increased/decreased with FC=1.5, DIFF=10) and fingerprintplot (heatmap PDF, cutoff=15%, rowSplit, Aggregate='A28')) — github.com/Drinchai/BloodGen3Module
- **BloodGen3** (Fixed repertoire of 382 transcriptional modules (module + aggregate two-tiered structure, e.g., A28 interferon) against which individual fingerprints are computed) — github.com/Drinchai/BloodGen3Module
- **R** (Runtime environment for the BloodGen3Module package and SummarizedExperiment/ExperimentHub data loading)
- **ExperimentHub** (Source of example/reference cohort data, e.g., GSE13015 via resource EH5429)
- **SummarizedExperiment** (Container format expected for the expression matrix and sample annotation input)

## Examples

```
Individual_df = Individualcomparison(GSE13015, sample_info=NULL, FC=1.5, DIFF=10, Group_column="Group_test", Ref_group="Control"); fingerprintplot(Individual_df, cutoff=15, rowSplit=TRUE, Ref_group="Control", Aggregate="A28", filename=tempfile())
```

## Evaluation signals

- A fingerprint heatmap PDF exists in the working directory after running fingerprintplot
- Modules are marked increased/decreased only where at least 15% of the module's constitutive genes pass FC = 1.5 / DIFF = 10 thresholds relative to the reference group
- Individual_df contains per-sample percentage values (0–100) for increased and decreased genes per module
- Per-cohort stratification counts of subjects with increased A28 module abundance match reported values (3/22 melanoma, 11/40 JDM)

## Limitations

- The BloodGen3 repertoire is not a ground truth: some aggregates are heterogeneous in functionality, and heterogeneity also exists within modules
- The repertoire was constructed at the probe level on Illumina HT12 v3 BeadArrays, which pre-dates RNA-seq; 388 of 14,168 probes are duplicate genes found in different modules
- The repertoire is specific to human blood; analyses of other tissues or species would require separate frameworks
- Results could change if the repertoire is altered by addition or removal of construction datasets, with more variation expected at the module level than the aggregate level

## Evidence

- [methods] The **Individualcomparison** function will perform an individual sample comparison analysis in reference to a control sample or group of samples.: "The **Individualcomparison** function will perform an individual sample comparison analysis in reference to a control sample or group of samples."
- [readme] The default cut off is set at fold change (FC) =1.5 and absolute difference (DIFF) =10.: "The default cut off is set at fold change (FC) =1.5 and absolute difference (DIFF) =10."
- [readme] The **fingerprintplot** ffunction will generate fingerprint heatmap plots as a PDF file.: "The **fingerprintplot** ffunction will generate fingerprint heatmap plots as a PDF file."
- [methods] Expressing results “at the module level” as the percentage of genes that are increased or decreased.: "Expressing results “at the module level” as the percentage of genes that are increased or decreased."
- [readme] Expression matrix and sample annotation files are required to perform this analysis.: "Expression matrix and sample annotation files are required to perform this analysis."
