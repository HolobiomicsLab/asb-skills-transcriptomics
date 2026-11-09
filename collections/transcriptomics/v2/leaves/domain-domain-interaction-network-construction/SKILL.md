---
name: domain-domain-interaction-network-construction
description: 'Use when linking transcript abundance changes (isoform switching) to
  protein interaction disruptions: you have dMDT gene lists per disease and want to
  flag PPI edges that can no longer occur because a Pfam domain is lost in the disease
  isoform.'
when_to_use_negative:
- The analysis targets single-nucleotide variants or ClinVar mutation annotation rather
  than isoform switching events — ClinVar genes are only highlighted in CanIsoNet
  networks, not the basis of the DDI/PPI join
- The input lacks RNA-seq-derived dMDT gene lists per disease type, so no disease-specific
  domain-loss intersection can be performed
- The goal is generic PPI network analysis on full proteomes rather than networks
  disrupted by disease isoforms
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0602
tools:
- name: CanIsoNet
  role: Web interface into which the assembled disease-specific isoform interaction
    networks are loaded for browsing and visualization of network disruptions
  repo: https://github.com/kahramanlab/CanIsoNet_Web
- name: STRING
  role: Source of protein-protein interaction edges (v10.0) onto which disrupted Pfam
    domain-domain interactions are overlaid per disease
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbad050
    title: 'CanIsoNet: a database to study the functional impact of isoform switching
      events in diseases'
  source_content_hash: sha256:81f56d0f4250c3823b94ff40ffcd0ea2a4d0a2dd14321c942f1c2aa59501da6d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kahramanlab/CanIsoNet_Web
  tools:
  - CanIsoNet
  - STRING
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# domain-domain-interaction-network-construction

## Summary

Constructs disease-specific isoform interaction networks by intersecting RNA-seq-derived dMDT gene lists with curated Pfam domain-domain interaction pairs from 3did and overlaying the disrupted domain interactions onto the STRING protein-protein interaction network, annotating each disrupted edge with its PFAM domain IDs.

## When to use

Use when linking transcript abundance changes (isoform switching) to protein interaction disruptions: you have dMDT gene lists per disease and want to flag PPI edges that can no longer occur because a Pfam domain is lost in the disease isoform.

## When NOT to use

- The analysis targets single-nucleotide variants or ClinVar mutation annotation rather than isoform switching events — ClinVar genes are only highlighted in CanIsoNet networks, not the basis of the DDI/PPI join
- The input lacks RNA-seq-derived dMDT gene lists per disease type, so no disease-specific domain-loss intersection can be performed
- The goal is generic PPI network analysis on full proteomes rather than networks disrupted by disease isoforms

## Inputs

- 3did (v 2020_05) curated Pfam domain-domain interaction pairs
- STRING (v10.0) protein-protein interaction data
- RNA-seq transcriptome-derived dMDT gene lists (genes whose isoforms lose domains) for 31 disease types
- PFAM domain-to-gene mappings

## Outputs

- Disease-specific isoform interaction networks for human and mouse
- Single annotated edge-list table (disease_isoform_interaction_networks.tsv) with PFAM domain IDs on disrupted interactions
- Per-disease STRING networks with disrupted edges highlighted for the CanIsoNet web interface

## How to apply

Parse the 3did (v 2020_05) database to extract curated Pfam domain-domain interaction (DDI) pairs and map each Pfam domain ID to the human genes carrying that domain. Retrieve STRING (v10.0) protein-protein interaction data and map protein-level identifiers to gene symbols so the DDI and PPI layers can be joined at gene level. For each of the 31 disease types (27 PCAWG cancer types plus OIH, Alzheimer's, Parkinson's and Behcet's RNA-seq datasets), intersect the RNA-seq transcriptome-derived dMDT gene list (genes whose isoforms lose domains) with the 3did DDI partner pairs to flag interactions that can no longer occur in the disease isoform. Overlay these disrupted domain interactions onto the per-disease STRING PPI network, annotating each disrupted edge with its PFAM domain IDs, and export all per-disease networks as a single annotated edge-list table (disease_isoform_interaction_networks.tsv) loadable into the CanIsoNet web interface. The gene-level join is the key decision point: DDI evidence from 3did is transferred to PPI edges in STRING only when both interaction partners share the corresponding domains.

## Related tools

- **CanIsoNet** (Web interface into which the assembled disease-specific isoform interaction networks are loaded for browsing and visualization of network disruptions) — https://github.com/kahramanlab/CanIsoNet_Web
- **STRING** (Source of protein-protein interaction edges (v10.0) onto which disrupted Pfam domain-domain interactions are overlaid per disease)

## Evaluation signals

- The exported disease_isoform_interaction_networks.tsv exists and is a well-formed TSV (field_present check)
- Every disrupted interaction row carries a PFAM domain ID annotation (contains_substring check on PFAM domain fields)
- All 31 disease types (27 PCAWG cancer types plus OIH, Alzheimer's, Parkinson's, Behcet's) are represented in the exported networks
- Each disrupted edge in a per-disease network corresponds to a gene present in that disease's dMDT gene list intersected with 3did DDI partner pairs
- Protein identifiers in the STRING layer resolve to gene symbols so DDI and PPI layers join without dangling nodes

## Limitations

- DDI and PPI layers are joined at gene level, so isoform-specific domain loss is inferred from dMDT membership rather than direct isoform-level interaction evidence
- Only genes carrying annotated Pfam domains (per 3did v 2020_05) can contribute disrupted edges; interactions without Pfam domain coverage are invisible
- Coverage is restricted to the 31 disease types with RNA-seq-derived dMDT lists (27 PCAWG cancer types plus four other RNA-seq datasets); annotation counts differ slightly between the paper (11,811 isoforms) and README (11,528 isoforms)

## Evidence

- [methods] Parse the 3did (v 2020_05) database to extract curated Pfam domain-domain interaction (DDI) pairs and map each Pfam domain ID to the human genes: [evidence span withheld — anchor status: not_found]
- [intro] CanIsoNet is the first webserver that merges isoform switching events in different disease types with their functional impact level and showing them on STRING interaction network: "CanIsoNet is the first webserver that merges isoform switching events in different disease types with their functional impact level and showing them on STRING interaction network"
- [abstract] incorporates isoform expression data with STRING interaction networks and ClinVar annotations: "incorporates isoform expression data with STRING interaction networks and ClinVar annotations"
- [readme] It holds annotations for 11,528 isoforms and 14,357 unique isoform switching events across 31 diseases.: "It holds annotations for 11,528 isoforms and 14,357 unique isoform switching events across 31 diseases."
- [intro] 31 disease types covering 27 different cancer types from PCAWG project: "31 disease types covering 27 different cancer types from PCAWG project"
