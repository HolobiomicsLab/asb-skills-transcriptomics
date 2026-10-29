---
name: cross-cohort-transcriptome-clustering-interpretation
description: Use when multiple blood transcriptome cohorts need to be compared at
  the module-aggregate level using the fixed BloodGen3 repertoire, e.g. to cluster
  disease states by aggregate activity or reproduce the aggregate abundance heatmap.
when_to_use_negative:
- The input data are from tissues or species other than human blood — the BloodGen3
  repertoire is specifically designed for human blood transcriptome data and other
  contexts would require a separate framework
- The goal is a per-sample fingerprint rather than a cross-cohort aggregate-level
  comparison — use Individualcomparison instead of group comparison workflows
- The expression matrix is already Log2-transformed — the BloodGen3Module functions
  expect un-Log2-transformed data since they transform internally
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0203
tools:
- name: BloodGen3Module
  role: R package providing Groupcomparison, Groupcomparisonlimma and fingerprintplot
    to compute module-level percentages of increased/decreased genes and render the
    cross-cohort aggregate fingerprint heatmap
  repo: github.com/Drinchai/BloodGen3Module
- name: R
  role: Execution environment for the BloodGen3Module workflows, ExperimentHub data
    retrieval and hierarchical clustering of the heatmap
- name: BloodGen3
  role: The fixed 382-module repertoire (with functional annotations and aggregate
    grouping) that supplies the module membership used to summarize results at the
    aggregate level
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-24584-w
    title: Development of a fixed module repertoire for the analysis and interpretation
      of blood transcriptome data
  source_content_hash: sha256:6df5016658882b48f378d8063011b547bdd555c8dddf65390d1e052c8bf4d888
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BloodGen3
  - BloodGen3Module
  - R
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# cross-cohort-transcriptome-clustering-interpretation

## Summary

Reproduce and interpret cross-cohort fingerprint heatmaps of the BloodGen3 module repertoire, summarizing the 382 modules into their 27 multi-module aggregates and comparing their activity (percentage of genes increased/decreased) across the 16 reference health states to reveal top-level clustering structure (e.g., acute HIV separating from the remaining states).

## When to use

Use when multiple blood transcriptome cohorts need to be compared at the module-aggregate level using the fixed BloodGen3 repertoire, e.g. to cluster disease states by aggregate activity or reproduce the aggregate abundance heatmap.

## When NOT to use

- The input data are from tissues or species other than human blood — the BloodGen3 repertoire is specifically designed for human blood transcriptome data and other contexts would require a separate framework
- The goal is a per-sample fingerprint rather than a cross-cohort aggregate-level comparison — use Individualcomparison instead of group comparison workflows
- The expression matrix is already Log2-transformed — the BloodGen3Module functions expect un-Log2-transformed data since they transform internally

## Inputs

- Normalized blood gene expression matrix (un-Log2-transformed, genes as rows as valid Gene Symbols, Sample IDs as columns), e.g. GSE13015 from ExperimentHub entry EH5429
- Sample annotation dataframe with a Sample ID column matching data.matrix and a group column naming the cohort/health state (e.g. Group_test)
- Fixed BloodGen3 module repertoire annotations (Module_listGen3/Gen3_ann) mapping the 382 modules to parent aggregates

## Outputs

- Per-cohort module-level group comparison results (Group_df or Group_limma) expressing percentage of genes increased or decreased per module
- Aggregate-level matrix for the 27 multi-module aggregates across the 16 reference states
- Fingerprint heatmap PDF with hierarchical clustering of cohorts by aggregate activity (rowSplit = TRUE, cutoff = 15)

## How to apply

