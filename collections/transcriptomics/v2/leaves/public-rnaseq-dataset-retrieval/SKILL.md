---
name: public-rnaseq-dataset-retrieval
description: Use when a new disease/control RNA-seq dataset is available via a public
  accession (GEO/ENA) and the goal is isoform-level quantification and dMDT switching
  analysis compatible with the CanIsoNet database schema.
when_to_use_negative:
- The disease type is already one of the 31 hosted at caniso.net (27 cancer types
  from PCAWG plus OIH, Alzheimer's, Parkinson's, Behcet's) — browse the existing webserver
  results instead
- The data is not bulk RNA-seq FASTQ (e.g. proteomics, imaging, or pre-processed feature
  tables) — the pipeline requires FASTQ-level re-quantification with Kallisto
- The organism is not human — the pipeline quantifies against the human reference
  transcriptome and human-centric STRING/ClinVar/PFAM annotations
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: CanIsoNet
  role: dMDT isoform-switching calling pipeline and target database schema; code released
    under Creative Commons license for running new datasets
  repo: https://github.com/kahramanlab/CanIsoNet_Web
- name: STRING
  role: protein interaction network onto which isoform switching events are projected
    to assess functional/pathogenic impact of disrupted interactions
- name: Kallisto
  role: transcript/isoform-level TPM abundance quantification of every disease and
    control sample
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1093/bioadv/vbad050
    title: 'CanIsoNet: a database to study the functional impact of isoform switching
      events in diseases'
  source_content_hash: sha256:55c71e760ac86e2fe911ee9dce9dc170414337d1e9ae83f8cab32beca68218e1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/kahramanlab/CanIsoNet_Web
  tools:
  - CanIsoNet
  - Kallisto
  - STRING
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# public-rnaseq-dataset-retrieval

## Summary

Retrieve and prepare a new publicly available disease RNA-seq dataset (FASTQ from GEO/ENA) and quantify isoform-level abundances with Kallisto so it can be run through the CanIsoNet dMDT isoform-switching pipeline; the CanIsoNet authors explicitly sanction this for adding diseases beyond the 31 hosted at caniso.net.

## When to use

Use when a new disease/control RNA-seq dataset is available via a public accession (GEO/ENA) and the goal is isoform-level quantification and dMDT switching analysis compatible with the CanIsoNet database schema.

## When NOT to use

- The disease type is already one of the 31 hosted at caniso.net (27 cancer types from PCAWG plus OIH, Alzheimer's, Parkinson's, Behcet's) — browse the existing webserver results instead
- The data is not bulk RNA-seq FASTQ (e.g. proteomics, imaging, or pre-processed feature tables) — the pipeline requires FASTQ-level re-quantification with Kallisto
- The organism is not human — the pipeline quantifies against the human reference transcriptome and human-centric STRING/ClinVar/PFAM annotations

## Inputs

- Disease and control RNA-seq FASTQ files from a public repository accession (GEO/ENA)
- Sample metadata defining disease vs. control grouping
- Human reference transcriptome for Kallisto quantification

## Outputs

- Per-sample Kallisto transcript/isoform TPM quantification table
- dMDT isoform switching event calls (disease vs. control isoform usage comparison)
- Per-event interaction disruption assessment (STRING-based impact scores, ClinVar-annotated genes, PFAM domain IDs)
- Results table conforming to the CanIsoNet database schema (gene, participating isoforms, switching statistics, impact scores)

## How to apply

Download disease and control FASTQ files from the public repository accession and verify the sample metadata grouping before any computation. Quantify transcript/isoform abundances for every sample with Kallisto against the human reference transcriptome, producing TPM expression values. Call dMDT isoform switching events by comparing isoform usage between disease and control samples using the pipeline code released with CanIsoNet at github.com/kahramanlab/CanIsoNet_Web (Creative Commons licensed). Assess each switching event's impact on protein interactions with the isoform-specific STRING interaction network, highlighting ClinVar-annotated genes and PFAM domain IDs of disrupted interactions. Assemble per-event and per-sample results (isoform expression values, switching statistics, interaction disruption scores) into a table matching the format of the 31 disease types hosted at caniso.net; note the authors state new datasets 'can be analysed and uploaded' upon request, so contact tuelay.karakulak@usz.ch for database inclusion.

## Related tools

- **CanIsoNet** (dMDT isoform-switching calling pipeline and target database schema; code released under Creative Commons license for running new datasets) — https://github.com/kahramanlab/CanIsoNet_Web
- **STRING** (protein interaction network onto which isoform switching events are projected to assess functional/pathogenic impact of disrupted interactions)
- **Kallisto** (transcript/isoform-level TPM abundance quantification of every disease and control sample)

## Evaluation signals

- FASTQ files for every sample in the accession are retrieved and the disease/control grouping matches the sample metadata
- Kallisto TPM quantification exists for every sample (no missing abundance values)
- Each called switching event record contains the required fields: gene, participating isoforms, switching statistics, and impact scores
- Output table conforms to the CanIsoNet database schema used for the 31 disease types at caniso.net so the new disease can be added alongside them
- ClinVar-annotated genes are flagged and PFAM domain IDs are reported for disrupted interactions in the STRING network assessment

## Limitations

- Uploading new datasets to the CanIsoNet database is not automatic — 'Upon request, new datasets can be analysed and uploaded to the CanIsoNet database' (contact tuelay.karakulak@usz.ch)
- The hosted database currently covers only 31 disease types (27 PCAWG cancer types plus four RNA-Seq datasets); new diseases rely on the local pipeline run rather than the webserver
- Impact scores depend on STRING interaction coverage and ClinVar annotation availability, so events in poorly annotated genes will have weak functional evidence

## Evidence

- [other] Upon request, new datasets can be analysed and uploaded to the CanIsoNet database.: "Upon request, new datasets can be analysed and uploaded to the CanIsoNet database."
- [intro] 31 disease types covering 27 different cancer types from PCAWG project: "31 disease types covering 27 different cancer types from PCAWG project"
- [readme] CanIsoNet is freely available at [caniso.net](https://www.caniso.net/): "CanIsoNet is freely available at [caniso.net](https://www.caniso.net/)"
- [abstract] The source codes can be found under a Creative Common License at https://github.com/kahramanlab/CanIsoNet_Web.: "The source codes can be found under a Creative Common License at https://github.com/kahramanlab/CanIsoNet_Web."
- [abstract] CanIsoNet is the first webserver that incorporates isoform expression data with STRING interaction networks and ClinVar annotations: "CanIsoNet is the first webserver that incorporates isoform expression data with STRING interaction networks and ClinVar annotations"
