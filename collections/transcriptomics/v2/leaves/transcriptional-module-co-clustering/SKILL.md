---
name: transcriptional-module-co-clustering
description: Use when a fixed module repertoire needs a two-tier organization — grouping
  modules into aggregates based on similarity of activity profiles across the reference
  datasets that were used to build the repertoire.
when_to_use_negative:
- The expression data are patient samples to be scored against an existing repertoire
  (use Groupcomparison/Individualcomparison, not re-clustering of modules)
- The data are not human blood transcriptomes — the BloodGen3 repertoire is specific
  to human blood, and analyses of other tissues or species would require a separate
  framework
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
tools:
- name: BloodGen3
  role: Fixed repertoire of 382 transcriptional modules whose activity profiles across
    the 16 reference datasets are clustered into aggregates
  repo: https://github.com/Drinchai/BloodGen3Module
- name: R
  role: Language in which the k-means clustering (k = 38) and assignment-table generation
    are performed
- name: BloodGen3module (R package)
  role: R package whose gridplot/fingerprintplot functions consume the module-aggregate
    assignment (Axx.yy identifiers) to render fingerprint grid and heatmap plots
  repo: github.com/Drinchai/BloodGen3Module
- name: WGCNA
  role: Co-expression network analysis framework underpinning construction of the
    module repertoire whose profiles are clustered
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-24584-w
    title: Development of a fixed module repertoire for the analysis and interpretation
      of blood transcriptome data
  source_content_hash: sha256:c441b8d10bc7ada578dad01250dd375ab3be7b049f52f25b59061f73c0bca895
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Drinchai/BloodGen3Module
  tools:
  - BloodGen3
  - BloodGen3module (R package)
  - R
  - WGCNA
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# transcriptional-module-co-clustering

## Summary

Partition a fixed repertoire of transcriptional modules (here the 382 BloodGen3 modules built from 16 datasets / 985 blood transcriptome profiles) into higher-level module aggregates by clustering module activity profiles across reference datasets, yielding 38 aggregates (A1–A38) used to structure fingerprint grid plots.

## When to use

Use when a fixed module repertoire needs a two-tier organization — grouping modules into aggregates based on similarity of activity profiles across the reference datasets that were used to build the repertoire.

## When NOT to use

- The expression data are patient samples to be scored against an existing repertoire (use Groupcomparison/Individualcomparison, not re-clustering of modules)
- The data are not human blood transcriptomes — the BloodGen3 repertoire is specific to human blood, and analyses of other tissues or species would require a separate framework

## Inputs

- Module activity profiles: 382 BloodGen3 modules x 16 reference datasets co-clustering matrix (built from 985 blood transcriptome profiles)
- R environment with BloodGen3module and WGCNA available

## Outputs

- Module-to-aggregate assignment table (module ID, aggregate ID A1–A38, Axx.yy grid identifier), e.g. module_aggregate_assignments.csv
- Mapping of the 27 aggregates containing two or more modules, each defining one row of the fingerprint grid

## How to apply

Load the module activity profiles for all 382 modules computed across the 16 reference datasets comprising 985 blood transcriptome profiles, and assemble the modules (rows) x datasets (columns) co-clustering matrix. Standardize module profiles so that distances reflect co-variation across immunological and physiological states spanning autoimmune, infectious and inflammatory diseases, cancer, liver transplant recipients, and pregnancy. Run k-means clustering in R on these profiles with k = 38 to obtain the 38 aggregates, then label each cluster A1–A38 and derive the Axx.yy grid identifiers used in fingerprint grid plots. Record which aggregates contain two or more modules — 27 of the 38 aggregates group two or more modules, and each of these occupies one row of the fingerprint grid, with modules within a row ordered first by functional annotation and then alphabetically. Write the module-to-aggregate assignment table (module ID, aggregate ID, grid identifier), e.g. to module_aggregate_assignments.csv, for downstream use by the BloodGen3module gridplot/fingerprintplot functions.

## Related tools

- **BloodGen3** (Fixed repertoire of 382 transcriptional modules whose activity profiles across the 16 reference datasets are clustered into aggregates) — https://github.com/Drinchai/BloodGen3Module
- **R** (Language in which the k-means clustering (k = 38) and assignment-table generation are performed)
- **BloodGen3module (R package)** (R package whose gridplot/fingerprintplot functions consume the module-aggregate assignment (Axx.yy identifiers) to render fingerprint grid and heatmap plots) — github.com/Drinchai/BloodGen3Module
- **WGCNA** (Co-expression network analysis framework underpinning construction of the module repertoire whose profiles are clustered)

## Examples

```
set.seed(1); km <- kmeans(scale(module_profiles), centers = 38); write.csv(data.frame(module = rownames(module_profiles), aggregate = paste0('A', km$cluster)), 'module_aggregate_assignments.csv', row.names = FALSE)
```

## Evaluation signals

- The assignment table contains exactly 382 module assignments distributed over exactly 38 aggregate identifiers (A1–A38)
- Exactly 27 of the 38 aggregates group two or more modules, matching the paper's reported structure
- Each multi-module aggregate maps to one row of the fingerprint grid, and modules within a row are ordered first by functional annotation, then alphabetically, with unannotated modules last
- Grid identifiers follow the Axx.yy convention and are accepted by the gridplot/fingerprintplot functions without errors

## Limitations

- The module repertoire is not a ground truth: some aggregates are clearly not homogeneous in terms of functionality, and heterogeneity also exists within modules
- The repertoire could be altered by addition or removal of datasets used for its construction, with more variation expected at the module level than the aggregate level
- The repertoire was constructed at the probe level on Illumina HT12 v3 BeadArrays; 388 out of 14,168 probes are duplicate genes found in different modules, requiring a handling choice
- The functional annotation map is expected to continue evolving even while the framework stays fixed

## Evidence

- [discussion] Two-tiered grouping of modules at the module level and module aggregate level: "the two-tiered grouping, first at the module level and at the module aggregate level, is new for this third-generation module repertoire"
- [abstract] 16 datasets, 985 profiles: "co-clustering patterns observed across sixteen immunological and physiological states encompassing 985 blood transcriptome profiles"
- [discussion] Modules grouped by annotation then alphabetically within grid rows: "modules within a given aggregate (=row) are grouped first according to functional annotations and then among the different annotations, from left to right according to alphabetical order"
- [discussion] BloodGen3module package generates fingerprint grids: "a R package, “BloodGen3module”, that can be used to generate fingerprint grid plots and heatmaps"
- [readme] gridplot consumes Group_df to produce grid plot PDF: "The **gridplot** function will generate a grid plot as a PDF file."
