---
name: per-sample-dmdt-table-parsing
description: Use when a task requires computing the fraction of disease samples exhibiting
  a specific isoform switch (e.g. HSPH1-001 vs HSPH1-002) from CanIsoNet's per-sample
  dMDT tables with expression values, rather than gene-level summary statistics.
when_to_use_negative:
- You need genome-wide isoform switching statistics or network disruptions across
  the 31 disease types rather than a specific event's per-sample frequencies — use
  CanIsoNet's disease statistics browsing views instead.
- Your input is a raw RNA-Seq dataset (FASTQ/BAM) rather than the already-computed
  CanIsoNet dMDT table — this skill parses precomputed switching events, it does not
  detect isoform switches de novo.
- The disease dataset is not among those covered by CanIsoNet (27 PCAWG cancer types
  plus the OIH, Alzheimer's, Parkinson's and Behcet's RNA-Seq datasets).
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CanIsoNet
  role: Web database providing the disease-type specific dMDT tables and per-sample
    isoform switching events with expression values that are parsed and counted
  repo: https://github.com/kahramanlab/CanIsoNet_Web
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbad050
    title: 'CanIsoNet: a database to study the functional impact of isoform switching
      events in diseases'
  source_content_hash: sha256:cd32bf9e32d041e3577ad65f9a8a75b4ec7a5e4ef5f2dcd9eee9d3cf8df2d55e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kahramanlab/CanIsoNet_Web
  tools:
  - CanIsoNet
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# per-sample-dmdt-table-parsing

## Summary

Extracts and counts per-sample isoform switching events from the CanIsoNet disease-type specific dMDT tables (e.g. the Alzheimer's disease RNA-Seq dataset GSE203206), converting sample-level switch records into switch-frequency metrics such as HSPH1-001→HSPH1-002 in 21% of samples and SLC25A25-005 in 11 of 39 samples.

## When to use

Use when a task requires computing the fraction of disease samples exhibiting a specific isoform switch (e.g. HSPH1-001 vs HSPH1-002) from CanIsoNet's per-sample dMDT tables with expression values, rather than gene-level summary statistics.

## When NOT to use

- You need genome-wide isoform switching statistics or network disruptions across the 31 disease types rather than a specific event's per-sample frequencies — use CanIsoNet's disease statistics browsing views instead.
- Your input is a raw RNA-Seq dataset (FASTQ/BAM) rather than the already-computed CanIsoNet dMDT table — this skill parses precomputed switching events, it does not detect isoform switches de novo.
- The disease dataset is not among those covered by CanIsoNet (27 PCAWG cancer types plus the OIH, Alzheimer's, Parkinson's and Behcet's RNA-Seq datasets).

## Inputs

- CanIsoNet per-sample dMDT table for a disease dataset (e.g. Alzheimer's disease, GSE203206), listing per-sample isoform switching events with expression values
- Target transcript identifiers (e.g. HSPH1-001/858 AA, HSPH1-002/814 AA, SLC25A25-005)

## Outputs

- Per-sample counts of samples showing the specified isoform switch
- Switch-frequency metric (e.g. 21% for HSPH1-001→HSPH1-002; 11/39 for SLC25A25-005)
- Summary CSV such as alzheimer_dmdt_frequencies.csv

## How to apply

Access CanIsoNet at caniso.net and open the per-sample isoform switching view for the target disease dataset (for Alzheimer's disease this is one of the four RNA-Seq datasets alongside OIH, Parkinson's disease and Behcet's disease). The per-sample table lists each isoform switching event with its expression values per sample; filter rows to the gene of interest (e.g. HSPH1 switch between the 858 AA HSPH1-001 and the shorter 814 AA HSPH1-002, or SLC25A25-005). Count samples carrying the switch and divide by the total number of samples in the disease dataset (for Alzheimer's, SLC25A25-005 is 11 of 39 samples, and the HSPH1 switch is 21%). As a control, check the same dMDT in the other disease datasets to confirm disease-specificity (SLC25A25-005 is absent in OIH, Parkinson's and Behcet's). Write the computed frequencies to a summary file (e.g. alzheimer_dmdt_frequencies.csv) and compare against paper-reported values to validate.

## Related tools

- **CanIsoNet** (Web database providing the disease-type specific dMDT tables and per-sample isoform switching events with expression values that are parsed and counted) — https://github.com/kahramanlab/CanIsoNet_Web

## Evaluation signals

- Computed HSPH1-001→HSPH1-002 switch frequency equals the paper-reported 21% of analysed Alzheimer's samples.
- SLC25A25-005 count equals 11 out of 39 Alzheimer's samples.
- SLC25A25-005 is confirmed absent from the OIH, Parkinson's disease and Behcet's disease dMDT tables.
- Transcript amino-acid lengths match expectations (HSPH1-001 = 858 AA, HSPH1-002 = 814 AA).
- Output file alzheimer_dmdt_frequencies.csv exists and its frequencies match the database per-sample counts.

## Limitations

- The per-sample event view is only available for the datasets precomputed in CanIsoNet (27 PCAWG cancer types and four RNA-Seq datasets on OIH, Alzheimer's disease, Parkinson's disease and Behcet's disease).
- Reported isoform annotation counts are inconsistent between sources (article: 11,811 isoforms; README: 11,528 isoforms), so database contents may vary by release.
- Frequencies depend on the disease dataset sample size (e.g. 39 Alzheimer's samples); switching to another dataset changes the denominator and the specificity check.

## Evidence

- [intro] per-sample isoform switching events with expression values: "4) Exploring the isoform switching event in each sample with their expression values."
- [readme] disease datasets covered by per-sample exploration: "four publicly available RNA-Seq datasets on OIH, Alzheimer's Disease, Parkinson's Disease and Behcet's Disease"
- [methods] database availability for extraction: "CanIsoNet is freely available at [caniso.net](https://www.caniso.net/)"
