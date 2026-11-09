---
name: cross-disease-isoform-specificity-check
description: Use when a candidate isoform switch (e.g. HSPH1-001↔HSPH1-002, SLC25A25-005)
  found in one CanIsoNet disease dataset must be checked for absence in the OIH, Alzheimer's,
  Parkinson's and Behcet's Disease datasets to claim disease specificity.
when_to_use_negative:
- The input is raw RNA-Seq reads or an unquantified transcript abundance matrix —
  CanIsoNet serves precomputed isoform switching events, not primary quantification
- The isoform of interest falls in one of the 27 PCAWG cancer types only, where the
  cross-disease panel should instead compare against other PCAWG disease types rather
  than the four non-cancer RNA-Seq datasets
- The question is functional/pathogenic impact of a switch (STRING network disruption,
  ClinVar annotation) rather than its sample-level frequency and cross-disease specificity
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CanIsoNet
  role: Web database queried for disease-type specific dMDTs and per-sample isoform
    switching events with expression values across the OIH, Alzheimer's, Parkinson's
    and Behcet's Disease datasets
  repo: https://github.com/kahramanlab/CanIsoNet_Web
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioadv/vbad050
    title: 'CanIsoNet: a database to study the functional impact of isoform switching
      events in diseases'
  source_content_hash: sha256:689a2087553d07e510251b5ad86ad17a7f23c21f7889334c93c8f6d309ed3d6b
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
yes_rate: 0.7142857142857143
n_claims: 7
---

# cross-disease-isoform-specificity-check

## Summary

Verifies that an isoform switching event observed in one disease dataset in CanIsoNet is specific to that disease by recomputing per-sample frequencies across the other disease datasets (OIH, Parkinson's Disease, Behcet's Disease) hosted in the same database.

## When to use

Use when a candidate isoform switch (e.g. HSPH1-001↔HSPH1-002, SLC25A25-005) found in one CanIsoNet disease dataset must be checked for absence in the OIH, Alzheimer's, Parkinson's and Behcet's Disease datasets to claim disease specificity.

## When NOT to use

- The input is raw RNA-Seq reads or an unquantified transcript abundance matrix — CanIsoNet serves precomputed isoform switching events, not primary quantification
- The isoform of interest falls in one of the 27 PCAWG cancer types only, where the cross-disease panel should instead compare against other PCAWG disease types rather than the four non-cancer RNA-Seq datasets
- The question is functional/pathogenic impact of a switch (STRING network disruption, ClinVar annotation) rather than its sample-level frequency and cross-disease specificity

## Inputs

- CanIsoNet per-sample dMDT/isoform switching table for the index disease dataset (e.g. Alzheimer's Disease, GSE203206) with expression values per sample
- CanIsoNet per-sample isoform switching tables for the comparison disease datasets (OIH, Parkinson's Disease, Behcet's Disease)
- Target isoform identifiers (e.g. HSPH1-001/HSPH1-002, SLC25A25-005) and expected transcript lengths (858 AA vs 814 AA)

## Outputs

- Switch frequency for the index disease dataset (fraction of samples with the event, e.g. 21%)
- Per-disease presence/absence table for the target isoform (e.g. 11/39 Alzheimer's samples, absent in OIH, Parkinson's Disease, Behcet's Disease)
- Summary CSV (e.g. alzheimer_dmdt_frequencies.csv) recording the computed frequencies

## How to apply

Query CanIsoNet (caniso.net) for the disease-type specific dMDT and the per-sample isoform switching table, which lists each switch event with expression values. For the index disease dataset (e.g. Alzheimer's Disease, GSE203206), count samples showing the event of interest and divide by the total sample count to get the switch frequency (expected 21% for the HSPH1-001→HSPH1-002 switch; 11 of 39 samples for SLC25A25-005). Then repeat the lookup in the per-sample tables of the other disease datasets in the database — OIH, Parkinson's Disease and Behcet's Disease — to confirm the isoform is absent there. Cross-disease absence of the event is the evidence that the switch is disease-specific rather than a generic isoform expression pattern.

## Related tools

- **CanIsoNet** (Web database queried for disease-type specific dMDTs and per-sample isoform switching events with expression values across the OIH, Alzheimer's, Parkinson's and Behcet's Disease datasets) — https://github.com/kahramanlab/CanIsoNet_Web

## Evaluation signals

- Computed HSPH1-001→HSPH1-002 switch frequency equals the paper-reported 21% of analysed Alzheimer's samples
- SLC25A25-005 is detected in exactly 11 of 39 Alzheimer's samples
- SLC25A25-005 is absent from the per-sample tables of the OIH, Parkinson's Disease and Behcet's Disease datasets
- HSPH1 isoform identity matches the expected protein lengths: canonical 858 AA HSPH1-001 vs shorter 814 AA HSPH1-002
- Summary CSV (alzheimer_dmdt_frequencies.csv) contains frequencies that match the paper-reported values for both events

## Limitations

- Specificity claims are limited to the four non-cancer RNA-Seq datasets (OIH, Alzheimer's, Parkinson's, Behcet's) plus 27 PCAWG cancer types available in CanIsoNet — absence elsewhere in the database does not guarantee absence in unanalysed diseases
- CanIsoNet annotations cover a fixed set of isoforms and events (11,811 isoforms and 14,357 unique switching events across 31 disease types per the paper; 11,528 isoforms per the README), so unlisted isoforms cannot be checked
- Per-sample frequencies depend on the sample counts of the source datasets (e.g. 39 Alzheimer's samples); reanalysis of the raw data is outside CanIsoNet's scope

## Evidence

- [intro] CanIsoNet provides followings to the users: 1) Browsing various statistics on disease-specific isoform switching events in each diseases: [evidence span withheld — anchor status: not_found]
- [readme] four publicly available RNA-Seq datasets on OIH, Alzheimer's Disease, Parkinson's Disease and Behcet's Disease: "four publicly available RNA-Seq datasets on OIH, Alzheimer's Disease, Parkinson's Disease and Behcet's Disease"
- [readme] Exploring the isoform switching event in each sample with their expression values: "4) Exploring the isoform switching event in each sample with their expression values. <br>"
- [abstract] Various annotations for 11 811 isoforms and 14 357 unique isoform switching events across 31 different disease types are available.: "Various annotations for 11 811 isoforms and 14 357 unique isoform switching events across 31 different disease types are available."
