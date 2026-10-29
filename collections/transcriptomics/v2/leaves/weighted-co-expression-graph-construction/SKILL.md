---
name: weighted-co-expression-graph-construction
description: 'Use when building a fixed transcriptional module repertoire from multiple
  blood transcriptome cohorts: you have several GEO expression matrices and need a
  co-clustering-derived weighted graph to segment transcripts into modules.'
when_to_use_negative:
- The input data are from tissues other than blood or from non-human species — the
  BloodGen3 repertoire is specifically designed for human blood transcriptome data,
  and other tissues/species would require separate frameworks
- The goal is to analyze new samples against the already-fixed 382-module repertoire
  (use Groupcomparison/Individualcomparison fingerprint workflows instead of rebuilding
  the graph)
- You have only a single cohort with no cross-cohort co-clustering signal — the graph
  weights derive from co-clustering frequencies across 16 datasets
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: R
  role: Implementation language; stats::kmeans with the jump statistic determines
    per-dataset cluster structure across profiles
  repo: https://github.com/Drinchai/BloodGen3Module
- name: ExperimentHub
  role: Retrieves the GEO reference cohort expression data (e.g., GSE13013, GSE13015
    via EH5429) used as construction input
- name: WGCNA
  role: Whole-genome co-expression network analysis framework underlying the co-expression
    graph construction
- name: BloodGen3Module
  role: R package whose annotated module membership table (Gen3 two-tiered module/aggregate
    grouping) is the target output and supports downstream fingerprinting
  repo: github.com/Drinchai/BloodGen3Module
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-24584-w
    title: Development of a fixed module repertoire for the analysis and interpretation
      of blood transcriptome data
  source_content_hash: sha256:daa7148fdb39bcc123622ef26addd4fe0ba80d2044ad8e7344412d1f310a5242
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
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# weighted-co-expression-graph-construction

## Summary

Constructs a weighted co-expression graph over blood transcripts by tabulating cross-cohort co-clustering frequencies from k-means clustering of 16 reference blood transcriptome datasets (985 profiles), then iteratively mining sub-networks to derive the fixed BloodGen3 repertoire of 382 modules spanning 14,168 transcripts.

## When to use

Use when building a fixed transcriptional module repertoire from multiple blood transcriptome cohorts: you have several GEO expression matrices and need a co-clustering-derived weighted graph to segment transcripts into modules.

## When NOT to use

- The input data are from tissues other than blood or from non-human species — the BloodGen3 repertoire is specifically designed for human blood transcriptome data, and other tissues/species would require separate frameworks
- The goal is to analyze new samples against the already-fixed 382-module repertoire (use Groupcomparison/Individualcomparison fingerprint workflows instead of rebuilding the graph)
- You have only a single cohort with no cross-cohort co-clustering signal — the graph weights derive from co-clustering frequencies across 16 datasets

## Inputs

- 16 GEO reference blood transcriptome expression matrices (985 unique profiles, non-log2 transformed, e.g., retrieved via ExperimentHub)
- Expression matrices annotated with valid Gene Symbols as row names
- Per-dataset sample annotation tables

## Outputs

- Cross-cohort co-clustering frequency table over the 985 profiles
- Weighted co-expression graph over 14,168 transcripts
- Module membership table mapping every transcript to its module and aggregate identifier (two-tiered module/aggregate grouping)
- Fixed repertoire of 382 transcriptional modules covering 14,168 transcripts (95.8% of detected transcripts)

## How to apply

Retrieve the 16 GEO reference blood transcriptome cohorts (e.g., via ExperimentHub, e.g., GSE13015 as res[["EH5429"]]) keeping expression matrices non-log2 transformed. For each dataset, run k-means clustering in R (stats::kmeans) using the jump statistic to determine per-dataset cluster structure across the 985 profiles. Tabulate co-clustering frequencies across the 16 datasets to build a cross-cohort co-clustering table, then convert these frequencies into a weighted co-expression graph over the 14,168 detected transcripts. Iteratively mine sub-networks from the weighted graph until segmentation stabilizes at the published 382 modules, and export a module membership table using the two-tiered module/aggregate grouping new to this third-generation repertoire. Rationale: co-clustering across sixteen immunological and physiological states gives module definitions broader state coverage than prior repertoires built from as few as seven states, and WGCNA-style whole-genome co-expression network analysis provides the graph framework.

## Related tools

- **R** (Implementation language; stats::kmeans with the jump statistic determines per-dataset cluster structure across profiles) — https://github.com/Drinchai/BloodGen3Module
- **ExperimentHub** (Retrieves the GEO reference cohort expression data (e.g., GSE13013, GSE13015 via EH5429) used as construction input)
- **WGCNA** (Whole-genome co-expression network analysis framework underlying the co-expression graph construction)
- **BloodGen3Module** (R package whose annotated module membership table (Gen3 two-tiered module/aggregate grouping) is the target output and supports downstream fingerprinting) — github.com/Drinchai/BloodGen3Module

## Examples

```
res = query(ExperimentHub(), "GSE13015"); GSE13015 = res[["EH5429"]]; km_fit <- stats::kmeans(t(expr(GSE13015)), centers = k)
```

## Evaluation signals

- The output repertoire comprises exactly 382 modules spanning 14,168 transcripts, matching the published BloodGen3 module list
- The 14,168 transcripts represent 95.8% of detected transcripts — confirm coverage against the input probe set
- Every transcript in the membership table is assigned to both a module and a module aggregate (two-tiered Gen3 grouping)
- Per-dataset k-means cluster counts are consistent with the jump statistic criterion applied to each of the 16 cohorts
- Duplicate probes (388 out of 14,168 map to the same genes) are handled explicitly with a documented assignment choice

## Limitations

- The repertoire was constructed at the probe level on Illumina HT12 v3 BeadArray data, which pre-dates RNA sequencing; 388 out of 14,168 probes are duplicate genes found in different modules requiring a handling choice
- The repertoire could be altered by addition or removal of construction datasets; more variation is expected at the module level than at the aggregate level
- The module repertoire is not a ground truth — some aggregates are heterogeneous in functionality and annotation titles confer subjectivity

## Evidence

- [abstract] The construction of this repertoire is based on co-clustering patterns observed across sixteen immunological and physiological states encompassing 985 blood transcriptome profiles.: "The construction of this repertoire is based on co-clustering patterns observed across sixteen immunological and physiological states encompassing 985 blood transcriptome profiles."
- [discussion] whole-genome co-expression network analysis (WGCNA)47: "whole-genome co-expression network analysis (WGCNA)47"
- [intro] Specifically, 16 input datasets were employed for the construction of the BloodGen3 repertoire, comprising 985 unique blood transcriptome profiles: "Specifically, 16 input datasets were employed for the construction of the BloodGen3 repertoire, comprising 985 unique blood transcriptome profiles"
- [intro] extensive functional annotations and reference transcriptome profiles for each of the 382 modules comprising the repertoire have been made available via interactive circle packing plots: "extensive functional annotations and reference transcriptome profiles for each of the 382 modules comprising the repertoire have been made available via interactive circle packing plots"
- [discussion] the BloodGen3 repertoire was constructed at the probe level (those present on the Illumina HT12 v3 Beadarrays): "the BloodGen3 repertoire was constructed at the probe level (those present on the Illumina HT12 v3 Beadarrays)"
