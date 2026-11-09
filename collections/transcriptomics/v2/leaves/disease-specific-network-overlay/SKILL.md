---
name: disease-specific-network-overlay
description: Use when RNA-seq transcriptome data for a disease has yielded isoform
  switching events / dMDT genes with lost protein domains, and you need to map those
  domain losses onto a protein interaction network to expose functional disruptions.
when_to_use_negative:
- The analysis goal is general protein-protein interaction lookup without isoform
  switching or domain-loss context — a plain STRING query suffices
- Input transcript data lack isoform-level resolution (e.g., gene-level counts only),
  since dMDT identification requires isoform switching events per sample
- Studying non-human/non-mouse species, since the DDI-to-gene mapping described covers
  human and mouse genomes
edam_operation: http://edamontology.org/operation_0276
edam_topics:
- http://edamontology.org/topic_0128
- http://edamontology.org/topic_0659
- http://edamontology.org/topic_3308
tools:
- name: CanIsoNet
  role: Web interface and database that hosts and displays the disease-specific isoform
    interaction networks, merging isoform switching events with functional impact
    on STRING networks and highlighting ClinVar-annotated genes
  repo: https://github.com/kahramanlab/CanIsoNet_Web
- name: STRING
  role: Source of protein-protein interaction network (v10.0) onto which disrupted
    domain interactions are overlaid per disease
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioadv/vbad050
    title: 'CanIsoNet: a database to study the functional impact of isoform switching
      events in diseases'
  source_content_hash: sha256:87ed8cea5ff7e43194a734c16622d0492d0a0d655fb49d36124f514e1e9ea1f5
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
  - http://edamontology.org/topic_0659
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# disease-specific-network-overlay

## Summary

Constructs disease-specific isoform interaction networks by intersecting RNA-seq-derived dMDT gene lists (isoforms losing Pfam domains) with 3did Pfam domain-domain interactions and overlaying the disrupted edges onto STRING protein-protein interaction networks, per disease type.

## When to use

Use when RNA-seq transcriptome data for a disease has yielded isoform switching events / dMDT genes with lost protein domains, and you need to map those domain losses onto a protein interaction network to expose functional disruptions.

## When NOT to use

- The analysis goal is general protein-protein interaction lookup without isoform switching or domain-loss context — a plain STRING query suffices
- Input transcript data lack isoform-level resolution (e.g., gene-level counts only), since dMDT identification requires isoform switching events per sample
- Studying non-human/non-mouse species, since the DDI-to-gene mapping described covers human and mouse genomes

## Inputs

- RNA-seq transcriptome data per disease yielding isoform switching events and dMDT gene lists (genes whose isoforms lose domains)
- 3did (v 2020_05) curated Pfam domain-domain interaction (DDI) pairs
- STRING (v10.0) protein-protein interaction data (protein-level identifiers)
- Pfam domain ID to gene-symbol mapping (human and mouse)

## Outputs

- Per-disease isoform interaction networks with disrupted edges annotated with PFAM domain IDs
- Single annotated edge-list table (disease_isoform_interaction_networks.tsv) suitable for loading into the CanIsoNet web interface
- Disease-specific dMDT gene sets and network disruption visualizations on the STRING interaction network

## How to apply

Parse the 3did (v 2020_05) database to extract curated Pfam domain-domain interaction (DDI) pairs and map each Pfam domain ID to the human (or mouse) genes carrying that domain. Retrieve STRING (v10.0) PPI data and map protein-level identifiers to gene symbols so the DDI and PPI layers can be joined at gene level. For each disease type (31 disease types in the paper: 27 PCAWG cancer types plus OIH, Alzheimer's, Parkinson's, and Behcet's RNA-Seq datasets), intersect the transcriptome-derived dMDT gene list with the 3did DDI partner pairs to flag interactions that can no longer occur in the disease isoform. Overlay these disrupted domain interactions onto the STRING PPI network per disease, annotating each disrupted edge with its PFAM domain IDs, and export all per-disease networks as a single annotated edge-list table (disease_isoform_interaction_networks.tsv) loadable into the CanIsoNet web interface.

## Related tools

- **CanIsoNet** (Web interface and database that hosts and displays the disease-specific isoform interaction networks, merging isoform switching events with functional impact on STRING networks and highlighting ClinVar-annotated genes) — https://github.com/kahramanlab/CanIsoNet_Web
- **STRING** (Source of protein-protein interaction network (v10.0) onto which disrupted domain interactions are overlaid per disease)

## Evaluation signals

- disease_isoform_interaction_networks.tsv exists and is a well-formed TSV (field_present check)
- Every disrupted edge in the exported table carries PFAM domain ID annotations (contains_substring check)
- Intersections only occur between dMDT genes and 3did DDI partner pairs joined at gene-symbol level after STRING identifier mapping
- One network is produced per disease type; counts should align with the paper's coverage (31 disease types; 14,357 unique isoform switching events; ~11,500–11,811 isoforms annotated)
- Disrupted edges overlay consistently onto STRING v10.0 PPI edges when visualized in CanIsoNet

## Limitations

- Coverage depends on 3did (v 2020_05) and STRING (v10.0) versions; DDI disruptions are only detectable for Pfam domain pairs curated in 3did
- Only isoform switching events that cause loss of Pfam domains (dMDTs) contribute disrupted edges; switches without domain loss are invisible to this overlay
- Disease scope is fixed to the 31 disease types analyzed (27 PCAWG cancer types plus OIH, Alzheimer's, Parkinson's, Behcet's); new diseases require re-running the RNA-seq analysis
- Protein-to-gene identifier mapping is required before joining layers, and identifier ambiguities can misalign DDI and PPI partners

## Evidence

- [other] Disease-specific isoform interaction networks for human and mouse were constructed by integrating RNAseq transcriptome data of different diseases with Pfam domain-domain interaction data from the 3did (v 2020_05) and the STRING (v10.0) protein–protein interaction databases.: [evidence span withheld — anchor status: not_found]
- [abstract] CanIsoNet is the first webserver that incorporates isoform expression data with STRING interaction networks and ClinVar annotations to predict the pathogenic impact of isoform switching events: "CanIsoNet is the first webserver that incorporates isoform expression data with STRING interaction networks and ClinVar annotations to predict the pathogenic impact of isoform switching events"
- [intro] We have recently analyzed the pathogenic impact of isoform switching events in 31 disease types covering 27 different cancer types from PCAWG project: "We have recently analyzed the pathogenic impact of isoform switching events in 31 disease types covering 27 different cancer types from PCAWG project"
- [readme] 2) Exploring and visualizing network disruptions in the STRING interaction network.<br>
3) Exploring the disease-type specific dMDTs.: "2) Exploring and visualizing network disruptions in the STRING interaction network.<br>
3) Exploring the disease-type specific dMDTs."
- [abstract] Various annotations for 11 811 isoforms and 14 357 unique isoform switching events across 31 different disease types are available.: "Various annotations for 11 811 isoforms and 14 357 unique isoform switching events across 31 different disease types are available."
