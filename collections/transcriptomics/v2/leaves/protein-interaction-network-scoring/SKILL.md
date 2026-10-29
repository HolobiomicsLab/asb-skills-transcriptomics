---
name: protein-interaction-network-scoring
description: Use when quantifying a protein's relative connectivity in the human STRING
  interaction network, or when scoring how isoform switching events disrupt protein-protein
  interactions in disease RNA-seq data.
when_to_use_negative:
- The question concerns differential gene or transcript expression alone (e.g. raw
  DE analysis) with no need to place events on an interaction network
- The target proteome is not covered by the human STRING network used by CanIsoNet
  (which annotates 31 human disease types from PCAWG and four RNA-Seq datasets)
- Input data is already a curated interaction-impact results table conforming to the
  CanIsoNet schema — no rescoring is needed
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_0128
tools:
- name: STRING
  role: Source of the human protein-protein interaction network with combined interaction
    scores on which network density and isoform-specific interaction disruption are
    computed and visualized
- name: CanIsoNet
  role: Pipeline and database (github.com/kahramanlab/CanIsoNet_Web, CC-BY-4.0) that
    calls dMDT events from Kallisto TPMs and scores their functional impact on STRING
    interactions; freely available at caniso.net
  repo: https://github.com/kahramanlab/CanIsoNet_Web
- name: Kallisto
  role: Quantifies transcript/isoform-level TPM abundances per sample as input for
    dMDT isoform switching calls
provenance:
  source_task_ids:
  - task_004
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbad050
    title: 'CanIsoNet: a database to study the functional impact of isoform switching
      events in diseases'
  source_content_hash: sha256:e8c211c5862e5aa32234598a1999652c064360bcc6cd7d30abad5a956f686f46
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kahramanlab/CanIsoNet_Web
  tools:
  - CanIsoNet
  - Kallisto
  - STRING
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_0128
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# protein-interaction-network-scoring

## Summary

Scores proteins and isoform-switching events within the human STRING protein-protein interaction network — e.g. the weighted network density score (HSPH1 = 78%) and the pathogenic impact of dMDT isoform switching events on their interactions, as implemented by CanIsoNet.

## When to use

Use when quantifying a protein's relative connectivity in the human STRING interaction network, or when scoring how isoform switching events disrupt protein-protein interactions in disease RNA-seq data.

## When NOT to use

- The question concerns differential gene or transcript expression alone (e.g. raw DE analysis) with no need to place events on an interaction network
- The target proteome is not covered by the human STRING network used by CanIsoNet (which annotates 31 human disease types from PCAWG and four RNA-Seq datasets)
- Input data is already a curated interaction-impact results table conforming to the CanIsoNet schema — no rescoring is needed

## Inputs

- Human STRING protein-protein interaction network with combined interaction scores
- Kallisto TPM quantifications of transcript/isoform abundances per sample (disease and control FASTQ from GEO/ENA)
- dMDT isoform switching event calls (gene, participating isoforms, switching statistics)
- ClinVar annotations and PFAM domain IDs

## Outputs

- hsph1_network_density.csv with computed and reported network density percentages side by side
- Per-event interaction disruption scores on the isoform-specific STRING network, with ClinVar-annotated genes highlighted and PFAM domain IDs of disrupted interactions
- Per-event and per-sample results table (isoform expression values, switching statistics, impact scores) matching the CanIsoNet database schema for the 31 disease types at caniso.net

## How to apply

Download the human STRING protein-protein interaction network with combined interaction scores; for a target protein (e.g. HSPH1), extract all its interactions and sum the interaction weights, then compute the same weighted sum for every protein, take the network-wide maximum, and normalize the target score by that maximum, expressing it as a percentage (network density score). Validate against the reported 78% for HSPH1 from Fig. 1C of Kahraman et al., Scientific Reports 2020, accepting within ±1% and writing computed vs reported values to hsph1_network_density.csv. For isoform-switching impact, call dMDT events from Kallisto TPM quantifications by comparing isoform usage between disease and control samples using the CanIsoNet pipeline code (github.com/kahramanlab/CanIsoNet_Web, CC-BY-4.0), then overlay each event on the isoform-specific interaction network built on STRING, highlighting ClinVar-annotated genes and reporting PFAM domain IDs for disrupted interactions. Assemble per-event and per-sample results (isoform expression values, switching statistics, interaction disruption scores) into a table conforming to the CanIsoNet schema used for its 31 hosted disease types.

## Related tools

- **STRING** (Source of the human protein-protein interaction network with combined interaction scores on which network density and isoform-specific interaction disruption are computed and visualized)
- **CanIsoNet** (Pipeline and database (github.com/kahramanlab/CanIsoNet_Web, CC-BY-4.0) that calls dMDT events from Kallisto TPMs and scores their functional impact on STRING interactions; freely available at caniso.net) — https://github.com/kahramanlab/CanIsoNet_Web
- **Kallisto** (Quantifies transcript/isoform-level TPM abundances per sample as input for dMDT isoform switching calls)

## Evaluation signals

- Computed HSPH1 network density score equals the reported 78% within ±1% tolerance
- hsph1_network_density.csv exists and contains computed and reported values side by side
- Per-event results table contains required fields (gene, participating isoforms, switching statistics, impact scores) and conforms to the CanIsoNet schema used for the 31 disease types
- Disrupted interactions carry PFAM domain IDs, and ClinVar-annotated genes are highlighted in the interactive interaction networks
- New-dataset output includes per-sample isoform expression values and can sit alongside the existing 31 disease types hosted at caniso.net

## Limitations

- The network density score is defined relative to the network-wide maximum weighted interaction sum, so its percentage depends on the current human STRING network version and on normal (not switched) isoform connectivity
- CanIsoNet covers 31 disease types (27 PCAWG cancer types plus OIH, Alzheimer's, Parkinson's and Behcet's RNA-Seq datasets); other diseases require a full new analysis run
- Annotation counts differ between sources — the article reports 11,811 isoforms annotated while the README reports 11,528 isoforms (both with 14,357 unique events) — so event counts should be checked against the current release

## Evidence

- [intro] CanIsoNet is the first webserver that merges isoform switching events in different disease types with their functional impact level and showing them on STRING interaction network: "CanIsoNet is the first webserver that merges isoform switching events in different disease types with their functional impact level and showing them on STRING interaction network"
- [abstract] incorporates isoform expression data with STRING interaction networks and ClinVar annotations to predict the pathogenic impact of isoform switching events: "CanIsoNet is the first webserver that incorporates isoform expression data with STRING interaction networks and ClinVar annotations to predict the pathogenic impact of isoform switching events"
- [abstract] the genes annotated in ClinVar are highlighted in interactive interaction networks: "the genes annotated in ClinVar are highlighted in interactive interaction networks"
- [readme] It holds annotations for 11,528 isoforms and 14,357 unique isoform switching events across 31 diseases.: "It holds annotations for 11,528 isoforms and 14,357 unique isoform switching events across 31 diseases."
- [abstract] Various annotations for 11 811 isoforms and 14 357 unique isoform switching events across 31 different disease types are available.: "Various annotations for 11 811 isoforms and 14 357 unique isoform switching events across 31 different disease types are available."
