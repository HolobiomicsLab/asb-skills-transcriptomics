---
name: pfam-domain-identifier-mapping
description: Use when building isoform-interaction or dMDT-based networks where transcript
  isoform switching loses Pfam domains, and you must connect Pfam DDI pairs to gene-level
  PPI networks to flag disrupted interactions.
when_to_use_negative:
- Your goal is generic gene-gene PPI analysis without an isoform/domain-loss dimension
  — plain STRING network construction suffices and no 3did DDI mapping is needed
- The isoform events of interest do not involve lost or disrupted Pfam domains (no
  dMDT genes), so DDI-based disruption flags cannot be defined
edam_operation: http://edamontology.org/operation_0289
edam_topics:
- http://edamontology.org/topic_0128
- http://edamontology.org/topic_3308
tools:
- name: CanIsoNet
  role: Web interface/database into which the per-disease isoform interaction networks
    are loaded; displays disrupted DDI edges annotated with PFAM domain IDs on the
    STRING network
  repo: https://github.com/kahramanlab/CanIsoNet_Web
- name: STRING
  role: Source of v10.0 protein-protein interaction data whose protein identifiers
    are mapped to gene symbols so the Pfam DDI layer can be overlaid per disease
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbad050
    title: 'CanIsoNet: a database to study the functional impact of isoform switching
      events in diseases'
  source_content_hash: sha256:b3ff096839f95dbc1fe5c9701f98f7f3d51a482fda81fd7561a705289ca2a9c2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kahramanlab/CanIsoNet_Web
  tools:
  - CanIsoNet
  - STRING
  edam_operation: http://edamontology.org/operation_0289
  edam_topics:
  - http://edamontology.org/topic_0128
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# pfam-domain-identifier-mapping

## Summary

Maps Pfam domain identifiers to human genes via the 3did (v2020_05) curated domain-domain interaction (DDI) database, then joins the DDI layer with STRING (v10.0) PPI data at gene-symbol level so isoform-driven domain losses can be annotated onto protein interaction networks as PFAM domain IDs.

## When to use

Use when building isoform-interaction or dMDT-based networks where transcript isoform switching loses Pfam domains, and you must connect Pfam DDI pairs to gene-level PPI networks to flag disrupted interactions.

## When NOT to use

- Your goal is generic gene-gene PPI analysis without an isoform/domain-loss dimension — plain STRING network construction suffices and no 3did DDI mapping is needed
- The isoform events of interest do not involve lost or disrupted Pfam domains (no dMDT genes), so DDI-based disruption flags cannot be defined

## Inputs

- 3did (v2020_05) database of curated Pfam domain-domain interaction (DDI) pairs
- STRING (v10.0) protein-protein interaction data (protein-level identifiers)
- Per-disease RNA-seq transcriptome-derived dMDT gene lists (genes whose isoforms lose domains) for 31 disease types (27 PCAWG cancer types plus OIH, Alzheimer's, Parkinson's, Behcet's disease)

## Outputs

- disease_isoform_interaction_networks.tsv — single annotated edge-list table of per-disease isoform interaction networks with PFAM domain IDs on disrupted edges
- Disease-specific isoform interaction networks displayed on the STRING interaction network in the CanIsoNet web interface

## How to apply

Parse the 3did (v2020_05) database to extract curated Pfam domain-domain interaction (DDI) pairs, mapping each Pfam domain ID to the human genes carrying the corresponding domain. Retrieve STRING (v10.0) protein-protein interaction data and map protein-level identifiers to gene symbols so the DDI and PPI layers can be joined at gene level. For each disease type, intersect the RNA-seq transcriptome-derived dMDT gene list (genes whose isoforms lose domains) with the 3did DDI partner pairs to flag interactions that can no longer occur in the disease isoform. Overlay the disrupted domain interactions onto the STRING PPI network per disease, annotating each disrupted edge with its PFAM domain IDs, and export per-disease networks as an annotated edge-list table (disease_isoform_interaction_networks.tsv) loadable into the CanIsoNet web interface.

## Related tools

- **CanIsoNet** (Web interface/database into which the per-disease isoform interaction networks are loaded; displays disrupted DDI edges annotated with PFAM domain IDs on the STRING network) — https://github.com/kahramanlab/CanIsoNet_Web
- **STRING** (Source of v10.0 protein-protein interaction data whose protein identifiers are mapped to gene symbols so the Pfam DDI layer can be overlaid per disease)

## Evaluation signals

- Exported disease_isoform_interaction_networks.tsv exists and is well-formed TSV (field_present check)
- Every flagged disrupted interaction carries PFAM domain ID annotations (contains_substring check)
- DDI and PPI layers join cleanly at gene-symbol level after identifier mapping (no unmatched protein IDs leaking into the edge list)
- Per-disease networks cover the expected 31 disease types (27 PCAWG cancer types plus OIH, Alzheimer's, Parkinson's, Behcet's disease)

## Limitations

- Pinned database versions matter: 3did v2020_05 and STRING v10.0; re-running with different versions will change DDI/PPI coverage and edge sets
- Disruption flags depend on the input dMDT gene lists; isoform switching events that do not lose annotated Pfam domains will not be captured
- Joining is performed at gene-symbol level, so multi-domain or many-to-many gene–protein mappings can introduce ambiguity in edge annotation

## Evidence

- [abstract] PFAM domain IDs of disrupted interactions: "PFAM domain IDs of disrupted interactions"
- [full_text] Disease-specific isoform interaction networks merged 3did Pfam and STRING data: [evidence span withheld — anchor status: not_found]
- [readme] README: exploring network disruptions in the STRING interaction network: "2) Exploring and visualizing network disruptions in the STRING interaction network."
- [readme] README: first webserver merging isoform switching with STRING networks: "CanIsoNet is the first webserver that merges isoform switching events in different disease types with their functional impact level and showing them on STRING interaction network."