Load normalized (un-Log2-transformed) expression data and sample annotation (GSE13015 via ExperimentHub entry EH5429) into R with the BloodGen3Module package attached, and annotate the expression matrix with module membership from the fixed repertoire (Module_listGen3/Gen3_ann), grouping the 382 modules into parent aggregates (e.g., A28). For each of the 16 cohorts run Groupcomparison (t-test) or Groupcomparisonlimma with FC = 1.5, pval = 0.1, FDR = TRUE, specifying Group_column, Test_group and Ref_group, to obtain the percentage of increased/decreased constitutive genes per module. Summarize module-level results to the aggregate level for the 27 multi-module aggregates, then generate the cross-cohort fingerprint heatmap with fingerprintplot using cutoff = 15 (percentage), rowSplit = TRUE, Group_column for cohort grouping, and Aggregate aggregation, writing a PDF to the working directory. Judge success by verifying the PDF exists, that hierarchical clustering separates acute HIV infection into one cluster with the remaining 14 states in a second, and that the A28 (interferon) row shows marked increases in SLE, influenza, HIV and tuberculosis; the clustering is driven by suppression of inflammation/myeloid aggregates (A34–A38) and increase of lymphocyte-associated aggregates (A1–A8).

## Related tools

- **BloodGen3Module** (R package providing Groupcomparison, Groupcomparisonlimma and fingerprintplot to compute module-level percentages of increased/decreased genes and render the cross-cohort aggregate fingerprint heatmap) — github.com/Drinchai/BloodGen3Module
- **R** (Execution environment for the BloodGen3Module workflows, ExperimentHub data retrieval and hierarchical clustering of the heatmap)
- **BloodGen3** (The fixed 382-module repertoire (with functional annotations and aggregate grouping) that supplies the module membership used to summarize results at the aggregate level)

## Examples

```
GSE13015 = res[["EH5429"]]; Group_df <- Groupcomparison(GSE13015, sample_info = NULL, FC = 1.5, pval = 0.1, FDR = TRUE, Group_column = "Group_test", Test_group = "Sepsis", Ref_group = "Control")
```

## Evaluation signals

- The output PDF heatmap exists in the working directory and shows rows split by aggregate (rowSplit = TRUE) with the 27 multi-module aggregates
- Hierarchical clustering of the 16 states separates acute HIV infection into one cluster and the remaining 14 states into a second cluster
- The A28 aggregate row shows marked increases (above the 15% visualization cutoff) in SLE, influenza, HIV and tuberculosis cohorts
- Group comparison parameters are FC = 1.5, pval = 0.1, FDR = TRUE (BH method) for every cohort comparison
- Module percentages are computed against a specified Ref_group and expression input was not pre-Log2-transformed

## Limitations

- The BloodGen3 repertoire was constructed at the probe level on Illumina HT12 v3 BeadArrays (pre-dating RNA-seq); 388 of 14,168 probes are duplicate genes found in different modules requiring a handling choice
- The module repertoire is not a ground truth — some aggregates are heterogeneous in functionality and annotation titles confer subjectivity
- Repertoire composition could change if datasets are added or removed from its construction; more variation is expected at the module level than at the aggregate level

## Evidence

- [readme] Group_df <- Groupcomparison(GSE13015, sample_info = NULL, FC = 1.5, pval = 0.1 , FDR = TRUE, Group_column = "Group_test", Test_group = "Sepsis", Ref_group = "Control"): "Group_df <- Groupcomparison(GSE13015,
                            sample_info = NULL,
                            FC = 1.5,
                            pval = 0.1 ,
                            FDR ="
- [readme] fingerprintplot function generates fingerprint heatmap plots as a PDF file with default 15% cutoff: "fingerprintplot(Individual_df,
                sample_info = NULL,
                cutoff = 15,
                rowSplit= TRUE ,
                Group_column= "Group_test""
- [methods] The default cut off for visualization is set at 15%; it can be changed to any value between 0 and 100%.: "The default cut off for visualization is set at 15%; it can be changed to any value between 0 and 100%."
- [discussion] BloodGen3 repertoire was constructed at the probe level on Illumina HT12 v3 Beadarrays: "the BloodGen3 repertoire was constructed at the probe level (those present on the Illumina HT12 v3 Beadarrays)"
- [discussion] Two-tiered grouping of modules at the module level and module aggregate level: "the two-tiered grouping, first at the module level and at the module aggregate level, is new for this third-generation module repertoire"
