---
name: module-aggregate-identifier-assignment
description: 'Use when building or reproducing the two-tier BloodGen3 repertoire structure:
  you have module-level activity/co-clustering profiles for the 382 modules and need
  aggregate IDs (A1–A38) and grid placement for fingerprint grid or heatmap plots.'
when_to_use_negative:
- Your expression data is not human blood transcriptome data — the BloodGen3 repertoire
  is specifically designed for human blood, and other tissues or species would require
  a separate framework
- You need data-driven de novo gene clustering (e.g. WGCNA on your own dataset) rather
  than assignment of modules to the fixed, pre-defined aggregate structure
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: BloodGen3
  role: Source of the fixed 382-module repertoire whose activity profiles across 16
    reference datasets are clustered into aggregates
  repo: https://github.com/Drinchai/BloodGen3Module
- name: R
  role: Environment in which k-means clustering with k = 38 is run on standardized
    module activity profiles
- name: BloodGen3module
  role: R package consuming the Axx.yy aggregate/grid identifiers to generate fingerprint
    grid plots and heatmaps (e.g. Aggregate = 'A28')
  repo: https://github.com/Drinchai/BloodGen3Module
- name: WGCNA
  role: Co-expression network framework underpinning module construction prior to
    two-tier aggregation
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-24584-w
    title: Development of a fixed module repertoire for the analysis and interpretation
      of blood transcriptome data
  source_content_hash: sha256:a4cd652bde633f133c4564aa52d6db71301dd7e1da478a04c52bdf7e8cb884af
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Drinchai/BloodGen3Module
  tools:
  - BloodGen3
  - BloodGen3module
  - R
  - WGCNA
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# module-aggregate-identifier-assignment

## Summary

Partitions the 382 BloodGen3 transcriptional modules into 38 higher-level module aggregates (A1–A38) by k-means clustering of module activity profiles across the 16 reference datasets (985 blood transcriptome profiles), and derives the Axx.yy grid identifiers used for fingerprint plots.

## When to use

Use when building or reproducing the two-tier BloodGen3 repertoire structure: you have module-level activity/co-clustering profiles for the 382 modules and need aggregate IDs (A1–A38) and grid placement for fingerprint grid or heatmap plots.

## When NOT to use

- Your expression data is not human blood transcriptome data — the BloodGen3 repertoire is specifically designed for human blood, and other tissues or species would require a separate framework
- You need data-driven de novo gene clustering (e.g. WGCNA on your own dataset) rather than assignment of modules to the fixed, pre-defined aggregate structure

## Inputs

- Module activity / co-clustering matrix: 382 BloodGen3 modules (rows) x 16 reference datasets (columns) built from 985 blood transcriptome profiles
- List of the 382 module identifiers from the fixed BloodGen3 repertoire

## Outputs

- module_aggregate_assignments.csv with columns: module ID, aggregate ID (A1–A38), grid identifier (Axx.yy)
- Mapping of the 27 aggregates that contain two or more modules, each occupying one row of the fingerprint grid
- Erythroid-type aggregate groups used downstream (e.g. A36, A37, A38)

## How to apply

Load the modules (rows) x datasets (columns) activity/co-clustering matrix computed across the 16 reference datasets spanning 985 blood transcriptome profiles, and standardize module profiles so that distances reflect co-variation across immunological and physiological states. Run k-means clustering in R with k = 38 to partition the 382 modules into 38 aggregates. Label each cluster with its aggregate identifier A1–A38 and derive the Axx.yy grid identifiers used by the gridplot/fingerprintplot functions, recording which aggregates are singletons versus the 27 aggregates containing two or more modules. Write the module-to-aggregate assignment table (module ID, aggregate ID, grid identifier) to CSV. This two-tiered grouping — first at the module level and then at the aggregate level — is new to the third-generation repertoire and organizes each multi-module aggregate as one row of the fingerprint grid.

## Related tools

- **BloodGen3** (Source of the fixed 382-module repertoire whose activity profiles across 16 reference datasets are clustered into aggregates) — https://github.com/Drinchai/BloodGen3Module
- **R** (Environment in which k-means clustering with k = 38 is run on standardized module activity profiles)
- **BloodGen3module** (R package consuming the Axx.yy aggregate/grid identifiers to generate fingerprint grid plots and heatmaps (e.g. Aggregate = 'A28')) — https://github.com/Drinchai/BloodGen3Module
- **WGCNA** (Co-expression network framework underpinning module construction prior to two-tier aggregation)

## Examples

```
set.seed(1); km <- kmeans(scale(module_profiles_382x16), centers = 38, nstart = 25); assignments <- data.frame(module_id = rownames(module_profiles_382x16), aggregate_id = paste0('A', km$cluster)) # then write.csv(assignments, 'module_aggregate_assignments.csv')
```

## Evaluation signals

- module_aggregate_assignments.csv contains exactly 382 module assignments distributed over exactly 38 aggregate identifiers (A1–A38)
- Exactly 27 of the 38 aggregates group two or more modules; the remaining 11 are single-module aggregates
- Every grid identifier follows the Axx.yy format and is consistent with the module-to-aggregate assignment table
- Aggregates with multiple modules each map to exactly one row of the fingerprint grid produced by gridplot
- Known functionally coherent aggregates are recovered (e.g. the circulating erythroid cell signature A36, A37, A38)

## Limitations

- The module repertoire and its aggregates are not a ground truth: some aggregates are clearly not homogeneous in terms of functionality, and heterogeneity also exists within modules
- Adding or removing datasets used to construct the repertoire could alter the partitioning; more variation is expected at the module level than at the aggregate level
- The repertoire was built at the probe level on Illumina HT12 v3 BeadArrays; 388 out of 14,168 probes are duplicate genes found in different modules, requiring a handling choice
- Functional annotation of aggregates is expected to continue evolving while the fixed framework stays unchanged

## Evidence

- [discussion] the two-tiered grouping, first at the module level and at the module aggregate level, is new for this third-generation module repertoire: "the two-tiered grouping, first at the module level and at the module aggregate level, is new for this third-generation module repertoire"
- [abstract] co-clustering across 16 states underpinning the repertoire: "The construction of this repertoire is based on co-clustering patterns observed across sixteen immunological and physiological states encompassing 985 blood transcriptome profiles."
- [readme] aggregate parameter in fingerprintplot README: [evidence span withheld — anchor status: external_doc]
- [discussion] gridplot aggregate rows: "modules within a given aggregate (=row) are grouped first according to functional annotations and then among the different annotations, from left to right according to alphabetical order"
