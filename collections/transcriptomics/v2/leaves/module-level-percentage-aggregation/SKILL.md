---
name: module-level-percentage-aggregation
description: Use when comparing blood transcriptome profiles between patient groups
  and results must be expressed as per-module percentage of increased/decreased constitutive
  genes for BloodGen3 fingerprint grids or heatmaps.
when_to_use_negative:
- Analyzing tissues other than blood or non-human species — the BloodGen3 repertoire
  is specifically designed for human blood transcriptome data
- Individual single-sample fingerprinting relative to a control — use Individualcomparison
  and fingerprintplot (heatmap) instead of Groupcomparison/gridplot
- Input expression matrix is already log2 transformed — the functions expect non-log2-normalized
  data and transform internally
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3360
tools:
- name: BloodGen3module (R package)
  role: R package providing Groupcomparison and gridplot functions to compute module-level
    percent increase/decrease and render fingerprint grids
  repo: github.com/Drinchai/BloodGen3Module
- name: BloodGen3
  role: Fixed repertoire of 382 transcriptional modules (aggregates A1–A38) used to
    annotate gene membership for the percentage aggregation
- name: R
  role: Runtime environment for the analysis workflow (devtools/BiocManager installation,
    ExperimentHub data access)
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-24584-w
    title: Development of a fixed module repertoire for the analysis and interpretation
      of blood transcriptome data
  source_content_hash: sha256:f032ee5145df42273ba5f1f68988c3541f738a15b1716c6896d3d6fd0968d580
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BloodGen3
  - BloodGen3module (R package)
  - R
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3360
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# module-level-percentage-aggregation

## Summary

Aggregates blood transcriptome group-comparison results to the BloodGen3 module level by computing, for each of the 382 constitutive modules, the percentage of its genes that are significantly increased or decreased between a test and reference group, enabling fingerprint grid visualization.

## When to use

Use when comparing blood transcriptome profiles between patient groups and results must be expressed as per-module percentage of increased/decreased constitutive genes for BloodGen3 fingerprint grids or heatmaps.

## When NOT to use

- Analyzing tissues other than blood or non-human species — the BloodGen3 repertoire is specifically designed for human blood transcriptome data
- Individual single-sample fingerprinting relative to a control — use Individualcomparison and fingerprintplot (heatmap) instead of Groupcomparison/gridplot
- Input expression matrix is already log2 transformed — the functions expect non-log2-normalized data and transform internally

## Inputs

- Normalized expression data matrix (not log2 transformed, genes as rows with valid gene symbols, sample IDs as columns), e.g. GSE13015/EH5429 from ExperimentHub as a SummarizedExperiment
- Sample annotation dataframe named 'sample.info' with a Sample ID column matching the matrix and a group annotation column
- Fixed module repertoire annotation (Module_listGen3/Gen3_ann) mapping genes to the 382 BloodGen3 modules and aggregates A1–A38

## Outputs

- Group_df dataframe with per-module percentage of increased/decreased constitutive genes after group comparison
- Fingerprint grid plot written as a PDF file in the working directory via gridplot (default cutoff 15%)

## How to apply

Load a normalized (non-log2-transformed) expression matrix with gene symbols as row names plus a sample annotation table, then run Groupcomparison with a t-test (or Groupcomparisonlimma for limma), FC = 1.5, pval = 0.1 with FDR = TRUE (BH method), specifying Group_column, Test_group, and Ref_group. The function annotates the matrix with module membership from the fixed Module_listGen3 repertoire, performs statistical tests per module, and expresses results at the module level as the percentage of genes increased or decreased. Downstream, gridplot renders the two-tiered aggregate/module fingerprint grid as a PDF using a default visualization cutoff of 15% (adjustable 0–100%). The two-tiered grouping — first at the module level and at the module aggregate level (A1–A38) — is specific to this third-generation repertoire; modules within an aggregate are ordered by functional annotation then alphabetically.

## Related tools

- **BloodGen3module (R package)** (R package providing Groupcomparison and gridplot functions to compute module-level percent increase/decrease and render fingerprint grids) — github.com/Drinchai/BloodGen3Module
- **BloodGen3** (Fixed repertoire of 382 transcriptional modules (aggregates A1–A38) used to annotate gene membership for the percentage aggregation)
- **R** (Runtime environment for the analysis workflow (devtools/BiocManager installation, ExperimentHub data access))

## Examples

```
Group_df <- Groupcomparison(GSE13015, sample_info = NULL, FC = 1.5, pval = 0.1, FDR = TRUE, Group_column = "Group_test", Test_group = "SLE", Ref_group = "Control"); gridplot(Group_df, cutoff = 15, Ref_group = "Control", filename = "fingerprint.pdf")
```

## Evaluation signals

- The output PDF fingerprint grid exists in the working directory and every aggregate row A1–A38 displays modules shaded only where the module-level percentage of differentially expressed genes meets or exceeds the 15% cutoff
- Group_df output is expressed as percentages of genes increased or decreased per module, consistent with FC = 1.5 and BH-adjusted pval = 0.1 (FDR = TRUE)
- Sample annotation column names (Group_column, Test_group, Ref_group) exactly match the sample_info dataframe and Sample IDs match the expression matrix columns
- Duplicate genes handling: duplicates concern only a small minority of probes (388 out of 14,168), so module percentages should remain stable with negligible impact
- Results are reported in the two-tiered structure — module level and aggregate level — matching the Gen3 repertoire layout

## Limitations

- Some module aggregates are not homogeneous in functionality and functional annotation titles confer subjectivity; the repertoire is not a ground truth
- The repertoire was built at the probe level on Illumina HT12 v3 BeadArrays; 388 of 14,168 probes are duplicate genes found in different modules requiring a handling choice
- Adding or removing construction datasets can alter the repertoire, with more expected variation at the module level than the aggregate level

## Evidence

- [methods] Expressing results at the module level as the percentage of genes that are increased or decreased: "3.	Expressing results “at the module level” as the percentage of genes that are increased or decreased."
- [readme] The gridplot function generates a grid plot as a PDF with a default 15% visualization cutoff: [evidence span withheld — anchor status: not_found]
- [discussion] Two-tiered grouping at module and aggregate level is new for the third-generation repertoire: "the two-tiered grouping, first at the module level and at the module aggregate level, is new for this third-generation module repertoire"
- [readme] Groupcomparison performs group comparison analyses with results expressed at the module level: "The **Groupcomparison** function will perform group comparison analyses. The results are expressed “at the module level” as the percentage of genes that are increased or decreased for a given module."
- [discussion] Module ordering within aggregates on the fingerprint grid: "modules within a given aggregate (=row) are grouped first according to functional annotations and then among the different annotations, from left to right according to alphabetical order"
