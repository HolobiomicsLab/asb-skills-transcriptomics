---
name: duplicate-probe-gene-mapping-across-modules
description: Use when annotating a gene expression matrix with BloodGen3 module membership
  and some probes/genes map to multiple modules, and a sanctioned duplicate-handling
  strategy must be chosen before running Groupcomparison.
when_to_use_negative:
- The input transcriptome is not human blood data — the BloodGen3 repertoire is specifically
  designed for human blood, and analyses of other tissues or species would require
  a separate framework
- The goal is constructing or modifying a module repertoire rather than applying the
  fixed BloodGen3 repertoire to an existing expression matrix
- The expression matrix uses gene-level identifiers with no probe-to-module membership
  ambiguity, so no duplicate-gene handling decision is required
edam_operation: http://edamontology.org/operation_0336
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: BloodGen3module
  role: R package providing Groupcomparison (and Groupcomparisonlimma, gridplot, fingerprintplot)
    used to compute module-level percent-increase/decrease values on each duplicate-handling
    variant
  repo: github.com/Drinchai/BloodGen3Module
- name: R
  role: Runtime environment in which the annotated matrices are built, the two duplicate-handling
    variants are constructed, and the per-module comparison table is computed
- name: ExperimentHub
  role: Source of the GSE13015 example dataset (object EH5429) used to test the duplicate-gene
    handling strategies
- name: SummarizedExperiment
  role: Data container for the annotated expression matrix passed to Groupcomparison
    with SummarizedExperiment = TRUE
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-24584-w
    title: Development of a fixed module repertoire for the analysis and interpretation
      of blood transcriptome data
  source_content_hash: sha256:86df60ca4047407549067b4667177ea75127ad4dd87328d8d39ff3a59d7034a3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BloodGen3
  - BloodGen3module
  - ExperimentHub
  - R
  - SummarizedExperiment
  edam_operation: http://edamontology.org/operation_0336
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# duplicate-probe-gene-mapping-across-modules

## Summary

A skill for handling duplicate probes/genes that map to more than one BloodGen3 module (388 out of 14,168 probes) when running module repertoire group comparisons, by either assigning each duplicated probe the same value in every module it belongs to or disregarding duplicates entirely, both of which yield essentially comparable module-level percent-increase/decrease results.

## When to use

Use when annotating a gene expression matrix with BloodGen3 module membership and some probes/genes map to multiple modules, and a sanctioned duplicate-handling strategy must be chosen before running Groupcomparison.

## When NOT to use

- The input transcriptome is not human blood data — the BloodGen3 repertoire is specifically designed for human blood, and analyses of other tissues or species would require a separate framework
- The goal is constructing or modifying a module repertoire rather than applying the fixed BloodGen3 repertoire to an existing expression matrix
- The expression matrix uses gene-level identifiers with no probe-to-module membership ambiguity, so no duplicate-gene handling decision is required

## Inputs

- Normalized (non-log2-transformed) gene expression matrix with genes as rows, sample IDs as columns, valid Gene Symbols as row names
- BloodGen3 module membership annotation mapping probes/genes to modules and aggregates
- Sample annotation table with Sample ID and group columns (e.g. Group_test with Sepsis/Control values)
- ExperimentHub object EH5429 (GSE13015 example dataset, SummarizedExperiment)

## Outputs

- Annotated expression matrix variant A (duplicated probes retained in every module)
- Annotated expression matrix variant B (duplicated probes disregarded)
- Per-module percentage of genes increased/decreased from Group_df output for each variant
- Combined comparison table written to duplicate_gene_handling_comparison.csv (one row per module with both variants' percent values)

## How to apply

Load the annotated expression matrix (e.g. GSE13015 via ExperimentHub object EH5429) along with the BloodGen3 module membership annotation, and identify the 388 duplicated probes out of 14,168. Build one matrix variant in which each duplicated probe carries the same value in every module it belongs to, and optionally a second variant in which duplicated probes are excluded from module membership. Run the BloodGen3Module Groupcomparison function on each variant with FC = 1.5, pval = 0.1, FDR = TRUE, Group_column = 'Group_test', Test_group = 'Sepsis', Ref_group = 'Control', SummarizedExperiment = TRUE, then extract per-module percentages of genes increased/decreased and diff them across variants. The rationale is that duplicates concern only a small minority of genes, so either handling option yields essentially comparable module-level results; the comparison is a validation of robustness rather than a source of divergence.

## Related tools

- **BloodGen3module** (R package providing Groupcomparison (and Groupcomparisonlimma, gridplot, fingerprintplot) used to compute module-level percent-increase/decrease values on each duplicate-handling variant) — github.com/Drinchai/BloodGen3Module
- **R** (Runtime environment in which the annotated matrices are built, the two duplicate-handling variants are constructed, and the per-module comparison table is computed)
- **ExperimentHub** (Source of the GSE13015 example dataset (object EH5429) used to test the duplicate-gene handling strategies)
- **SummarizedExperiment** (Data container for the annotated expression matrix passed to Groupcomparison with SummarizedExperiment = TRUE)

## Examples

```
Group_df <- Groupcomparison(GSE13015, sample_info = NULL, FC = 1.5, pval = 0.1, FDR = TRUE, Group_column = "Group_test", Test_group = "Sepsis", Ref_group = "Control")
```

## Evaluation signals

- Module-level percent-increase/decrease values differ only minimally between the two duplicate-handling variants, consistent with duplicates concerning only 388 of 14,168 probes
- duplicate_gene_handling_comparison.csv contains one row per module with percent-increase/decrease values for both variants
- Group_df output from Groupcomparison is expressed 'at the module level' as the percentage of genes increased or decreased for each of the 382 modules
- Exactly 388 probes out of 14,168 are flagged as duplicated across modules and handled identically within each variant
- Groupcomparison parameters match the reference run: FC = 1.5, pval = 0.1, FDR = TRUE, Group_column = 'Group_test', Test_group = 'Sepsis', Ref_group = 'Control'

## Limitations

- The duplicate-handling impact assessment applies to the probe-level repertoire built on Illumina HT12 v3 BeadArrays; extending to RNA-seq input requires an explicit gene-level mapping decision
- Either handling option gives essentially comparable results only because duplicates concern a small minority of genes (388/14,168); a repertoire with heavier duplication could be more sensitive
- The BloodGen3 repertoire is not a ground truth — some aggregates are heterogeneous and module functional annotations are subjective — so minimal divergence between variants does not guarantee biological correctness

## Evidence

- [discussion] Duplicate handling has minimal impact on results: "which in our hands shows to have very little impact on the results since they concern only a small minority of genes (388 out of 14,168 probes)"
- [discussion] Repertoire constructed at probe level on Illumina HT12 v3: "the BloodGen3 repertoire was constructed at the probe level (those present on the Illumina HT12 v3 Beadarrays)"
- [readme] Module-level results expressed as percentage of genes increased or decreased: "The results are expressed “at the module level” as the percentage of genes that are increased or decreased for a given module."
- [methods] Annotating expression matrix with module membership is step 1 of the workflow: "1.	Annotating the gene expression data matrix with module membership information."
