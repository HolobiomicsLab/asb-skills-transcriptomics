---
name: string-ppi-data-integration
description: Use when you have disease isoform switching results (dMDT gene lists
  whose isoforms lose domains) and need to map those domain losses onto protein-protein
  interactions from STRING to identify disrupted edges.
when_to_use_negative:
- The goal is general PPI network analysis without isoform switching or domain-loss
  context — STRING alone suffices and the 3did DDI overlay adds nothing.
- Input lacks disease-specific dMDT gene lists (e.g., only variant-level or expression-level
  data without isoform-level domain loss), so no edges can be flagged as disrupted.
- You need isoform-specific PPIs measured directly (e.g., AP-MS) rather than domain-loss-inferred
  disruption of reference PPIs.
edam_operation: http://edamontology.org/operation_0276
edam_topics:
- http://edamontology.org/topic_0128
tools:
- name: STRING
  role: Source of the protein-protein interaction network onto which disrupted Pfam
    domain interactions are overlaid per disease type
- name: CanIsoNet
  role: Downstream web interface that loads the exported per-disease isoform interaction
    network edge lists for browsing and visualization
  repo: https://github.com/kahramanlab/CanIsoNet_Web
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbad050
    title: 'CanIsoNet: a database to study the functional impact of isoform switching
      events in diseases'
  source_content_hash: sha256:78404ea73e889e15590821f1185cf9c289a31fdfc185b4b1b69e7b117665faa9
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

# STRING PPI data integration

## Summary

Integrates STRING (v10.0) protein-protein interactions with 3did (v 2020_05) Pfam domain-domain interaction pairs to overlay isoform-driven domain losses from disease RNA-seq dMDT gene lists onto the PPI network, producing annotated disease-specific isoform interaction networks as consumed by CanIsoNet.

## When to use

Use when you have disease isoform switching results (dMDT gene lists whose isoforms lose domains) and need to map those domain losses onto protein-protein interactions from STRING to identify disrupted edges.

## When NOT to use

- The goal is general PPI network analysis without isoform switching or domain-loss context — STRING alone suffices and the 3did DDI overlay adds nothing.
- Input lacks disease-specific dMDT gene lists (e.g., only variant-level or expression-level data without isoform-level domain loss), so no edges can be flagged as disrupted.
- You need isoform-specific PPIs measured directly (e.g., AP-MS) rather than domain-loss-inferred disruption of reference PPIs.

## Inputs

- 3did (v 2020_05) Pfam domain-domain interaction (DDI) pairs
- STRING (v10.0) protein-protein interaction network
- RNA-seq transcriptome-derived dMDT gene lists per disease (genes whose isoforms lose domains), 31 disease types including 27 PCAWG cancer types
- Pfam domain ID to gene mappings

## Outputs

- disease-specific isoform interaction networks per disease type
- disease_isoform_interaction_networks.tsv: annotated edge-list table with PFAM domain IDs for disrupted interactions
- disrupted PPI edges flagged where a DDI partner pair can no longer occur in the disease isoform

## How to apply

Parse the 3did (v 2020_05) database to extract curated Pfam domain-domain interaction (DDI) pairs, then map each Pfam domain ID to the human genes carrying that domain. Retrieve STRING (v10.0) PPI data and normalize protein-level identifiers to gene symbols so the DDI and PPI layers can be joined at gene level. For each of the 31 disease types, intersect the RNA-seq transcriptome-derived dMDT gene list with the 3did DDI partner pairs to flag interactions that can no longer occur in the disease isoform. Overlay the disrupted domain interactions onto the STRING PPI network per disease, annotating each disrupted edge with its PFAM domain IDs, and export all per-disease networks as a single annotated edge-list table (disease_isoform_interaction_networks.tsv) suitable for loading into the CanIsoNet web interface.

## Related tools

- **STRING** (Source of the protein-protein interaction network onto which disrupted Pfam domain interactions are overlaid per disease type)
- **CanIsoNet** (Downstream web interface that loads the exported per-disease isoform interaction network edge lists for browsing and visualization) — https://github.com/kahramanlab/CanIsoNet_Web

## Evaluation signals

- disease_isoform_interaction_networks.tsv exists and is a well-formed TSV (field_present check).
- Disrupted interactions carry PFAM domain ID annotations (contains_substring check for PFAM IDs in the exported table).
- Each disrupted edge can be traced back to a 3did DDI partner pair intersected with a disease dMDT gene list.
- Per-disease networks cover the 31 disease types (27 PCAWG cancer types plus OIH, Alzheimer's, Parkinson's, Behcet's datasets) without empty gene-ID joins.
- STRING protein identifiers resolve to gene symbols consistently across the DDI and PPI layers (gene-level join yields no unmapped edges).

## Limitations

- Interaction disruption is inferred from Pfam domain loss on reference 3did/STRING edges, not from experimentally measured isoform-specific interactions, so edge disruption is a domain-based prediction.
- Disease coverage is fixed at 31 disease types (27 PCAWG cancer types plus four RNA-Seq datasets: OIH, Alzheimer's, Parkinson's, Behcet's); other diseases require re-running the pipeline with new dMDT lists.
- DDI and PPI layers must be joined at gene level, so isoform-level protein identifier ambiguity can lose or merge edges.

## Evidence

- [intro] CanIsoNet is the first webserver that merges isoform switching events in different disease types with their functional impact level and showing them on STRING interaction network: "CanIsoNet is the first webserver that merges isoform switching events in different disease types with their functional impact level and showing them on STRING interaction network"
- [abstract] CanIsoNet is the first webserver that incorporates isoform expression data with STRING interaction networks and ClinVar annotations to predict the pathogenic impact of isoform switching events: "CanIsoNet is the first webserver that incorporates isoform expression data with STRING interaction networks and ClinVar annotations to predict the pathogenic impact of isoform switching events"
- [abstract] PFAM domain IDs of disrupted interactions: "incorporates isoform expression data with STRING interaction networks and ClinVar annotations"
- [readme] Exploring and visualizing network disruptions in the STRING interaction network.: "2) Exploring and visualizing network disruptions in the STRING interaction network."
- [intro] 31 disease types covering 27 different cancer types from PCAWG project: "31 disease types covering 27 different cancer types from PCAWG project"
