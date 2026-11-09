---
name: isoform-expression-count-aggregation
description: Use when you need per-sample isoform-switch frequencies or isoform-expression
  prevalence counts from CanIsoNet's precomputed dMDT tables, e.g. verifying paper-reported
  values like '21% of samples' or '11 of 39 samples' for a specific transcript pair.
when_to_use_negative:
- Input is raw RNA-Seq FASTQ/BAM needing alignment and transcript quantification —
  CanIsoNet serves precomputed isoform switching results, not a quantification pipeline
- The goal is functional-impact prediction of a switch (STRING network disruption,
  ClinVar annotation) rather than per-sample frequency counting
- The isoform event of interest is outside the 31 disease types covered by CanIsoNet
  (27 PCAWG cancer types plus OIH, Alzheimer's, Parkinson's, Behcet's disease)
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: CanIsoNet
  role: Source of the per-sample dMDT tables and isoform expression values that are
    counted to derive switch frequencies
  repo: https://github.com/kahramanlab/CanIsoNet_Web
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbad050
    title: 'CanIsoNet: a database to study the functional impact of isoform switching
      events in diseases'
  source_content_hash: sha256:04cb216057f7cb283335dabe0cf0ae2de9875a0f2859cd62a3bd0bc3fd7960e0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kahramanlab/CanIsoNet_Web
  tools:
  - CanIsoNet
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4166666666666667
n_claims: 12
---

# isoform-expression-count-aggregation

## Summary

Aggregates the per-sample dMDT (disease-specific mutual dominant transcript) tables in CanIsoNet into per-sample isoform-switch frequency counts, e.g. the 21% frequency of the HSPH1-001 (858 AA) to HSPH1-002 (814 AA) switch and the 11-of-39 sample count for SLC25A25-005 expression in the Alzheimer's disease dataset (GSE203206).

## When to use

Use when you need per-sample isoform-switch frequencies or isoform-expression prevalence counts from CanIsoNet's precomputed dMDT tables, e.g. verifying paper-reported values like '21% of samples' or '11 of 39 samples' for a specific transcript pair.

## When NOT to use

- Input is raw RNA-Seq FASTQ/BAM needing alignment and transcript quantification — CanIsoNet serves precomputed isoform switching results, not a quantification pipeline
- The goal is functional-impact prediction of a switch (STRING network disruption, ClinVar annotation) rather than per-sample frequency counting
- The isoform event of interest is outside the 31 disease types covered by CanIsoNet (27 PCAWG cancer types plus OIH, Alzheimer's, Parkinson's, Behcet's disease)

## Inputs

- CanIsoNet per-sample dMDT table for the Alzheimer's disease dataset (GSE203206) with per-sample isoform switching events and expression values
- CanIsoNet per-sample dMDT tables for the OIH, Parkinson's disease, and Behcet's disease datasets (for cross-disease absence checks)

## Outputs

- Computed switch frequency for HSPH1-001/HSPH1-002 (fraction of Alzheimer's samples, expected 21%)
- Sample count for SLC25A25-005 expression (expected 11 of 39 Alzheimer's samples) with absence confirmation in OIH, Parkinson's disease, and Behcet's disease
- Summary table written to alzheimer_dmdt_frequencies.csv

## How to apply

Access CanIsoNet at caniso.net and open the disease-type specific dMDT view, which CanIsoNet provides as its fourth user feature: exploring the isoform switching event in each sample with their expression values. From the per-sample table for the Alzheimer's disease dataset (GSE203206), count samples showing the switch between the canonical HSPH1-001 transcript (858 AA) and the shorter 814 AA HSPH1-002 transcript, then divide by the total number of Alzheimer's samples to obtain the switch frequency (expected 21%). Separately count samples expressing the SLC25A25-005 isoform (expected 11 of 39) and check the OIH, Parkinson's disease, and Behcet's disease dMDT tables to verify SLC25A25-005 is absent in those diseases. The rationale is that CanIsoNet's per-sample exploration is the only interface exposing sample-level isoform expression values, so frequencies must be computed from those rows rather than inferred from summary statistics.

## Related tools

- **CanIsoNet** (Source of the per-sample dMDT tables and isoform expression values that are counted to derive switch frequencies) — https://github.com/kahramanlab/CanIsoNet_Web

## Evaluation signals

- Computed HSPH1-001/HSPH1-002 switch frequency equals the paper-reported 21% of Alzheimer's samples
- SLC25A25-005 expression count equals 11 of 39 Alzheimer's samples
- SLC25A25-005 is confirmed absent from the OIH, Parkinson's disease, and Behcet's disease datasets in the database
- Summary written to alzheimer_dmdt_frequencies.csv matches the recomputed values
- Every counted sample row carries per-sample expression values from the dMDT table (not inferred from aggregates)

## Limitations

- Counts are restricted to the 31 disease types precomputed in CanIsoNet (27 PCAWG cancer types plus OIH, Alzheimer's, Parkinson's, Behcet's disease); no custom dataset ingestion
- Isoform annotation counts are version-dependent: the paper reports 11,811 annotated isoforms while the README reports 11,528, so counts may vary between database versions
- Frequencies depend on the fixed sample sizes of the underlying datasets (e.g. 39 Alzheimer's samples), so they cannot be re-estimated from new cohorts

## Evidence

- [readme] Exploring the isoform switching event in each sample with their expression values.: "4) Exploring the isoform switching event in each sample with their expression values. <br>"
- [abstract] CanIsoNet provides annotations for 11,811 isoforms and 14,357 unique isoform switching events across 31 disease types: "Various annotations for 11 811 isoforms and 14 357 unique isoform switching events across 31 different disease types are available."
- [intro] CanIsoNet is a database covering per-sample isoform switching in four non-cancer RNA-Seq disease datasets: "four publicly available RNA-Seq datasets on OIH, Alzheimer's Disease, Parkinson's Disease and Behcet's Disease"
- [readme] It holds annotations for 11,528 isoforms and 14,357 unique isoform switching events across 31 diseases.: "It holds annotations for 11,528 isoforms and 14,357 unique isoform switching events across 31 diseases."
