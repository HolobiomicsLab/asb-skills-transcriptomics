---
name: sensitivity-analysis-of-gene-handling-options
description: Use when porting the BloodGen3 probe-level repertoire to gene-level inputs
  (e.g. RNA-seq) or when deciding whether duplicate genes mapping to multiple modules
  must be resolved before running Groupcomparison fingerprints.
when_to_use_negative:
- Analyzing tissues or species other than human blood — the BloodGen3 repertoire is
  specifically designed for human blood transcriptome profiles
- Log2-transformed expression input — the README requires data.matrix that 'should
  not be Log2 transformed'
- Repertoire construction or module re-definition tasks — this skill only evaluates
  downstream sensitivity of module-level results, not rebuilding the 382-module repertoire
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: BloodGen3module
  role: R package providing the Groupcomparison function used to compute module-level
    percent-increase/decrease for each duplicate-handling variant
  repo: github.com/Drinchai/BloodGen3Module
- name: R
  role: Runtime environment for loading data, building the two annotated matrix variants,
    and comparing outputs
- name: ExperimentHub
  role: Downloads the GSE13015 example dataset (object EH5429) used for the comparison
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
  source_content_hash: sha256:8e512fa14386ee05f58b45bb45acb17d88604e3b7bb2bc451f435832b5dc24d7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BloodGen3
  - BloodGen3module
  - ExperimentHub
  - R
  - SummarizedExperiment
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# sensitivity-analysis-of-gene-handling-options

## Summary

A sensitivity analysis that quantifies how sanctioned duplicate-gene resolution strategies (assigning a duplicated probe to every module vs. disregarding duplicates) affect BloodGen3 module-level percent-increase/decrease fingerprints, exploiting the fact that only 388 of 14,168 Illumina HT12 v3 probes are duplicated across modules.

## When to use

Use when porting the BloodGen3 probe-level repertoire to gene-level inputs (e.g. RNA-seq) or when deciding whether duplicate genes mapping to multiple modules must be resolved before running Groupcomparison fingerprints.

## When NOT to use

- Analyzing tissues or species other than human blood — the BloodGen3 repertoire is specifically designed for human blood transcriptome profiles
- Log2-transformed expression input — the README requires data.matrix that 'should not be Log2 transformed'
- Repertoire construction or module re-definition tasks — this skill only evaluates downstream sensitivity of module-level results, not rebuilding the 382-module repertoire

## Inputs

- Normalized (non-log2) expression matrix as a SummarizedExperiment, e.g. GSE13015 object EH5429 from ExperimentHub, with valid gene symbols as row names
- BloodGen3 module membership annotation mapping probes to modules and aggregates
- Sample annotation with Sample ID and group columns (Group_test: Sepsis vs Control)

## Outputs

- duplicate_gene_handling_comparison.csv containing one row per module with percent-increase/decrease values for both duplicate-handling variants
- Per-module and global difference metrics between variant A (duplicates in all modules) and variant B (duplicates excluded)

## How to apply

Load the GSE13015 example dataset (object EH5429) via ExperimentHub together with the BloodGen3 module membership annotation, then build two annotated expression-matrix variants: variant A assigns each of the 388 duplicated probes to every module it belongs to, while variant B excludes those 388 duplicated probes from module membership. Run the BloodGen3Module Groupcomparison function on both variants with FC = 1.5, pval = 0.1, FDR = TRUE, Group_column = 'Group_test', Test_group = 'Sepsis', Ref_group = 'Control', SummarizedExperiment = TRUE. Extract per-module percentages of genes increased/decreased from both runs, compute per-module and global differences between the two handling options, and write the combined table (one row per module, both variants' values) to duplicate_gene_handling_comparison.csv. The rationale is that duplicates are a small minority of genes, so the comparison empirically validates which handling option can be adopted for RNA-seq workflows.

## Related tools

- **BloodGen3module** (R package providing the Groupcomparison function used to compute module-level percent-increase/decrease for each duplicate-handling variant) — github.com/Drinchai/BloodGen3Module
- **R** (Runtime environment for loading data, building the two annotated matrix variants, and comparing outputs)
- **ExperimentHub** (Downloads the GSE13015 example dataset (object EH5429) used for the comparison)
- **SummarizedExperiment** (Data container for the annotated expression matrix passed to Groupcomparison with SummarizedExperiment = TRUE)

## Examples

```
Group_df <- Groupcomparison(GSE13015, sample_info = NULL, FC = 1.5, pval = 0.1, FDR = TRUE, Group_column = "Group_test", Test_group = "Sepsis", Ref_group = "Control")
```

## Evaluation signals

- Module-level percent-increase/decrease values differ only minimally between the two handling options, consistent with duplicates concerning only 388 of 14,168 probes
- duplicate_gene_handling_comparison.csv contains exactly one row per module with percent-increase/decrease values for both variants
- Both Groupcomparison runs use identical parameters (FC = 1.5, pval = 0.1, FDR = TRUE, Group_column = 'Group_test', Test_group = 'Sepsis', Ref_group = 'Control')
- Variant B confirms exactly 388 duplicated probes are excluded from module membership out of 14,168 total probes
- Global difference metrics between variants are near zero for the vast majority of modules

## Limitations

- The BloodGen3 repertoire was constructed at the probe level (Illumina HT12 v3 Beadarrays), which pre-dates RNA sequencing; duplicate probes map to the same genes across modules and require a handling choice
- The repertoire is specific to human blood transcriptome data; other tissues or species require separate frameworks
- The module repertoire is not a ground truth — some aggregates are heterogeneous, so near-identical fingerprints between variants do not certify biological correctness

## Evidence

- [discussion] Disregarding duplicate genes has minimal impact: "which in our hands shows to have very little impact on the results since they concern only a small minority of genes (388 out of 14,168 probes)"
- [discussion] Repertoire built at probe level: "the BloodGen3 repertoire was constructed at the probe level (those present on the Illumina HT12 v3 Beadarrays)"
- [readme] Groupcomparison performs group comparisons: "The **Groupcomparison** function will perform group comparison analyses."
- [methods] Module-level results expressed as percentages: "Expressing results “at the module level” as the percentage of genes that are increased or decreased."
