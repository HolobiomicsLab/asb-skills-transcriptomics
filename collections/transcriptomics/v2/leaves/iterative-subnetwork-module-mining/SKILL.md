---
name: iterative-subnetwork-module-mining
description: 'Use when building a fixed blood transcriptome module repertoire from
  multiple GEO cohort expression datasets: transcripts must be grouped by co-clustering
  across cohorts and mined from a weighted graph into stable modules and aggregates.'
when_to_use_negative:
- Analyzing tissues other than blood or non-human species — the BloodGen3 repertoire
  is specifically designed for human blood transcriptome profiling data, and other
  contexts require separate frameworks
- Performing group comparison or individual fingerprint analysis on an already-built
  repertoire — use the BloodGen3Module Groupcomparison/Individualcomparison/gridplot
  functions instead of re-mining the network
- Input expression data are already summarized into fixed gene sets/modules — this
  skill only applies when constructing a new repertoire from raw cohort-level expression
  profiles
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: R
  role: 'Implementation language for the stepwise pipeline: k-means clustering with
    the jump statistic (stats::kmeans) per dataset and cross-cohort co-clustering
    tabulation'
- name: BloodGen3Module
  role: R package accompanying the constructed 382-module repertoire; provides module
    membership annotation and downstream fingerprint analysis of the mined modules
  repo: github.com/Drinchai/BloodGen3Module
- name: ExperimentHub
  role: Retrieval of the 16 GEO reference blood transcriptome datasets used as construction
    input
- name: WGCNA
  role: Whole-genome co-expression network analysis framework underpinning the weighted-graph
    construction and sub-network mining
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-24584-w
    title: Development of a fixed module repertoire for the analysis and interpretation
      of blood transcriptome data
  source_content_hash: sha256:f86f7e11d2ebf2c40a689772240cffbc3911dd9478215fe9631074b870908a4b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BloodGen3
  - BloodGen3Module
  - ExperimentHub
  - R
  - WGCNA
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# iterative-subnetwork-module-mining

## Summary

Constructs the BloodGen3 fixed module repertoire by co-clustering transcripts across 16 blood transcriptome datasets (985 unique profiles), converting cross-cohort co-clustering frequencies into a weighted co-expression graph, and iteratively mining sub-networks until segmentation stabilizes into 382 modules covering 14,168 transcripts.

## When to use

Use when building a fixed blood transcriptome module repertoire from multiple GEO cohort expression datasets: transcripts must be grouped by co-clustering across cohorts and mined from a weighted graph into stable modules and aggregates.

## When NOT to use

- Analyzing tissues other than blood or non-human species — the BloodGen3 repertoire is specifically designed for human blood transcriptome profiling data, and other contexts require separate frameworks
- Performing group comparison or individual fingerprint analysis on an already-built repertoire — use the BloodGen3Module Groupcomparison/Individualcomparison/gridplot functions instead of re-mining the network
- Input expression data are already summarized into fixed gene sets/modules — this skill only applies when constructing a new repertoire from raw cohort-level expression profiles

## Inputs

- Non-log2 transformed expression matrices from the 16 GEO reference blood transcriptome cohorts (e.g., GSE100150), 985 unique profiles, retrieved via ExperimentHub
- Dataset/sample metadata for the 16 immunological and physiological states used for construction

## Outputs

- Module membership table mapping each of 14,168 transcripts to one of 382 modules and to its module aggregate (two-tiered Gen3_ann-style grouping)
- Weighted cross-cohort co-clustering co-expression graph over the 14,168 transcripts

## How to apply

Retrieve the 16 GEO reference blood transcriptome cohorts (e.g., GSE100150) comprising 985 unique profiles via ExperimentHub in R, keeping expression matrices non-log2 transformed. For each dataset, run k-means clustering with the jump statistic (stats::kmeans) to determine per-dataset cluster structure across profiles, then tabulate co-clustering frequencies across all 16 datasets over the 985 profiles. Convert the co-clustering frequencies into a weighted co-expression graph over the 14,168 transcripts and iteratively mine sub-networks from the weighted graph until the segmentation stabilizes at the published 382 modules. Export a module membership table mapping every transcript to its module and aggregate identifier using the two-tiered module/aggregate grouping (as in Gen3_ann). Validate that the output repertoire comprises exactly 382 modules spanning 14,168 transcripts — 95.8% of detected transcripts — matching the published BloodGen3 module list.

## Related tools

- **R** (Implementation language for the stepwise pipeline: k-means clustering with the jump statistic (stats::kmeans) per dataset and cross-cohort co-clustering tabulation)
- **BloodGen3Module** (R package accompanying the constructed 382-module repertoire; provides module membership annotation and downstream fingerprint analysis of the mined modules) — github.com/Drinchai/BloodGen3Module
- **ExperimentHub** (Retrieval of the 16 GEO reference blood transcriptome datasets used as construction input)
- **WGCNA** (Whole-genome co-expression network analysis framework underpinning the weighted-graph construction and sub-network mining)

## Examples

```
library(ExperimentHub); dat = ExperimentHub(); res = query(dat, "GSE13015"); GSE13015 = res[["EH5429"]]
```

## Evaluation signals

- Output repertoire contains exactly 382 modules spanning 14,168 transcripts, i.e., 95.8% of detected transcripts, matching the published BloodGen3 module list
- Every transcript maps to both a module and a module aggregate (two-tiered grouping consistent with Gen3_ann identifiers); no transcripts are left unmapped
- The co-clustering table covers all 16 datasets and 985 unique profiles with no missing cohort contributions
- Duplicate probes assigned to different modules (388 out of 14,168) are explicitly identified and handled rather than silently dropped
- Iterative sub-network mining has converged: module segmentation no longer changes between successive mining iterations

## Limitations

- The repertoire was constructed at the probe level on Illumina HT12 v3 BeadArrays (pre-dating RNA-seq); 388 of 14,168 probes are duplicates of the same gene found in different modules, requiring an explicit handling choice
- The module repertoire is not a ground truth: some aggregates are heterogeneous in functionality and heterogeneity also exists within modules; functional annotation titles confer subjectivity
- Adding or removing construction datasets can alter the repertoire, with more variation expected at the module level than at the aggregate level

## Evidence

- [abstract] Co-clustering across sixteen states underpins repertoire construction: "The construction of this repertoire is based on co-clustering patterns observed across sixteen immunological and physiological states encompassing 985 blood transcriptome profiles."
- [intro] 16 input datasets comprising 985 profiles: "Specifically, 16 input datasets were employed for the construction of the BloodGen3 repertoire, comprising 985 unique blood transcriptome profiles"
- [discussion] Two-tiered module/aggregate grouping is new in Gen3: "the two-tiered grouping, first at the module level and at the module aggregate level, is new for this third-generation module repertoire"
- [discussion] Duplicate probes across modules are a small minority (388/14,168): "which in our hands shows to have very little impact on the results since they concern only a small minority of genes (388 out of 14,168 probes)"
