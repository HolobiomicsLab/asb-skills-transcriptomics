---
name: pfam-domain-domain-interaction-annotation
description: Use when a transcript isoform switching event must be characterized at
  the interaction level — specifically when deciding which of a transcript's STRING
  edges are physical, domain-mediated interactions disrupted by the switch.
when_to_use_negative:
- The input is an isoform switching event without STRING network annotations in CanIsoNet,
  so no combined-score-filtered partner edges can be retrieved.
- The question is about expression-level or clinical impact of isoform switching rather
  than loss of physical, domain-mediated interactions (CanIsoNet also covers ClinVar
  and per-sample expression views).
- The interaction partners of interest lack Pfam domain annotations, since the physical-interaction-loss
  determination depends on Pfam domain-domain evidence from 3did.
edam_operation: http://edamontology.org/operation_0276
edam_topics:
- http://edamontology.org/topic_0128
tools:
- name: CanIsoNet
  role: Web database queried for the disease-specific dMDT entry and its isoform switching
    event, providing PFAM domain IDs of disrupted interactions and the STRING network
    view
  repo: https://github.com/kahramanlab/CanIsoNet_Web
- name: STRING
  role: Source of the transcript's interaction network; edges are filtered to high-quality
    (combined score > 900) interaction partners
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbad050
    title: 'CanIsoNet: a database to study the functional impact of isoform switching
      events in diseases'
  source_content_hash: sha256:90b16e79f95feeb127c9e1bff19309aa42e39dcea9c50d8f971405c7d030ad90
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kahramanlab/CanIsoNet_Web
  tools:
  - CanIsoNet
  - STRING
  edam_operation: http://edamontology.org/operation_0276
  edam_topics:
  - http://edamontology.org/topic_0128
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# pfam-domain-domain-interaction-annotation

## Summary

Annotates STRING interaction edges of an isoform-switched transcript with Pfam domain-domain interaction evidence (3did catalog) to determine which physical interactions are lost by an isoform switching event, as practiced in CanIsoNet's functional impact analysis for disease dMDTs (e.g., Alzheimer's HSPH1-002).

## When to use

Use when a transcript isoform switching event must be characterized at the interaction level — specifically when deciding which of a transcript's STRING edges are physical, domain-mediated interactions disrupted by the switch.

## When NOT to use

- The input is an isoform switching event without STRING network annotations in CanIsoNet, so no combined-score-filtered partner edges can be retrieved.
- The question is about expression-level or clinical impact of isoform switching rather than loss of physical, domain-mediated interactions (CanIsoNet also covers ClinVar and per-sample expression views).
- The interaction partners of interest lack Pfam domain annotations, since the physical-interaction-loss determination depends on Pfam domain-domain evidence from 3did.

## Inputs

- CanIsoNet dMDT entry for an isoform switching event (e.g., transcript ID ENST00000380405 / HSPH1-002) with STRING interaction partners
- STRING interaction network edges with combined scores for the transcript's partner proteins
- Pfam domain composition of the dominant and switched isoforms
- 3did catalog of Pfam domain-domain interactions

## Outputs

- List of high-quality STRING interactions passing combined score > 900 (e.g., 44 edges for HSPH1-002)
- Flagged subset of edges mediated by known 3did Pfam domain-domain pairs
- Table of lost physical interactions caused by the isoform switch (e.g., 12 edges including DNAJB2 and DNAJB6)

## How to apply

Start from the CanIsoNet (caniso.net) entry for the disease-specific dominant transcript (dMDT), e.g. HSPH1-002 / ENST00000380405, and retrieve its STRING interaction partners. Filter the edges to high-quality interactions using the STRING combined score threshold > 900, which for HSPH1-002 yields 44 high-quality interactions. Cross-reference the filtered edge list against the 3did catalog of Pfam domain-domain interactions to flag which edges are mediated by known domain-domain pairs, using the PFAM domain IDs of disrupted interactions annotated in CanIsoNet. Compare the Pfam domain composition of the dominant versus the switched isoform to determine which domain-mediated physical interactions disappear in the switched isoform. Compile the lost physical interactions into a table and validate counts against the published result (12 lost physical interactions, including DNAJB2 and DNAJB6).

## Related tools

- **CanIsoNet** (Web database queried for the disease-specific dMDT entry and its isoform switching event, providing PFAM domain IDs of disrupted interactions and the STRING network view) — https://github.com/kahramanlab/CanIsoNet_Web
- **STRING** (Source of the transcript's interaction network; edges are filtered to high-quality (combined score > 900) interaction partners)

## Evaluation signals

- Filtering STRING edges at combined score > 900 reproduces the published 44 high-quality interactions for HSPH1-002.
- The compiled lost-physical-interaction table contains exactly 12 edges.
- The lost physical interactions include edges to DNAJB2 and DNAJB6.
- Every compiled lost physical interaction maps to a Pfam domain-domain pair present in the 3did catalog.
- The dominant vs. switched isoform domain-composition comparison accounts for all 12 lost edges (no unexplained losses).

## Limitations

- Loss of physical interactions can only be established for edges supported by known Pfam domain-domain interactions in 3did; interactions lacking domain-domain evidence cannot be classified this way.
- STRING combined scores include non-physical associations, so the > 900 filter alone does not guarantee a physical interaction — the 3did/Pfam cross-reference is required.
- Analysis is scoped to the disease types annotated in CanIsoNet (27 PCAWG cancer types plus OIH, Alzheimer's, Parkinson's and Behcet's disease RNA-Seq datasets).

## Evidence

- [abstract] PFAM domain IDs of disrupted interactions: "PFAM domain IDs of disrupted interactions"
- [intro] the first webserver that merges isoform switching events in different disease types with their functional impact level and showing them on STRING interaction network: "the first webserver that merges isoform switching events in different disease types with their functional impact level and showing them on STRING interaction network"
- [readme] Exploring and visualizing network disruptions in the STRING interaction network.: "Exploring and visualizing network disruptions in the STRING interaction network."
- [abstract] Here, we present CanIsoNet, a database to view, browse and search isoform switching events in diseases.: "Here, we present CanIsoNet, a database to view, browse and search isoform switching events in diseases."
