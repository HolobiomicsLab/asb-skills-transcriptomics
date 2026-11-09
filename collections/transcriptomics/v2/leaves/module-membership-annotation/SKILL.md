---
name: module-membership-annotation
description: Use when a normalized blood transcriptome expression matrix (non-log2,
  gene symbols as row names, e.g. Illumina HT12 v3 probes) must be mapped to the fixed
  382-module BloodGen3 repertoire before module-level testing or fingerprint plotting.
when_to_use_negative:
- The expression matrix is log2 transformed — the README specifies the matrix 'should
  not be Log2 transformed'
- Analyzing tissues or species other than human blood — the BloodGen3 repertoire is
  specifically designed for human blood transcriptome data
- The task is de novo co-expression module discovery on new datasets — this skill
  only applies the fixed published repertoire, it does not rebuild it
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: BloodGen3Module
  role: R package providing the fixed 382-module membership annotations and the downstream
    functions (Groupcomparison, Individualcomparison, gridplot, fingerprintplot) that
    consume the annotated matrix
  repo: https://github.com/Drinchai/BloodGen3Module
- name: BloodGen3
  role: The fixed module repertoire itself (382 modules, 14,168 transcripts) whose
    membership table is used for annotation
  repo: https://github.com/Drinchai/BloodGen3Module
- name: ExperimentHub
  role: Used to retrieve example reference cohort input files (e.g. GSE13015 via EH5429)
    for the annotation workflow
- name: WGCNA
  role: Whole-genome co-expression network analysis method underlying the construction
    of the fixed repertoire being applied
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-24584-w
    title: Development of a fixed module repertoire for the analysis and interpretation
      of blood transcriptome data
  source_content_hash: sha256:6687b70fb421905580e4ecfbd31b93b5b76f60a2409ccd128a30642c868041ab
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Drinchai/BloodGen3Module
  tools:
  - BloodGen3
  - BloodGen3Module
  - ExperimentHub
  - R
  - WGCNA
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# module-membership-annotation

## Summary

Annotates a normalized blood transcriptome expression matrix with fixed BloodGen3 module repertoire membership, mapping each transcript (gene symbol/probe) to its two-tiered module and aggregate identifiers so downstream module-level group or individual comparisons can be run.

## When to use

Use when a normalized blood transcriptome expression matrix (non-log2, gene symbols as row names, e.g. Illumina HT12 v3 probes) must be mapped to the fixed 382-module BloodGen3 repertoire before module-level testing or fingerprint plotting.

## When NOT to use

- The expression matrix is log2 transformed — the README specifies the matrix 'should not be Log2 transformed'
- Analyzing tissues or species other than human blood — the BloodGen3 repertoire is specifically designed for human blood transcriptome data
- The task is de novo co-expression module discovery on new datasets — this skill only applies the fixed published repertoire, it does not rebuild it

## Inputs

- Normalized expression data matrix (genes as rows, Sample ID columns, non-log2 transformed, valid Gene Symbol row names; e.g. Illumina HT12 v3 BeadArray probe-level data)
- Sample annotation dataframe with Sample ID and group columns
- BloodGen3 fixed module membership table mapping transcripts to 382 modules and their aggregates (two-tiered Gen3 grouping)

## Outputs

- Expression data matrix annotated with module membership information for downstream module-level analysis
- Module-level results expressing the percentage of genes increased or decreased per module (e.g. Group_df or Individual_df dataframes)

## How to apply

Install the BloodGen3Module R package (devtools::install_github('Drinchai/BloodGen3Module') or via Bioconductor) and load the module membership table that maps every transcript to its module and module aggregate (the two-tiered Gen3 grouping). Verify the expression matrix is normalized, NOT log2 transformed, arranged with genes as rows and Sample IDs as columns, and that row names are valid Gene Symbols. Annotate each matrix row with its module membership (382 modules, 14,168 transcripts total); note that 388 of the 14,168 probes are duplicate genes found in different modules, so duplicates require an explicit handling decision (the authors report this has little impact since they concern only a small minority of genes). The annotated matrix becomes the input to Groupcomparison/Groupcomparisonlimma (t-test or limma, e.g. FC=1.5, FDR=TRUE, pval=0.1) and Individualcomparison. Validation: the repertoire must contain exactly 382 modules spanning 14,168 transcripts, matching the published BloodGen3 module list built from 16 datasets (985 profiles).

## Related tools

- **BloodGen3Module** (R package providing the fixed 382-module membership annotations and the downstream functions (Groupcomparison, Individualcomparison, gridplot, fingerprintplot) that consume the annotated matrix) — https://github.com/Drinchai/BloodGen3Module
- **BloodGen3** (The fixed module repertoire itself (382 modules, 14,168 transcripts) whose membership table is used for annotation) — https://github.com/Drinchai/BloodGen3Module
- **ExperimentHub** (Used to retrieve example reference cohort input files (e.g. GSE13015 via EH5429) for the annotation workflow)
- **WGCNA** (Whole-genome co-expression network analysis method underlying the construction of the fixed repertoire being applied)

## Examples

```
install.packages("devtools"); devtools::install_github("Drinchai/BloodGen3Module"); library(BloodGen3Module); Group_df <- Groupcomparison(GSE13015, sample_info=NULL, FC=1.5, pval=0.1, FDR=TRUE, Group_column="Group_test", Test_group="Sepsis", Ref_group="Control")
```

## Evaluation signals

- The annotated repertoire contains exactly 382 modules spanning 14,168 transcripts (388 probes are duplicate genes across modules)
- Expression matrix passed in is normalized and NOT log2 transformed, with valid Gene Symbol row names and Sample ID columns matching the sample annotation
- Module results are expressible 'at the module level' as a percentage of genes increased or decreased for every annotated module
- Downstream Groupcomparison (t-test or limma, FC=1.5, FDR=TRUE, pval=0.1 in the README example) runs without unmatched transcripts and returns Group_df/Group_limma dataframes
- Fingerprint grid visualization with default cutoff of 15% produces a grid/heatmap consistent with the published repertoire layout (two-tiered module/aggregate grouping)

## Limitations

- The repertoire was constructed at the probe level on Illumina HT12 v3 BeadArrays; 388 out of 14,168 probes are duplicate genes found in different modules, requiring an explicit handling choice
- The BloodGen3 repertoire is not a ground truth — some aggregates are heterogeneous in functionality and functional annotation titles confer subjectivity
- The module repertoire could be altered by adding or removing datasets used for its construction; it is specific to human blood, and other tissues or species require separate frameworks

## Evidence

- [readme] First step of module analysis: "Annotating the gene expression data matrix with module membership information."
- [readme] Input matrix requirements: [evidence span withheld — anchor status: external_doc]
- [readme] Module-level expression of results: "The results are expressed “at the module level” as the percentage of genes that are increased or decreased for a given module."
- [discussion] Two-tiered grouping: "the two-tiered grouping, first at the module level and at the module aggregate level, is new for this third-generation module repertoire"
- [discussion] Duplicate probes: "the BloodGen3 repertoire was constructed at the probe level (those present on the Illumina HT12 v3 Beadarrays)"
