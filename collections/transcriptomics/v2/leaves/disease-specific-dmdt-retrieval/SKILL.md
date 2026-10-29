---
name: disease-specific-dmdt-retrieval
description: Use when a transcript isoform switching event (e.g. an ENST identifier
  such as HSPH1-002 / ENST00000380405 in Alzheimer's disease) must be queried in CanIsoNet
  and its disrupted STRING interactions quantified.
when_to_use_negative:
- The gene or transcript is outside the 31 disease types covered by CanIsoNet (27
  PCAWG cancer types plus OIH, Alzheimer's, Parkinson's and Behçet's) — no dMDT annotation
  exists there
- The goal is general protein-protein interaction analysis without an isoform switching
  event — use plain STRING rather than the dMDT/isoform-switching layer
- You need isoform switching events in diseases not annotated by ClinVar or the loaded
  RNA-Seq datasets — CanIsoNet covers its indexed disease types only
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0602
tools:
- name: CanIsoNet
  role: Web database queried for the disease-type specific dMDT entry, its disrupted
    STRING network, Pfam domain IDs of disrupted interactions, and per-sample isoform
    switching events with expression values
  repo: https://github.com/kahramanlab/CanIsoNet_Web
- name: STRING
  role: Source of protein-protein interaction networks; edges are filtered to high
    quality using the STRING combined score threshold > 900 before domain-domain analysis
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbad050
    title: 'CanIsoNet: a database to study the functional impact of isoform switching
      events in diseases'
  source_content_hash: sha256:f040060424de19dd3d217e8b8b8a2d159ad6adb251873a82218e43226133292a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kahramanlab/CanIsoNet_Web
  tools:
  - CanIsoNet
  - STRING
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# disease-specific-dmdt-retrieval

## Summary

Retrieves and annotates disease-type specific dMDTs (disease-specific major disrupted transcripts) and their isoform switching events from CanIsoNet, then characterizes the functional impact of a given dMDT by filtering its STRING interaction network and identifying domain-mediated interactions lost by the switch.

## When to use

Use when a transcript isoform switching event (e.g. an ENST identifier such as HSPH1-002 / ENST00000380405 in Alzheimer's disease) must be queried in CanIsoNet and its disrupted STRING interactions quantified.

## When NOT to use

- The gene or transcript is outside the 31 disease types covered by CanIsoNet (27 PCAWG cancer types plus OIH, Alzheimer's, Parkinson's and Behçet's) — no dMDT annotation exists there
- The goal is general protein-protein interaction analysis without an isoform switching event — use plain STRING rather than the dMDT/isoform-switching layer
- You need isoform switching events in diseases not annotated by ClinVar or the loaded RNA-Seq datasets — CanIsoNet covers its indexed disease types only

## Inputs

- dMDT transcript identifier (e.g. HSPH1-002 / ENST00000380405) from CanIsoNet
- disease-type context (one of 31 disease types: 27 PCAWG cancer types or OIH, Alzheimer's, Parkinson's, Behçet's disease RNA-Seq datasets)
- STRING interaction network with combined score per edge
- 3did catalog of known Pfam domain-domain interactions
- Pfam domain composition of the dominant and switched isoforms

## Outputs

- High-quality STRING interaction set for the dMDT (combined score > 900; e.g. 44 interactions for HSPH1-002)
- Table of lost physical (domain-domain-mediated) interactions (e.g. 12 edges for HSPH1-002, including DNAJB2 and DNAJB6)
- Pfam domain IDs of disrupted interactions
- Per-sample isoform switching event annotations with expression values

## How to apply

Query the CanIsoNet web resource at caniso.net for the dMDT entry of interest (e.g. HSPH1-002 / ENST00000380405 in the Alzheimer's disease dataset) and retrieve its STRING interaction partners. Filter the interaction edges to high-confidence interactions using the STRING combined score threshold > 900 to yield the high-quality interaction set (e.g. 44 interactions for HSPH1-002). Cross-reference the filtered interaction list against the 3did catalog of Pfam domain-domain interactions to flag edges mediated by known domain-domain pairs. Compare the Pfam domain composition of the dominant versus the switched isoform to determine which domain-mediated physical interactions are lost by the switch, and compile the lost edges (e.g. 12 for HSPH1-002, including DNAJB2 and DNAJB6) into a table. Rationale: domain-domain evidence (3did/Pfam) distinguishes physical interactions plausibly disrupted by isoform switching from merely high-scoring STRING edges.

## Related tools

- **CanIsoNet** (Web database queried for the disease-type specific dMDT entry, its disrupted STRING network, Pfam domain IDs of disrupted interactions, and per-sample isoform switching events with expression values) — https://github.com/kahramanlab/CanIsoNet_Web
- **STRING** (Source of protein-protein interaction networks; edges are filtered to high quality using the STRING combined score threshold > 900 before domain-domain analysis)

## Evaluation signals

- The filtered high-confidence interaction count matches the reported value (44 high-quality STRING interactions at combined score > 900 for HSPH1-002)
- The compiled lost physical interaction table contains exactly 12 edges, including interactions with DNAJB2 and DNAJB6
- Every edge flagged as lost is supported by a known 3did Pfam domain-domain interaction and the domain is absent from the switched isoform's Pfam composition
- Retrieved dMDT entries fall within CanIsoNet's coverage of 31 disease types and the 14,357 unique isoform switching events reported

## Limitations

- Coverage is limited to 31 disease types (27 PCAWG cancer types plus four RNA-Seq datasets: OIH, Alzheimer's, Parkinson's, Behçet's); other diseases are not annotated
- Isoform annotation counts are inconsistent between sources: the article reports 11,811 isoforms annotated while the repository README reports 11,528 isoforms (both with 14,357 unique isoform switching events)
- Access is via the web interface at caniso.net; no CLI/API endpoint is documented in the article or README, so programmatic retrieval requires manual or ad hoc querying

## Evidence

- [abstract] Here, we present CanIsoNet, a database to view, browse and search isoform switching events in diseases.: "Here, we present CanIsoNet, a database to view, browse and search isoform switching events in diseases."
- [intro] CanIsoNet provides followings to the users: 1) Browsing various statistics on disease-specific isoform switching events in each diseases: [evidence span withheld — anchor status: not_found]
- [readme] Exploring and visualizing network disruptions in the STRING interaction network.: "2) Exploring and visualizing network disruptions in the STRING interaction network."
- [abstract] CanIsoNet is freely available at https://www.caniso.net.: "CanIsoNet is freely available at https://www.caniso.net."
- [intro] four publicly available RNA-Seq datasets on OIH, Alzheimer's Disease, Parkinson's Disease and Behcet's Disease: "four publicly available RNA-Seq datasets on OIH, Alzheimer's Disease, Parkinson's Disease and Behcet's Disease"
