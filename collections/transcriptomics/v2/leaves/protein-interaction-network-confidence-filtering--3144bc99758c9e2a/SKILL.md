---
name: protein-interaction-network-confidence-filtering
description: 'Use when quantifying network disruption from an isoform switching event:
  STRING interaction partners must be filtered to high-quality edges (combined score
  > 900) before classifying which physical interactions are lost.'
when_to_use_negative:
- The disease or isoform switching event is not covered by CanIsoNet (its 31 disease
  types cover 27 PCAWG cancer types plus OIH, Alzheimer's, Parkinson's and Behcet's
  RNA-Seq datasets) — the dMDT entry will not exist to filter against
- You need de novo prediction of protein-protein interactions rather than confidence
  filtering of existing STRING edges
- You are analyzing interactions at the gene level where isoform domain composition
  differences are irrelevant
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0121
tools:
- name: CanIsoNet
  role: Web database queried for the disease-specific dMDT entry and its annotated
    disrupted STRING interactions and PFAM domain IDs
  repo: https://github.com/kahramanlab/CanIsoNet_Web
- name: STRING
  role: Source of the protein interaction network whose edges are filtered to high-quality
    interactions using the combined score > 900 threshold
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioadv/vbad050
    title: 'CanIsoNet: a database to study the functional impact of isoform switching
      events in diseases'
  source_content_hash: sha256:ca7091f8b357b196074e1cf0acbb179edf021a9322f5cb8b65112ac7875d27dc
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kahramanlab/CanIsoNet_Web
  tools:
  - CanIsoNet
  - STRING
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# protein-interaction-network-confidence-filtering

## Summary

Filters a protein interaction network to high-confidence edges — in the CanIsoNet workflow, STRING edges with combined score > 900 — before assessing which physical, domain-mediated interactions are lost by an isoform switching event (dMDT).

## When to use

Use when quantifying network disruption from an isoform switching event: STRING interaction partners must be filtered to high-quality edges (combined score > 900) before classifying which physical interactions are lost.

## When NOT to use

- The disease or isoform switching event is not covered by CanIsoNet (its 31 disease types cover 27 PCAWG cancer types plus OIH, Alzheimer's, Parkinson's and Behcet's RNA-Seq datasets) — the dMDT entry will not exist to filter against
- You need de novo prediction of protein-protein interactions rather than confidence filtering of existing STRING edges
- You are analyzing interactions at the gene level where isoform domain composition differences are irrelevant

## Inputs

- dMDT transcript identifier from CanIsoNet (e.g. HSPH1-002 / ENST00000380405) for a disease in the database
- STRING interaction list for the gene with combined scores per edge
- 3did catalog of Pfam domain-domain interactions
- Pfam domain composition of the dominant isoform and the switched isoform

## Outputs

- Filtered set of high-quality STRING interactions (combined score > 900; 44 edges for the HSPH1-002 example)
- Flags marking which filtered edges are mediated by known Pfam domain-domain pairs (3did)
- Table of lost physical interactions (12 edges for HSPH1-002, including DNAJB2 and DNAJB6)

## How to apply

Query the CanIsoNet web resource (caniso.net) for the disease-specific dMDT entry of interest, e.g. HSPH1-002 / ENST00000380405 for Alzheimer's disease. Retrieve the transcript's STRING interaction partners and retain only high-confidence edges by applying the STRING combined score threshold > 900; for HSPH1-002 this yields 44 high-quality interactions. Cross-reference the filtered edge list against the 3did catalog of Pfam domain-domain interactions to flag edges mediated by known domain-domain pairs (CanIsoNet annotates 'PFAM domain IDs of disrupted interactions'). Compare the Pfam domain composition of the dominant versus the switched isoform to determine which domain-mediated physical interactions are lost by the switch. Compile the lost physical interactions into a table; for HSPH1-002 the expected result is 12 lost physical edges including interactions with DNAJB2 and DNAJB6. The rationale for a strict score cutoff is that high combined scores enrich for edges supported by known Pfam domain-domain interactions, which are the edges most plausibly disrupted by loss of a protein domain.

## Related tools

- **CanIsoNet** (Web database queried for the disease-specific dMDT entry and its annotated disrupted STRING interactions and PFAM domain IDs) — https://github.com/kahramanlab/CanIsoNet_Web
- **STRING** (Source of the protein interaction network whose edges are filtered to high-quality interactions using the combined score > 900 threshold)

## Evaluation signals

- Filtered edge count matches the reported 44 high-quality STRING interactions for HSPH1-002 at combined score > 900
- Lost physical interaction table contains exactly 12 edges for the HSPH1-002 example
- DNAJB2 and DNAJB6 appear among the lost physical interaction partners
- Every retained edge has a STRING combined score > 900 (no low-confidence edges leak into the filtered set)
- Each domain-mediated lost edge is traceable to a known Pfam domain-domain pair in the 3did catalog

## Limitations

- STRING combined score is a confidence proxy, not experimental proof of physical association; the > 900 cutoff is the high-quality threshold used by CanIsoNet but edges near the cutoff remain uncertain
- Coverage is limited to isoform switching events annotated in CanIsoNet (11,528–11,811 isoforms, 14,357 unique events across 31 disease types)
- Domain-mediated loss detection depends on Pfam/3did coverage — interactions lacking annotated Pfam domain-domain pairs cannot be flagged as physical domain-mediated losses

## Evidence

- [abstract] CanIsoNet is the first webserver that incorporates isoform expression data with STRING interaction networks and ClinVar annotations to predict the pathogenic impact of isoform switching events: "CanIsoNet is the first webserver that incorporates isoform expression data with STRING interaction networks and ClinVar annotations to predict the pathogenic impact of isoform switching events"
- [intro] CanIsoNet merges isoform switching events with STRING interaction network: "the first webserver that merges isoform switching events in different disease types with their functional impact level and showing them on STRING interaction network"
- [readme] Exploring and visualizing network disruptions in the STRING interaction network: "2) Exploring and visualizing network disruptions in the STRING interaction network.<br>"
