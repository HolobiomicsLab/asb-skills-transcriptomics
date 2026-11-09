---
name: k-means-clustering-with-jump-statistic
description: Use when building a fixed co-expression module repertoire from multiple
  transcriptome cohorts by clustering each dataset independently (k-means with jump
  statistic) and tabulating cross-cohort co-clustering of transcripts.
when_to_use_negative:
- Analyzing non-blood tissues or other species — the article states the BloodGen3
  repertoire is specifically designed for human blood transcriptome data and other
  analyses require separate frameworks
- Downstream fingerprinting of a new dataset against the fixed repertoire — use the
  BloodGen3Module Groupcomparison/Individualcomparison/gridplot functions instead
  of re-running repertoire construction
- Expression matrices that are already log2 transformed — inputs must be normalized
  but non-log2 transformed
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: BloodGen3
  role: Fixed module repertoire whose construction pipeline embeds the per-dataset
    k-means/jump-statistic clustering step
- name: R
  role: Implementation language; stats::kmeans is used for clustering with the jump
    statistic
- name: BloodGen3Module
  role: Companion R package for downstream module repertoire analysis and fingerprint
    visualization after construction
  repo: github.com/Drinchai/BloodGen3Module
- name: WGCNA
  role: Whole-genome co-expression network analysis framework underlying the weighted-graph
    sub-network mining stage
- name: ExperimentHub
  role: Retrieves the reference GEO cohorts (e.g., GSE13013, GSE13015 / EH5429) used
    as clustering input
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-24584-w
    title: Development of a fixed module repertoire for the analysis and interpretation
      of blood transcriptome data
  source_content_hash: sha256:60f88a65b30ac7871350be22204502d6afe15b744b068c7ae6c6ab052ba84973
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - BloodGen3
  - BloodGen3Module
  - ExperimentHub
  - R
  - WGCNA
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# k-means-clustering-with-jump-statistic

## Summary

Per-dataset k-means clustering (R stats::kmeans) with the jump statistic is the first stage of the BloodGen3 repertoire construction: cluster structures from 16 blood transcriptome datasets (985 profiles) are converted into cross-cohort co-clustering frequencies that seed the 382-module repertoire.

## When to use

Use when building a fixed co-expression module repertoire from multiple transcriptome cohorts by clustering each dataset independently (k-means with jump statistic) and tabulating cross-cohort co-clustering of transcripts.

## When NOT to use

- Analyzing non-blood tissues or other species — the article states the BloodGen3 repertoire is specifically designed for human blood transcriptome data and other analyses require separate frameworks
- Downstream fingerprinting of a new dataset against the fixed repertoire — use the BloodGen3Module Groupcomparison/Individualcomparison/gridplot functions instead of re-running repertoire construction
- Expression matrices that are already log2 transformed — inputs must be normalized but non-log2 transformed

## Inputs

- Expression matrices from 16 GEO blood transcriptome datasets (e.g., GSE100150, GSE13013, GSE13015), retrieved via ExperimentHub, kept non-log2 transformed
- 985 unique blood transcriptome profiles spanning autoimmune, infectious and inflammatory diseases, cancer, liver transplant recipients, and pregnancy

## Outputs

- Per-dataset k-means cluster assignments (cluster count chosen by jump statistic)
- Cross-cohort co-clustering frequency table over the 985 profiles
- Weighted co-expression graph over 14,168 transcripts (95.8% of detected transcripts)
- BloodGen3 module membership table with 382 modules under two-tiered module/aggregate grouping

## How to apply

Retrieve the 16 GEO reference blood transcriptome cohorts (e.g., GSE100150; 985 unique profiles) via ExperimentHub in R, keeping expression matrices non-log2 transformed. For each dataset, run k-means clustering with the jump statistic (R stats::kmeans) to determine per-dataset cluster structure across profiles, with the jump statistic selecting the number of clusters. Tabulate co-clustering frequencies across all 16 datasets to build the cross-cohort co-clustering table over the 985 profiles, then convert these frequencies into a weighted co-expression graph over the 14,168 transcripts. Iteratively mine sub-networks from this weighted graph (WGCNA-style network mining) until segmentation stabilizes at the published 382 modules, and export a module membership table mapping every transcript to its module and aggregate identifier using the two-tiered module/aggregate grouping.

## Related tools

- **BloodGen3** (Fixed module repertoire whose construction pipeline embeds the per-dataset k-means/jump-statistic clustering step)
- **R** (Implementation language; stats::kmeans is used for clustering with the jump statistic)
- **BloodGen3Module** (Companion R package for downstream module repertoire analysis and fingerprint visualization after construction) — github.com/Drinchai/BloodGen3Module
- **WGCNA** (Whole-genome co-expression network analysis framework underlying the weighted-graph sub-network mining stage)
- **ExperimentHub** (Retrieves the reference GEO cohorts (e.g., GSE13013, GSE13015 / EH5429) used as clustering input)

## Examples

```
km_fit <- stats::kmeans(t(expr_matrix), centers = k_jump)  # per-dataset clustering; k chosen via jump statistic
```

## Evaluation signals

- The mined repertoire contains exactly 382 modules, matching the published BloodGen3 module list
- The repertoire spans 14,168 transcripts, i.e., 95.8% of detected transcripts (only 388 duplicate probes out of 14,168 remain)
- The co-clustering frequency table is computed across all 16 datasets covering 985 unique profiles
- Every transcript maps to both a module and a module aggregate identifier (two-tiered grouping as in Gen3)
- Input expression matrices were kept non-log2 transformed throughout construction

## Limitations

- The repertoire was constructed at the probe level on Illumina HT12 v3 BeadArrays, which pre-dates RNA sequencing; 388 of 14,168 probes are duplicate genes found in different modules requiring a handling choice
- The repertoire could be altered by addition or removal of datasets used for construction, with more variation expected at the module level than the aggregate level
- The module repertoire is not a ground truth: some aggregates are heterogeneous and functional annotation titles confer subjectivity

## Evidence

- [abstract] The construction of this repertoire is based on co-clustering patterns observed across sixteen immunological and physiological states encompassing 985 blood transcriptome profiles.: "The construction of this repertoire is based on co-clustering patterns observed across sixteen immunological and physiological states encompassing 985 blood transcriptome profiles."
- [intro] Specifically, 16 input datasets were employed for the construction of the BloodGen3 repertoire, comprising 985 unique blood transcriptome profiles: "Specifically, 16 input datasets were employed for the construction of the BloodGen3 repertoire, comprising 985 unique blood transcriptome profiles"
- [discussion] whole-genome co-expression network analysis (WGCNA)47: "whole-genome co-expression network analysis (WGCNA)47"
- [discussion] the two-tiered grouping, first at the module level and at the module aggregate level, is new for this third-generation module repertoire: "the two-tiered grouping, first at the module level and at the module aggregate level, is new for this third-generation module repertoire"
- [discussion] the BloodGen3 repertoire was constructed at the probe level (those present on the Illumina HT12 v3 Beadarrays): "the BloodGen3 repertoire was constructed at the probe level (those present on the Illumina HT12 v3 Beadarrays)"
- [readme] devtools::install_github("Drinchai/BloodGen3Module"): "devtools::install_github("Drinchai/BloodGen3Module")"
