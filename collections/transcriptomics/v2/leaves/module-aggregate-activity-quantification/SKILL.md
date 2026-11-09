---
name: module-aggregate-activity-quantification
description: Use when normalized blood expression data (e.g. GSE13015/EH5429) must
  be reduced to per-module or per-aggregate differential-expression percentages for
  cross-cohort fingerprinting and clustering.
when_to_use_negative:
- The input is not human blood transcriptome data — the BloodGen3 repertoire is specifically
  designed for human blood; other tissues or species require separate frameworks.
- The expression matrix is Log2-transformed — functions expect un-Log2-transformed
  data since they transform internally.
- You need single-gene or cell-type deconvolution answers rather than fixed-module
  percentage activity (ABIS/CIBERSORT-style analyses serve that purpose).
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0203
tools:
- name: BloodGen3Module
  role: R package providing Groupcomparison, Groupcomparisonlimma, Individualcomparison
    and fingerprintplot functions for module/aggregate activity quantification and
    visualization
  repo: github.com/Drinchai/BloodGen3Module
- name: R
  role: Runtime environment in which the BloodGen3module analysis workflows are executed
- name: ExperimentHub
  role: Source of the GSE13015 reference expression data and sample annotation (entry
    EH5429)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-24584-w
    title: Development of a fixed module repertoire for the analysis and interpretation
      of blood transcriptome data
  source_content_hash: sha256:0771a3d1e01f25c975627fc4b840fcc89b480cd282df36b767bdccd0d628434c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BloodGen3
  - BloodGen3Module
  - ExperimentHub
  - R
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# module-aggregate-activity-quantification

## Summary

Quantifies activity of fixed blood transcriptome modules (BloodGen3, 382 modules) and their parent aggregates as the percentage of constitutive genes increased or decreased per cohort comparison, enabling aggregate-level heatmaps such as the 16-reference-state interferon aggregate fingerprint.

## When to use

Use when normalized blood expression data (e.g. GSE13015/EH5429) must be reduced to per-module or per-aggregate differential-expression percentages for cross-cohort fingerprinting and clustering.

## When NOT to use

- The input is not human blood transcriptome data — the BloodGen3 repertoire is specifically designed for human blood; other tissues or species require separate frameworks.
- The expression matrix is Log2-transformed — functions expect un-Log2-transformed data since they transform internally.
- You need single-gene or cell-type deconvolution answers rather than fixed-module percentage activity (ABIS/CIBERSORT-style analyses serve that purpose).

## Inputs

- Normalized (not Log2-transformed) expression matrix with valid gene symbols as row names and Sample IDs as columns (e.g. GSE13015 via ExperimentHub EH5429)
- Sample annotation dataframe with Sample ID and group columns (Group_column, Test_group, Ref_group)
- Fixed module repertoire annotation (Module_listGen3/Gen3_ann, 382 modules, 27 multi-module aggregates)

## Outputs

- Group_df / Group_limma dataframe: per-module percentage of genes increased or decreased
- Aggregate-level summary table for the 27 multi-module aggregates across 16 reference states
- Fingerprint heatmap PDF from fingerprintplot (cutoff = 15, rowSplit = TRUE)

## How to apply

Load the normalized expression matrix un-Log2-transformed (e.g. GSE13015 via ExperimentHub entry EH5429) with the BloodGen3Module R package, annotating genes with module membership from the fixed repertoire (Module_listGen3/Gen3_ann). For each of the 16 cohorts run Groupcomparison (t-test) or Groupcomparisonlimma with FC = 1.5, pval = 0.1, FDR = TRUE, specifying Group_column, Test_group and Ref_group, to obtain the percentage of increased/decreased constitutive genes per module. Summarize module-level results into the 27 multi-module aggregates (e.g. A28) as percentages of genes increased or decreased. Optionally call fingerprintplot with cutoff = 15 (the default visualization cutoff, adjustable 0–100%), rowSplit = TRUE and Aggregate = 'A28' to emit a heatmap PDF. Validate by checking that module percentages fall in 0–100% and that known biology is recovered (e.g. A28 marked increases in SLE, influenza, HIV, tuberculosis).

## Related tools

- **BloodGen3Module** (R package providing Groupcomparison, Groupcomparisonlimma, Individualcomparison and fingerprintplot functions for module/aggregate activity quantification and visualization) — github.com/Drinchai/BloodGen3Module
- **R** (Runtime environment in which the BloodGen3module analysis workflows are executed)
- **ExperimentHub** (Source of the GSE13015 reference expression data and sample annotation (entry EH5429))

## Examples

```
Group_df <- Groupcomparison(GSE13015, sample_info = NULL, FC = 1.5, pval = 0.1, FDR = TRUE, Group_column = "Group_test", Test_group = "Sepsis", Ref_group = "Control")
```

## Evaluation signals

- Output dataframe reports each module's percentage of increased/decreased genes bounded between 0 and 100%.
- Groupcomparison was run with FC = 1.5, pval = 0.1, FDR = TRUE (BH-method) and correctly named Group_column/Test_group/Ref_group.
- Aggregate-level summary covers the 27 multi-module aggregates (including A28) across all 16 reference cohorts.
- Fingerprint heatmap PDF exists in the working directory with cutoff = 15 and rows split by aggregate; A28 row shows marked increases in SLE, influenza, HIV and tuberculosis.
- Cross-cohort clustering separates acute HIV infection from the remaining 14 states, driven by suppression of inflammation/myeloid aggregates (A34–A38) and increase of lymphocyte-associated aggregates (A1–A8).

## Limitations

- The BloodGen3 repertoire was built at probe level on Illumina HT12 v3 BeadArrays; 388 of 14,168 probes are duplicate genes appearing in different modules, requiring a handling choice.
- The repertoire is not a ground truth — some aggregates are heterogeneous and functional annotation titles confer subjectivity.
- It is specific to human blood; analyses of other tissues or species would require separate frameworks.

## Evidence

- [methods] Running statistical tests to determine the proportion of constitutive genes that are differentially expressed for each module: "2.	Running statistical tests to determine the proportion of constitutive genes that are differentially expressed for each module."
- [methods] Expressing results at the module level as the percentage of genes that are increased or decreased: "3.	Expressing results “at the module level” as the percentage of genes that are increased or decreased."
- [readme] Groupcomparison with FC = 1.5, pval = 0.1, FDR = TRUE and Group_test/Control groups: "Group_df <- Groupcomparison(GSE13015, sample_info = NULL, FC = 1.5, pval = 0.1 , FDR = TRUE, Group_column = "Group_test", Test_group = "Sepsis", Ref_group = "Control")"
- [readme] Default visualization cutoff is 15%, changeable between 0 and 100%: "The default cut off for visualization is set at 15%, it can changed to any value between 0-100%."
- [readme] Expression matrix should not be Log2 transformed with gene symbols as rows: [evidence span withheld — anchor status: external_doc]
- [discussion] Repertoire not a ground truth; aggregates heterogeneous and annotation titles subjective: "our module repertoire is not meant to constitute a ground truth. Indeed, some aggregates are clearly not homogeneous in terms of functionality"
