---
name: blood-transcriptome-repertoire-construction
description: Use when building or reproducing a fixed whole-blood co-expression module
  repertoire from multiple reference transcriptome datasets, or when partitioning
  modules into higher-level aggregates (A1–A38) for fingerprint grid plots.
when_to_use_negative:
- Analyzing tissues other than blood or non-human species — the BloodGen3 repertoire
  is specifically designed for human blood transcriptome data and other tissues/species
  would require a separate framework
- The goal is per-dataset analysis with custom, data-derived modules rather than reuse/benchmarking
  against a stable fixed module repertoire
- Input is already a module-level fingerprint output (Group_df/Individual_df) rather
  than a raw normalized expression matrix
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0091
tools:
- name: R
  role: Implementation environment for the clustering workflow and companion BloodGen3module
    R package used to run analysis workflows
  repo: github.com/Drinchai/BloodGen3Module
- name: BloodGen3
  role: The fixed repertoire of 382 transcriptional modules produced by this skill
    from co-clustering across 16 datasets / 985 profiles
  repo: github.com/Drinchai/BloodGen3Module
- name: BloodGen3module
  role: R package providing functions to annotate modules and generate fingerprint
    grid plots and heatmaps from the constructed repertoire
  repo: github.com/Drinchai/BloodGen3Module
- name: WGCNA
  role: Whole-genome co-expression network analysis used to define the transcriptional
    modules underlying the repertoire
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-24584-w
    title: Development of a fixed module repertoire for the analysis and interpretation
      of blood transcriptome data
  source_content_hash: sha256:78666df8fa7ef024df498ec62fecd9fd7c1d5de36c5693182e2f7bdd166c802d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BloodGen3
  - BloodGen3module
  - R
  - WGCNA
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# blood-transcriptome-repertoire-construction

## Summary

Constructs a fixed blood transcriptome module repertoire (BloodGen3, 382 modules) by co-clustering genes across 16 reference immunological/physiological datasets (985 profiles), then organizes modules into 38 aggregates (A1–A38) via k-means on their activity profiles for fingerprint grid placement.

## When to use

Use when building or reproducing a fixed whole-blood co-expression module repertoire from multiple reference transcriptome datasets, or when partitioning modules into higher-level aggregates (A1–A38) for fingerprint grid plots.

## When NOT to use

- Analyzing tissues other than blood or non-human species — the BloodGen3 repertoire is specifically designed for human blood transcriptome data and other tissues/species would require a separate framework
- The goal is per-dataset analysis with custom, data-derived modules rather than reuse/benchmarking against a stable fixed module repertoire
- Input is already a module-level fingerprint output (Group_df/Individual_df) rather than a raw normalized expression matrix

## Inputs

- 16 reference blood transcriptome datasets (985 unique blood transcriptome profiles) spanning diverse immunological and physiological states
- Normalized expression data matrix at Illumina HT12 v3 BeadArray probe level (genes/probes as rows, Sample ID as columns, valid Gene Symbol row names)
- Sample annotation table with Sample ID and group columns
- Number of aggregates k = 38 for the k-means partition of the 382 modules

## Outputs

- Fixed repertoire of 382 blood transcriptional modules (BloodGen3)
- Module-to-aggregate assignment table (module ID, aggregate ID A1–A38, Axx.yy grid identifier) e.g. module_aggregate_assignments.csv
- 38 module aggregates, 27 of which contain two or more modules (one fingerprint grid row each)
- Functional annotations and reference transcriptome profiles per module for circle packing plots and fingerprint grid plots

## How to apply

