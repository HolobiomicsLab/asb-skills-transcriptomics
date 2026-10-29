---
name: co-clustering-frequency-tabulation
description: Use when building a fixed transcriptional module repertoire from multiple
  gene expression cohorts, by quantifying cross-dataset co-clustering of transcripts
  over shared profiles.
when_to_use_negative:
- Analyses of tissues other than human blood, or other species — the BloodGen3 repertoire
  is specifically designed for human blood transcriptome profiling data
- The goal is downstream group comparison or fingerprinting of samples against an
  existing repertoire (use the BloodGen3Module Groupcomparison/Individualcomparison
  functions instead)
- A framework capturing specificities of a narrowly defined system outside blood is
  needed — co-clustering across diverse immunological/physiological states may miss
  such specificity
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
tools:
- name: BloodGen3
  role: The fixed 382-module repertoire produced by the co-clustering frequency tabulation
    and network-mining pipeline
  repo: https://github.com/Drinchai/BloodGen3Module
- name: R
  role: 'Implementation environment: stats::kmeans clustering per dataset, co-clustering
    tabulation, and graph mining'
- name: BloodGen3Module
  role: R package companion ecosystem for downstream analysis (Groupcomparison, gridplot,
    fingerprintplot) once the module membership table exists
  repo: https://github.com/Drinchai/BloodGen3Module
- name: ExperimentHub
  role: Retrieval of the reference GEO blood transcriptome input datasets (e.g., GSE13015
    = res[['EH5429']])
- name: WGCNA
  role: Whole-genome co-expression network analysis framework contextual to the co-expression
    graph construction
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-24584-w
    title: Development of a fixed module repertoire for the analysis and interpretation
      of blood transcriptome data
  source_content_hash: sha256:a52ef89790c855c73aa5020309932c55625bd023a61129228d150b9e3ca6d18c
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
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# co-clustering-frequency-tabulation

## Summary

Tabulates how frequently gene pairs (transcripts on Illumina HT-12 v3 BeadArrays) co-cluster across multiple blood transcriptome datasets and converts those frequencies into a weighted co-expression graph, which is iteratively mined to define a fixed module repertoire such as the 382-module BloodGen3 repertoire.

## When to use

Use when building a fixed transcriptional module repertoire from multiple gene expression cohorts, by quantifying cross-dataset co-clustering of transcripts over shared profiles.

## When NOT to use

- Analyses of tissues other than human blood, or other species — the BloodGen3 repertoire is specifically designed for human blood transcriptome profiling data
- The goal is downstream group comparison or fingerprinting of samples against an existing repertoire (use the BloodGen3Module Groupcomparison/Individualcomparison functions instead)
- A framework capturing specificities of a narrowly defined system outside blood is needed — co-clustering across diverse immunological/physiological states may miss such specificity

## Inputs

- 16 GEO blood transcriptome expression matrices (non-log2 transformed; Illumina HT-12 v3 BeadArray probe level), e.g. retrieved via ExperimentHub (EH5429 / GSE13015)
- 985 unique blood transcriptome profiles spanning autoimmune, infectious and inflammatory diseases, cancer, liver transplant recipients, and pregnancy

## Outputs

- Cross-cohort co-clustering frequency table over the 985 profiles
- Weighted co-expression graph over 14,168 transcripts
- Fixed repertoire of 382 modules spanning 14,168 transcripts (95.8% of detected transcripts)
- Module membership table mapping transcripts to modules and two-tiered module/aggregate identifiers

## How to apply

Retrieve the 16 GEO reference blood transcriptome cohorts (e.g., GSE13015, GSE13013 via ExperimentHub) comprising 985 unique profiles, keeping the expression matrices non-log2 transformed. For each dataset, run k-means clustering in R (stats::kmeans) using the jump statistic to determine the per-dataset cluster structure. Tabulate co-clustering frequencies across the 16 datasets over the 985 profiles, then convert the frequencies into a weighted co-expression graph over the 14,168 detected transcripts. Iteratively mine sub-networks from the weighted graph until segmentation stabilizes, yielding the 382-module repertoire covering 95.8% of detected transcripts. Export a module membership table assigning every transcript to its module and its two-tiered aggregate identifier (module level and module aggregate level, as in Gen3_ann).

## Related tools

- **BloodGen3** (The fixed 382-module repertoire produced by the co-clustering frequency tabulation and network-mining pipeline) — https://github.com/Drinchai/BloodGen3Module
- **R** (Implementation environment: stats::kmeans clustering per dataset, co-clustering tabulation, and graph mining)
- **BloodGen3Module** (R package companion ecosystem for downstream analysis (Groupcomparison, gridplot, fingerprintplot) once the module membership table exists) — https://github.com/Drinchai/BloodGen3Module
- **ExperimentHub** (Retrieval of the reference GEO blood transcriptome input datasets (e.g., GSE13015 = res[['EH5429']]))
- **WGCNA** (Whole-genome co-expression network analysis framework contextual to the co-expression graph construction)

## Examples

```
cl <- stats::kmeans(expr_matrix, centers = k, nstart = 25)  # run per dataset with jump statistic, then tabulate co-clustering frequencies across the 16 cohorts
```

## Evaluation signals

- Final repertoire comprises exactly 382 modules spanning 14,168 transcripts, matching the published BloodGen3 module list
- Coverage check: 14,168 transcripts equals 95.8% of detected transcripts (only 388 of 14,168 probes are duplicate genes found in different modules)
- Every transcript in the membership table maps to both a module and a module aggregate (two-tiered grouping)
- Co-clustering table is built over 985 unique profiles across 16 datasets; input matrices remain non-log2 transformed
- Module assignments are stable — iterative sub-network mining proceeds until segmentation stabilizes

## Limitations

- The repertoire was constructed at the probe level on Illumina HT-12 v3 BeadArrays, which pre-dates RNA sequencing; 388 of 14,168 probes are duplicate genes mapping to the same gene in different modules, requiring a handling choice (though impact is small since they concern only a small minority of genes)
- The module repertoire is not a ground truth: some aggregates are not homogeneous in functionality, and heterogeneity also exists within modules; functional annotation titles confer subjectivity
- The repertoire could be altered by addition or removal of datasets used for its construction, with more variation expected at the module level than at the aggregate level
- The functional annotation map is expected to continue evolving, while the module framework itself stays fixed

## Evidence

- [abstract] The construction of this repertoire is based on co-clustering patterns observed across sixteen immunological and physiological states encompassing 985 blood transcriptome profiles.: "The construction of this repertoire is based on co-clustering patterns observed across sixteen immunological and physiological states encompassing 985 blood transcriptome profiles."
- [intro] Specifically, 16 input datasets were employed for the construction of the BloodGen3 repertoire, comprising 985 unique blood transcriptome profiles: "Specifically, 16 input datasets were employed for the construction of the BloodGen3 repertoire, comprising 985 unique blood transcriptome profiles"
- [intro] extensive functional annotations and reference transcriptome profiles for each of the 382 modules comprising the repertoire have been made available: "extensive functional annotations and reference transcriptome profiles for each of the 382 modules comprising the repertoire have been made available"
- [discussion] they concern only a small minority of genes (388 out of 14,168 probes): "which in our hands shows to have very little impact on the results since they concern only a small minority of genes (388 out of 14,168 probes)"
- [discussion] the two-tiered grouping, first at the module level and at the module aggregate level, is new for this third-generation module repertoire: "the two-tiered grouping, first at the module level and at the module aggregate level, is new for this third-generation module repertoire"
