---
name: isoform-switch-frequency-quantification
description: Use when you need the per-sample prevalence of an isoform switching event
  in a CanIsoNet disease dataset (PCAWG cancers or the OIH, Alzheimer's, Parkinson's,
  Behcet's RNA-Seq datasets), e.g. 'what fraction of AD samples switch HSPH1-001 to
  HSPH1-002?'
when_to_use_negative:
- The disease or tissue of interest is not among the 31 disease types covered by CanIsoNet
  (27 PCAWG cancer types plus OIH, Alzheimer's, Parkinson's and Behcet's RNA-Seq datasets)
- The goal is to score network disruption or compute a STRING network density score
  for a gene (e.g. the HSPH1 78% density score) rather than per-sample switch frequency
- Only aggregate (not per-sample) isoform expression data are available, since frequency
  quantification requires the per-sample event table with expression values
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CanIsoNet
  role: Source of the disease-specific per-sample dMDT tables listing isoform switching
    events with expression values, browsed at caniso.net
  repo: https://www.caniso.net/
provenance:
  source_task_ids:
  - task_005
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbad050
    title: 'CanIsoNet: a database to study the functional impact of isoform switching
      events in diseases'
  source_content_hash: sha256:00b8e1d2a3ed076a545573b67ff159650b6b938ef349c39b4633b12e13e66697
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - CanIsoNet
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# isoform-switch-frequency-quantification

## Summary

Quantifies the fraction of disease samples carrying a specific isoform switch by mining the per-sample dMDT tables exposed by CanIsoNet (e.g. the Alzheimer's disease RNA-Seq dataset GSE203206), and validates computed frequencies against paper-reported values such as the 21% HSPH1-001-to-HSPH1-002 switch rate.

## When to use

Use when you need the per-sample prevalence of an isoform switching event in a CanIsoNet disease dataset (PCAWG cancers or the OIH, Alzheimer's, Parkinson's, Behcet's RNA-Seq datasets), e.g. 'what fraction of AD samples switch HSPH1-001 to HSPH1-002?'

## When NOT to use

- The disease or tissue of interest is not among the 31 disease types covered by CanIsoNet (27 PCAWG cancer types plus OIH, Alzheimer's, Parkinson's and Behcet's RNA-Seq datasets)
- The goal is to score network disruption or compute a STRING network density score for a gene (e.g. the HSPH1 78% density score) rather than per-sample switch frequency
- Only aggregate (not per-sample) isoform expression data are available, since frequency quantification requires the per-sample event table with expression values

## Inputs

- Per-sample dMDT table for the target disease dataset exported from CanIsoNet (caniso.net), with per-sample isoform switching events and expression values
- Disease dataset identifier (e.g. Alzheimer's disease RNA-Seq dataset GSE203206 within CanIsoNet)
- Transcript identifiers of the switch event (e.g. HSPH1-001/HSPH1-002) or isoform of interest (e.g. SLC25A25-005)

## Outputs

- Switch frequency (fraction/percentage of disease samples showing the event, e.g. 21% for HSPH1-001→HSPH1-002)
- Isoform prevalence count (e.g. 11 of 39 Alzheimer's samples express SLC25A25-005)
- Summary CSV (e.g. alzheimer_dmdt_frequencies.csv) with computed and paper-reported frequencies side by side

## How to apply

Access caniso.net and extract the disease-specific dMDT (disease-specific major determinant transcripts) table, which lists per-sample isoform switching events together with isoform expression values. For the event of interest (e.g. HSPH1-001, 858 AA, to the shorter 814 AA HSPH1-002), count the samples in which the switch occurs and divide by the total number of samples in that disease dataset to obtain the switch frequency. For isoform-level prevalence (e.g. SLC25A25-005), count samples expressing that isoform out of the disease sample total (expected 11 of 39 in Alzheimer's). Cross-check specificity by verifying the isoform is absent from the other disease datasets in the database (OIH, Parkinson's disease, Behcet's disease). Validate by comparing computed frequencies to the paper-reported values (21% for HSPH1; 11/39 for SLC25A25-005) and write computed and reported values side by side to alzheimer_dmdt_frequencies.csv.

## Related tools

- **CanIsoNet** (Source of the disease-specific per-sample dMDT tables listing isoform switching events with expression values, browsed at caniso.net) — https://www.caniso.net/

## Examples

```
python -c "import pandas as pd; df=pd.read_csv('alzheimer_dMDT_per_sample.csv'); s=df[df.event=='HSPH1-001_to_HSPH1-002']; print(len(s)/len(df))"
```

## Evaluation signals

- Computed HSPH1-001↔HSPH1-002 switch frequency in the Alzheimer's dataset equals the paper-reported 21%
- SLC25A25-005 is detected in 11 of 39 Alzheimer's samples and is absent from the OIH, Parkinson's disease and Behcet's disease datasets
- Transcript length check passes: HSPH1-001 is the 858 AA canonical transcript and HSPH1-002 the shorter 814 AA isoform
- Output CSV (alzheimer_dmdt_frequencies.csv) contains computed and reported values side by side with matching frequencies
- Denominator equals the total number of samples in the disease dataset (e.g. 39 Alzheimer's samples), not the number of switch events

## Limitations

- Frequency estimates are bounded by the four non-cancer RNA-Seq datasets (OIH, Alzheimer's, Parkinson's, Behcet's) and 27 PCAWG cancer types included in CanIsoNet; no other disease cohorts can be queried
- The README reports 11,528 annotated isoforms while the article abstract reports 11,811, so isoform coverage may differ between database versions
- Counts depend on the per-sample event calls stored in the dMDT tables; samples without switch calls are treated as non-switched, so low-expression isoforms may be undercounted

## Evidence

- [readme] Exploring the isoform switching event in each sample with their expression values.: "Exploring the isoform switching event in each sample with their expression values."
- [readme] Exploring the disease-type specific dMDTs.: "Exploring the disease-type specific dMDTs."
- [intro] four publicly available RNA-Seq datasets on OIH, Alzheimer's Disease, Parkinson's Disease and Behcet's Disease: "four publicly available RNA-Seq datasets on OIH, Alzheimer's Disease, Parkinson's Disease and Behcet's Disease"
- [methods] CanIsoNet is freely available at https://www.caniso.net.: "CanIsoNet is freely available at [caniso.net](https://www.caniso.net/)"