Build the repertoire from 16 input datasets comprising 985 unique blood transcriptome profiles spanning autoimmune, infectious, and inflammatory diseases, cancer, liver transplant recipients, and pregnancy, relying on co-clustering patterns across the sixteen states. Work at the probe level (Illumina HT12 v3 BeadArray probes) and handle the 388 duplicate probes mapping to the same genes across modules, which affect only a small minority of the 14,168 probes. Run whole-genome co-expression network analysis (WGCNA) in R to define the 382 transcriptional modules of the fixed repertoire. Then assemble the modules (rows) x datasets (columns) co-clustering matrix, standardize module activity profiles, and run k-means in R with k = 38 to produce the 38 module aggregates (A1–A38), of which 27 contain two or more modules and occupy one row each of the fingerprint grid. Record module-to-aggregate assignments with Axx.yy grid identifiers (e.g. module_aggregate_assignments.csv) and annotate each of the 382 modules with functional annotations and reference transcriptome profiles for downstream fingerprinting.

## Related tools

- **R** (Implementation environment for the clustering workflow and companion BloodGen3module R package used to run analysis workflows) — github.com/Drinchai/BloodGen3Module
- **BloodGen3** (The fixed repertoire of 382 transcriptional modules produced by this skill from co-clustering across 16 datasets / 985 profiles) — github.com/Drinchai/BloodGen3Module
- **BloodGen3module** (R package providing functions to annotate modules and generate fingerprint grid plots and heatmaps from the constructed repertoire) — github.com/Drinchai/BloodGen3Module
- **WGCNA** (Whole-genome co-expression network analysis used to define the transcriptional modules underlying the repertoire)

## Examples

```
install.packages("devtools"); devtools::install_github("Drinchai/BloodGen3Module")
```

## Evaluation signals

- The repertoire contains exactly 382 transcriptional modules built from co-clustering across 16 datasets and 985 blood transcriptome profiles
- The module-to-aggregate assignment table has exactly 382 assignments distributed over exactly 38 aggregate identifiers (A1–A38)
- Exactly 27 of the 38 aggregates contain two or more modules, each occupying one fingerprint grid row
- Grid identifiers follow the Axx.yy convention derived from aggregate ID and module position within the aggregate
- Duplicate gene handling is documented: 388 out of 14,168 probes map to the same genes across different modules

## Limitations

- The repertoire was constructed at the probe level on Illumina HT12 v3 BeadArrays, which pre-dates RNA sequencing; 388 of 14,168 probes are duplicate genes found in different modules requiring a handling choice
- The module repertoire is not a ground truth — some aggregates are heterogeneous in functionality and heterogeneity also exists within modules; annotation titles confer subjectivity
- The repertoire could be altered by addition or removal of the 16 construction datasets, with more variation expected at the module level than at the aggregate level
- It is specific to human blood; analyses of other tissues or species require separate frameworks

## Evidence

- [abstract] The construction of this repertoire is based on co-clustering patterns observed across sixteen immunological and physiological states encompassing 985 blood transcriptome profiles.: "The construction of this repertoire is based on co-clustering patterns observed across sixteen immunological and physiological states encompassing 985 blood transcriptome profiles."
- [intro] Specifically, 16 input datasets were employed for the construction of the BloodGen3 repertoire, comprising 985 unique blood transcriptome profiles: "Specifically, 16 input datasets were employed for the construction of the BloodGen3 repertoire, comprising 985 unique blood transcriptome profiles"
- [discussion] the two-tiered grouping, first at the module level and at the module aggregate level, is new for this third-generation module repertoire: "the two-tiered grouping, first at the module level and at the module aggregate level, is new for this third-generation module repertoire"
- [discussion] the BloodGen3 repertoire was constructed at the probe level (those present on the Illumina HT12 v3 Beadarrays): "the BloodGen3 repertoire was constructed at the probe level (those present on the Illumina HT12 v3 Beadarrays)"
- [discussion] they concern only a small minority of genes (388 out of 14,168 probes): "they concern only a small minority of genes (388 out of 14,168 probes)"
- [methods] devtools::install_github("Drinchai/BloodGen3Module"): "devtools::install_github("Drinchai/BloodGen3Module")"
