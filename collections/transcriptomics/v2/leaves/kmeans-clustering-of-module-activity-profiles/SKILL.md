---
name: kmeans-clustering-of-module-activity-profiles
description: 'Use when you need to reproduce or re-derive the module-aggregate hierarchy
  of the BloodGen3 repertoire: you have a 382-module x 16-dataset activity/co-clustering
  matrix and need A1–A38 aggregate labels and grid identifiers for fingerprint grid
  plots.'
when_to_use_negative:
- Your data are not human blood transcriptome profiles — the article states the BloodGen3
  repertoire is specifically designed for human blood and that analyses of other tissues
  or species would require separate frameworks
- You already have a fixed module-to-aggregate assignment and only need to run downstream
  analyses (Groupcomparison, gridplot) — reuse the existing Axx.yy labels rather than
  re-clustering
- You need a validated ground-truth functional classification — the article cautions
  the repertoire is not ground truth and some aggregates are not homogeneous in functionality
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: R
  role: Environment in which k-means clustering is run on the standardized module
    activity profiles to produce the 38 aggregates
  repo: https://github.com/Drinchai/BloodGen3Module
- name: BloodGen3module
  role: R package whose fingerprint grid plots consume the module-to-aggregate assignment
    and Axx.yy grid identifiers produced by this clustering
  repo: https://github.com/Drinchai/BloodGen3Module
- name: BloodGen3
  role: Fixed repertoire of 382 transcriptional modules whose co-clustering-based
    activity profiles are the clustering input
- name: WGCNA
  role: Whole-genome co-expression network analysis used in the broader construction
    of the module repertoire from which activity profiles derive
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-24584-w
    title: Development of a fixed module repertoire for the analysis and interpretation
      of blood transcriptome data
  source_content_hash: sha256:2a76447f1f3a34da74dc456623427d710ef378c09d7f21708a4c5e54477e62b7
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# k-means clustering of module activity profiles

## Summary

Partitions the 382 BloodGen3 blood transcriptome modules into 38 higher-level module aggregates (A1–A38) by running k-means (k = 38) on the standardized modules-by-datasets co-clustering activity profiles computed across 16 reference datasets (985 blood transcriptome profiles). This two-tiered grouping underlies the Axx.yy grid identifiers used on fingerprint grid plots.

## When to use

Use when you need to reproduce or re-derive the module-aggregate hierarchy of the BloodGen3 repertoire: you have a 382-module x 16-dataset activity/co-clustering matrix and need A1–A38 aggregate labels and grid identifiers for fingerprint grid plots.

## When NOT to use

- Your data are not human blood transcriptome profiles — the article states the BloodGen3 repertoire is specifically designed for human blood and that analyses of other tissues or species would require separate frameworks
- You already have a fixed module-to-aggregate assignment and only need to run downstream analyses (Groupcomparison, gridplot) — reuse the existing Axx.yy labels rather than re-clustering
- You need a validated ground-truth functional classification — the article cautions the repertoire is not ground truth and some aggregates are not homogeneous in functionality

## Inputs

- Module activity profile matrix: 382 BloodGen3 modules (rows) x 16 reference datasets (columns) co-clustering profiles
- R statistical environment with k-means implementation (stats::kmeans)
- Module membership annotation for the 382-module BloodGen3 repertoire

## Outputs

- module_aggregate_assignments.csv with columns: module ID, aggregate ID (A1–A38), grid identifier (Axx.yy)
- Aggregate-level grouping showing 38 aggregates, of which 27 contain two or more modules (one fingerprint-grid row each)

## How to apply

Load the module activity profiles for all 382 BloodGen3 modules computed across the 16 reference datasets that comprise 985 blood transcriptome profiles spanning autoimmune, infectious and inflammatory diseases, cancer, liver transplant recipients, and pregnancy. Assemble the modules (rows) x datasets (columns) co-clustering matrix and standardize module profiles so that distances reflect co-variation across the immunological and physiological states. Run k-means in R with k = 38, then label each cluster with its aggregate identifier A1–A38 and derive the Axx.yy grid identifiers used by the gridplot function, recording which aggregates contain two or more modules (27 do; each occupies one row of the fingerprint grid). Write the module-to-aggregate assignment table (module ID, aggregate ID, grid identifier) and validate that it contains exactly 382 assignments over exactly 38 aggregate identifiers. Rationale: the article states this two-tiered grouping — module level then aggregate level — is new to this third-generation repertoire, and aggregates are expected to be functionally coherent, so grouping by similarity of activity profiles supports biological interpretation on fingerprint grids.

## Related tools

- **R** (Environment in which k-means clustering is run on the standardized module activity profiles to produce the 38 aggregates) — https://github.com/Drinchai/BloodGen3Module
- **BloodGen3module** (R package whose fingerprint grid plots consume the module-to-aggregate assignment and Axx.yy grid identifiers produced by this clustering) — https://github.com/Drinchai/BloodGen3Module
- **BloodGen3** (Fixed repertoire of 382 transcriptional modules whose co-clustering-based activity profiles are the clustering input)
- **WGCNA** (Whole-genome co-expression network analysis used in the broader construction of the module repertoire from which activity profiles derive)

## Examples

```
set.seed(1); km <- kmeans(scale(module_activity_profiles), centers = 38, nstart = 25); aggregate_id <- paste0("A", km$cluster)
```

## Evaluation signals

- Assignment table contains exactly 382 module-to-aggregate rows
- Number of distinct aggregate identifiers equals exactly 38 (A1–A38)
- Exactly 27 of the 38 aggregates contain two or more modules, matching the article's fingerprint grid description
- Grid identifiers follow the Axx.yy format and each multi-module aggregate occupies one row of the fingerprint grid produced by gridplot()
- Cluster membership is stable with respect to the standardized activity profiles (co-variation distances across the 16 immunological/physiological states), and aggregates remain meaningful given the article's caveat that annotation titles confer subjectivity

## Limitations

- The module repertoire is not a ground truth: some aggregates are clearly not homogeneous in terms of functionality, and heterogeneity also exists within modules
- Repertoire (and hence the aggregate partition) could be altered by the addition or removal of datasets used for its construction, with more variation expected at the module level than the aggregate level
- The repertoire was constructed at the probe level (Illumina HT12 v3 BeadArrays, pre-dating RNA-seq); 388 of 14,168 probes are duplicate genes found in different modules, requiring a handling choice

## Evidence

- [discussion] the two-tiered grouping, first at the module level and at the module aggregate level, is new for this third-generation module repertoire: "the two-tiered grouping, first at the module level and at the module aggregate level, is new for this third-generation module repertoire"
- [abstract] The construction of this repertoire is based on co-clustering patterns observed across sixteen immunological and physiological states encompassing 985 blood transcriptome profiles.: "The construction of this repertoire is based on co-clustering patterns observed across sixteen immunological and physiological states encompassing 985 blood transcriptome profiles."
- [discussion] modules within a given aggregate (=row) are grouped first according to functional annotations and then among the different annotations, from left to right according to alphabetical order: "modules within a given aggregate (=row) are grouped first according to functional annotations and then among the different annotations, from left to right according to alphabetical order"
- [discussion] our module repertoire is not meant to constitute a ground truth. Indeed, some aggregates are clearly not homogeneous in terms of functionality: "our module repertoire is not meant to constitute a ground truth. Indeed, some aggregates are clearly not homogeneous in terms of functionality"
- [readme] The **gridplot** function will generate a grid plot as a PDF file.: "The **gridplot** function will generate a grid plot as a PDF file."
