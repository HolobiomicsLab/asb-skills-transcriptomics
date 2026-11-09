---
name: transcript-isoform-identifier-resolution
description: Use when a dMDT entry or isoform switching event is named by gene code
  and transcript number (e.g. HSPH1-002) and the corresponding ENST accession must
  be resolved before querying CanIsoNet or STRING.
when_to_use_negative:
- The input is a gene symbol only, with no isoform-level resolution available or required
  (CanIsoNet analysis is isoform-specific).
- The isoform switching event is not annotated in CanIsoNet's 31 disease types (27
  PCAWG cancer types plus OIH, Alzheimer's, Parkinson's, Behcet's disease datasets).
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_0114
- http://edamontology.org/topic_3524
tools:
- name: CanIsoNet
  role: Web database used to browse and search the dMDT entry and confirm the isoform
    switching event and its transcript identifiers
  repo: https://github.com/kahramanlab/CanIsoNet_Web
- name: STRING
  role: Interaction network resource queried with the resolved transcript to retrieve
    interaction partners for disruption analysis
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbad050
    title: 'CanIsoNet: a database to study the functional impact of isoform switching
      events in diseases'
  source_content_hash: sha256:a5dd92cd2197fef0e306b77d8dc24e4dd5dafcd8ca52b13538672c0a969245ac
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kahramanlab/CanIsoNet_Web
  tools:
  - CanIsoNet
  - STRING
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_0114
  - http://edamontology.org/topic_3524
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# transcript-isoform-identifier-resolution

## Summary

Resolve a disease-specific isoform switching event (dMDT) to its exact transcript identifiers (e.g. HSPH1-002 / ENST00000380405) and locate it in the CanIsoNet web resource so downstream interaction-disruption analysis queries the correct isoform.

## When to use

Use when a dMDT entry or isoform switching event is named by gene code and transcript number (e.g. HSPH1-002) and the corresponding ENST accession must be resolved before querying CanIsoNet or STRING.

## When NOT to use

- The input is a gene symbol only, with no isoform-level resolution available or required (CanIsoNet analysis is isoform-specific).
- The isoform switching event is not annotated in CanIsoNet's 31 disease types (27 PCAWG cancer types plus OIH, Alzheimer's, Parkinson's, Behcet's disease datasets).

## Inputs

- dMDT entry label from CanIsoNet (gene name + isoform code, e.g. HSPH1-002)
- Ensembl transcript accession (ENST identifier, e.g. ENST00000380405)

## Outputs

- Resolved transcript identifier pair (dMDT code + ENST accession) usable for CanIsoNet and STRING queries
- Confirmed CanIsoNet dMDT entry for the disease type of interest (e.g. Alzheimer's disease)

## How to apply

Take the dMDT label from the CanIsoNet entry (gene name plus isoform code, e.g. HSPH1-002) and map it to its Ensembl transcript accession (ENST00000380405 for HSPH1-002); both identifiers should be carried together because CanIsoNet indexes isoform switching events by transcript while STRING interaction data are retrieved per transcript. Query the CanIsoNet web resource at caniso.net for the dMDT entry to confirm the transcript is among the annotated 11,811 isoforms and 14,357 unique isoform switching events across 31 disease types. Once resolved, the identifier pair anchors subsequent steps: retrieving STRING interaction partners, filtering to high-confidence edges (combined score > 900), and comparing Pfam domain composition between the dominant and switched isoform. Validation requires that the resolved transcript is the Alzheimer's disease dMDT entry and that downstream queries return the reported 44 high-quality STRING interactions.

## Related tools

- **CanIsoNet** (Web database used to browse and search the dMDT entry and confirm the isoform switching event and its transcript identifiers) — https://github.com/kahramanlab/CanIsoNet_Web
- **STRING** (Interaction network resource queried with the resolved transcript to retrieve interaction partners for disruption analysis)

## Evaluation signals

- The dMDT code and ENST accession both resolve to the same gene/transcript pair (e.g. HSPH1-002 = ENST00000380405).
- The resolved transcript appears as a dMDT entry for the expected disease type in CanIsoNet (Alzheimer's disease for HSPH1-002).
- Downstream STRING query with the resolved transcript returns the expected 44 high-quality interactions (combined score > 900).
- The resolved event is one of the 14,357 unique isoform switching events annotated in CanIsoNet.

## Limitations

- Resolution depends on the transcript being annotated in CanIsoNet; the database covers 11,811 isoforms across 31 disease types and entries outside these will not resolve.
- Identifier versions may differ between CanIsoNet, STRING, and Ensembl releases, so the exact ENST version should be checked when cross-referencing.

## Evidence

- [abstract] CanIsoNet is freely available at https://www.caniso.net: "CanIsoNet is freely available at https://www.caniso.net"
- [abstract] Various annotations for 11 811 isoforms and 14 357 unique isoform switching events across 31 different disease types are available.: "Various annotations for 11 811 isoforms and 14 357 unique isoform switching events across 31 different disease types are available."
- [other] Query the CanIsoNet web resource at caniso.net for the dMDT entry HSPH1-002 / ENST00000380405: [evidence span withheld — anchor status: not_found]
- [intro] four publicly available RNA-Seq datasets on OIH, Alzheimer's Disease, Parkinson's Disease and Behcet's Disease: "four publicly available RNA-Seq datasets on OIH, Alzheimer's Disease, Parkinson's Disease and Behcet's Disease"
