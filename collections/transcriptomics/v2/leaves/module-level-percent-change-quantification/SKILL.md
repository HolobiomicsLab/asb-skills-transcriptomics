---
name: module-level-percent-change-quantification
description: Use when you have a normalized (non-log2) blood expression matrix plus
  sample annotation and need per-module percent-increase/decrease values for fingerprinting,
  or when testing whether alternate gene-to-module assignments (e.g., duplicate-probe
  handling) change module-level results.
when_to_use_negative:
- The input is log2-transformed expression data — the README requires data.matrix
  that 'should not be Log2 transformed'
- The data are not human blood transcriptome profiles — the BloodGen3 repertoire is
  specifically designed for human blood, and other tissues or species would require
  a separate framework
- The goal is transcriptome-wide differential gene lists rather than module-level
  fingerprint summaries — a standard DE analysis is more appropriate
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: BloodGen3module
  role: R package providing Groupcomparison (and Groupcomparisonlimma) to compute
    module-level percentages of genes increased/decreased and gridplot/fingerprintplot
    for visualization
  repo: github.com/Drinchai/BloodGen3Module
- name: BloodGen3
  role: Fixed repertoire of 382 transcriptional modules supplying the module membership
    annotation used to assign genes to modules
- name: ExperimentHub
  role: Source of the GSE13015 example expression dataset (object EH5429) used for
    the group comparison analysis
- name: R
  role: Execution environment for the BloodGen3module workflow (installed via devtools::install_github
    or BiocManager)
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-24584-w
    title: Development of a fixed module repertoire for the analysis and interpretation
      of blood transcriptome data
  source_content_hash: sha256:cdb5dc3d6918d554ada30dcb6bb0f2d224522aac5edfb329e3108cd71cbbeb42
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BloodGen3
  - BloodGen3module
  - ExperimentHub
  - R
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# module-level-percent-change-quantification

## Summary

Quantifies group-comparison results in the BloodGen3 fixed module repertoire by expressing them at the module level as the percentage of constitutive genes that are increased or decreased, using the BloodGen3module R package's Groupcomparison function with fold-change and FDR cutoffs.

## When to use

Use when you have a normalized (non-log2) blood expression matrix plus sample annotation and need per-module percent-increase/decrease values for fingerprinting, or when testing whether alternate gene-to-module assignments (e.g., duplicate-probe handling) change module-level results.

## When NOT to use

- The input is log2-transformed expression data — the README requires data.matrix that 'should not be Log2 transformed'
- The data are not human blood transcriptome profiles — the BloodGen3 repertoire is specifically designed for human blood, and other tissues or species would require a separate framework
- The goal is transcriptome-wide differential gene lists rather than module-level fingerprint summaries — a standard DE analysis is more appropriate

## Inputs

- Normalized expression data matrix (not Log2 transformed), genes as rows, sample IDs as columns, valid Gene Symbols as row names (e.g., SummarizedExperiment EH5429 from GSE13015)
- Sample annotation dataframe with a Sample ID column matching data.matrix and a group column (e.g., Group_test with values Sepsis/Control)
- BloodGen3 module membership annotation (382 modules)

## Outputs

- Group_df dataframe with per-module percentage of genes increased/decreased
- Duplicate-gene handling comparison table (e.g., duplicate_gene_handling_comparison.csv) with one row per module and percent-increase/decrease values for both variants
- Optional fingerprint grid plot PDF via gridplot at a chosen visualization cutoff (default 15%)

## How to apply

Annotate the expression matrix with BloodGen3 module membership (e.g., load the GSE13015 example object EH5429 via ExperimentHub). Run the BloodGen3module Groupcomparison function on the annotated matrix with concrete cutoffs — FC = 1.5, pval = 0.1, FDR = TRUE (BH method), Group_column = 'Group_test', Test_group = 'Sepsis', Ref_group = 'Control' — or use Groupcomparisonlimma for a limma-based test. The function first annotates module membership, runs statistical tests (t-test or limma) to determine the proportion of constitutive genes differentially expressed per module, then expresses results as the percentage of genes increased or decreased per module. For duplicate-gene sensitivity, build two variants of the annotated matrix — one where each of the 388 duplicated probes is assigned to every module it belongs to, and one where duplicates are excluded — run Groupcomparison on both, and compute per-module and global differences. Results can be visualized with gridplot (default visualization cutoff 15%) or exported as tables such as duplicate_gene_handling_comparison.csv.

## Related tools

- **BloodGen3module** (R package providing Groupcomparison (and Groupcomparisonlimma) to compute module-level percentages of genes increased/decreased and gridplot/fingerprintplot for visualization) — github.com/Drinchai/BloodGen3Module
- **BloodGen3** (Fixed repertoire of 382 transcriptional modules supplying the module membership annotation used to assign genes to modules)
- **ExperimentHub** (Source of the GSE13015 example expression dataset (object EH5429) used for the group comparison analysis)
- **R** (Execution environment for the BloodGen3module workflow (installed via devtools::install_github or BiocManager))

## Examples

```
Group_df <- Groupcomparison(GSE13015, sample_info = NULL, FC = 1.5, pval = 0.1, FDR = TRUE, Group_column = 'Group_test', Test_group = 'Sepsis', Ref_group = 'Control')
```

## Evaluation signals

- Group_df contains one row per module (up to 382 modules) with the percentage of genes increased and decreased computed at FC = 1.5 and FDR = TRUE (BH method)
- Duplicate-gene sensitivity check: module-level percentages differ only minimally between variant A (388 duplicated probes assigned to every module) and variant B (duplicates excluded), consistent with duplicates concerning only 388 out of 14,168 probes
- duplicate_gene_handling_comparison.csv contains one row per module with percent-increase/decrease values from both handling options
- Input matrix validation: genes as rows with valid Gene Symbol row names, expression values not Log2 transformed, sample IDs exactly matching the sample_info columns
- Visualized fingerprint respects the chosen cutoff (default 15%): only modules exceeding the percentage cutoff appear on the gridplot

## Limitations

- Duplicate genes (probes mapping to more than one module; 388 out of 14,168 probes) require a handling choice, though both options yield essentially comparable module-level results
- The repertoire was constructed at the probe level on Illumina HT12 v3 BeadArrays, which pre-dates RNA sequencing; the duplicate-handling strategies are the sanctioned route for gene-level RNA-seq input
- The BloodGen3 module repertoire is not a ground truth — some aggregates and modules are heterogeneous in functionality
- The default visualization cutoff of 15% affects which modules appear in grid plots, but can be changed from 0 to 100%

## Evidence

- [methods] Expressing results “at the module level” as the percentage of genes that are increased or decreased: "Expressing results “at the module level” as the percentage of genes that are increased or decreased."
- [discussion] Disregarding duplicate genes has little impact: "which in our hands shows to have very little impact on the results since they concern only a small minority of genes (388 out of 14,168 probes)"
- [readme] README example Groupcomparison call with parameters: "Group_df <- Groupcomparison(GSE13015,
                            sample_info = NULL,
                            FC = 1.5,
                            pval = 0.1 ,
                            FDR ="
- [readme] README input matrix requirements: [evidence span withheld — anchor status: external_doc]
