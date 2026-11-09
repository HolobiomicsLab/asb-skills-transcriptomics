---
name: dmdt-differential-transcript-usage-analysis
description: Use when you have disease and control transcriptome RNA-seq FASTQ data
  and want to identify isoform switching events and assess whether they disrupt protein
  interactions pathogenically.
when_to_use_negative:
- The data are not transcriptome-wide RNA-seq samples with a disease-vs-control grouping
  (dMDT calling requires comparing isoform usage between sample groups)
- The goal is gene-level differential expression only, not isoform switching and its
  impact on protein interactions
- The analysis needs splicing-junction-level or exon-level events rather than whole-transcript
  isoform usage differences
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: CanIsoNet
  role: Pipeline and database that calls dMDT isoform switching events and hosts per-event/per-sample
    results for 31 disease types at caniso.net
  repo: https://github.com/kahramanlab/CanIsoNet_Web
- name: STRING
  role: Protein interaction network on which isoform switching events are displayed
    and network disruptions are visualized
- name: ClinVar
  role: Annotation source used to highlight potentially pathogenic genes in the interactive
    interaction networks
- name: PFAM
  role: Provides PFAM domain IDs for interactions disrupted by isoform switching
- name: Kallisto
  role: Quantifies transcript/isoform-level abundances (TPM) for every sample prior
    to dMDT calling
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbad050
    title: 'CanIsoNet: a database to study the functional impact of isoform switching
      events in diseases'
  source_content_hash: sha256:22f746b5168bd3260618a77d0addb7cdcbd0628b8281ec3c6e73e3be829144e0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kahramanlab/CanIsoNet_Web
  tools:
  - CanIsoNet
  - ClinVar
  - Kallisto
  - PFAM
  - STRING
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# dMDT differential transcript usage analysis

## Summary

Detects differentially modified transcript isoform (dMDT) switching events between disease and control RNA-seq samples and scores their functional/pathogenic impact using an isoform-specific interaction network built on STRING, as implemented in the CanIsoNet pipeline.

## When to use

Use when you have disease and control transcriptome RNA-seq FASTQ data and want to identify isoform switching events and assess whether they disrupt protein interactions pathogenically.

## When NOT to use

- The data are not transcriptome-wide RNA-seq samples with a disease-vs-control grouping (dMDT calling requires comparing isoform usage between sample groups)
- The goal is gene-level differential expression only, not isoform switching and its impact on protein interactions
- The analysis needs splicing-junction-level or exon-level events rather than whole-transcript isoform usage differences

## Inputs

- Disease and control RNA-seq FASTQ files from GEO/ENA (e.g. from PCAWG or one of the four additional RNA-Seq datasets: OIH, Alzheimer's Disease, Parkinson's Disease, Behcet's Disease)
- Sample metadata defining disease vs control grouping
- Human reference transcriptome index for Kallisto
- CanIsoNet pipeline source code (github.com/kahramanlab/CanIsoNet_Web)

## Outputs

- Per-sample isoform expression values (Kallisto TPM quantifications)
- dMDT isoform switching statistics comparing disease vs control isoform usage
- Interaction disruption scores from the isoform-specific STRING network
- Results table conforming to the CanIsoNet database schema (gene, participating isoforms, switching statistics, impact scores, PFAM domain IDs)

## How to apply

Download disease and control FASTQ files from a public repository accession (GEO/ENA) and verify the sample metadata grouping. Quantify transcript/isoform-level abundances for every sample with Kallisto against the human reference transcriptome. Call dMDT isoform switching events by comparing isoform usage between disease and control samples using the pipeline code released with CanIsoNet (github.com/kahramanlab/CanIsoNet_Web, CC BY 4.0). Assess each event's impact on protein interactions with the isoform-specific interaction network built on STRING, highlighting ClinVar-annotated genes and reporting PFAM domain IDs for disrupted interactions. Assemble per-event and per-sample results (isoform expression values, switching statistics, interaction disruption scores) into a table matching the schema of the 31 disease types hosted at caniso.net, since the authors state that upon request new datasets can be analysed and uploaded to the database.

## Related tools

- **CanIsoNet** (Pipeline and database that calls dMDT isoform switching events and hosts per-event/per-sample results for 31 disease types at caniso.net) — https://github.com/kahramanlab/CanIsoNet_Web
- **STRING** (Protein interaction network on which isoform switching events are displayed and network disruptions are visualized)
- **ClinVar** (Annotation source used to highlight potentially pathogenic genes in the interactive interaction networks)
- **PFAM** (Provides PFAM domain IDs for interactions disrupted by isoform switching)
- **Kallisto** (Quantifies transcript/isoform-level abundances (TPM) for every sample prior to dMDT calling)

## Evaluation signals

- The output file exists and contains the required fields: gene, participating isoforms, switching statistics, and impact scores
- Results conform to the CanIsoNet database schema so the new disease can be added alongside the existing 31 disease types
- Every sample has Kallisto TPM quantifications against the human reference transcriptome
- ClinVar-annotated genes are highlighted and PFAM domain IDs are reported for disrupted interactions
- Per-sample isoform switching events include their expression values, matching the per-sample exploration feature of CanIsoNet

## Limitations

- The database currently covers 31 disease types (27 PCAWG cancer types plus OIH, Alzheimer's, Parkinson's, and Behcet's Disease); adding a new disease requires contacting the authors ('Upon request, new datasets can be analysed and uploaded to the CanIsoNet database.')
- New disease analysis depends on the researchers' request/upload path rather than self-service database insertion
- Disruption scoring depends on coverage of isoform-specific interactions in STRING and PFAM domain annotations

## Evidence

- [abstract] CanIsoNet is the first webserver that incorporates isoform expression data with STRING interaction networks and ClinVar annotations to predict the pathogenic impact of isoform switching events: "CanIsoNet is the first webserver that incorporates isoform expression data with STRING interaction networks and ClinVar annotations to predict the pathogenic impact of isoform switching events"
- [abstract] Various annotations for 11 811 isoforms and 14 357 unique isoform switching events across 31 different disease types are available.: "Various annotations for 11 811 isoforms and 14 357 unique isoform switching events across 31 different disease types are available."
- [intro] We have recently analyzed the pathogenic impact of isoform switching events in 31 disease types covering 27 different cancer types from PCAWG project: "We have recently analyzed the pathogenic impact of isoform switching events in 31 disease types covering 27 different cancer types from PCAWG project"
- [abstract] The source codes can be found under a Creative Common License at https://github.com/kahramanlab/CanIsoNet_Web.: "The source codes can be found under a Creative Common License at https://github.com/kahramanlab/CanIsoNet_Web."
- [readme] Exploring the isoform switching event in each sample with their expression values.: "4) Exploring the isoform switching event in each sample with their expression values. <br>"
