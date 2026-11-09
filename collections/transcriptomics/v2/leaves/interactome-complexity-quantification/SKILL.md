---
name: interactome-complexity-quantification
description: Use when a gene cluster table of significantly changed genes with LFC
  values needs interactome-level assessment of key interactors, e.g. to characterize
  cellular perturbation events or prioritize network targets.
when_to_use_negative:
- Input is not significantly changed genes with LFC values — the score is defined
  on LFC-based significance, so unfiltered or raw expression data is out of scope
- The goal is de novo prediction of physical interactions rather than assessment of
  known interactors from STRING/UniProt resources
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0128
- http://edamontology.org/topic_0085
- http://edamontology.org/topic_3577
tools:
- name: OmicInt
  role: R package providing the network-assessment functions, including protein type
    and cellular location classification of key interactors; used at version v1.1.7
  repo: https://github.com/AusteKan/OmicInt
- name: STRINGdb
  role: R interface to STRING v11 used to retrieve known protein-protein interactors
    at the proteome level for each gene cluster
- name: RCurl
  role: HTTP querying of UniProt for protein type and cellular location annotations
    of cluster genes and their interactors
- name: UniProt
  role: Resource supplying functional annotations (protein type, cellular location)
    used to classify the key interactors
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.ailsci.2021.100025
    title: 'OmicInt package: Exploring omics data and regulatory networks using integrative
      analyses and machine learning'
  source_content_hash: sha256:5e24cb228b9a7bd3c4a8cd05aa0c0f2e7fb674d337dbef7d71db0e8aa90ca1a3
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/AusteKan/OmicInt
  repo_url: https://github.com/AusteKan/OmicInt
  tools:
  - OmicInt
  - R
  - RCurl
  - STRING v11
  - STRINGdb
  - UniProt
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0128
  - http://edamontology.org/topic_0085
  - http://edamontology.org/topic_3577
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4375
n_claims: 16
---

# interactome-complexity-quantification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies predicted interactome complexity for significantly changed genes by combining known protein-protein interactors (STRING v11 via STRINGdb, UniProt) with log-fold-change (LFC) values, as implemented in the OmicInt R package for exploring gene networks.

## When to use

Use when a gene cluster table of significantly changed genes with LFC values needs interactome-level assessment of key interactors, e.g. to characterize cellular perturbation events or prioritize network targets.

## When NOT to use

- Input is not significantly changed genes with LFC values — the score is defined on LFC-based significance, so unfiltered or raw expression data is out of scope
- The goal is de novo prediction of physical interactions rather than assessment of known interactors from STRING/UniProt resources

## Inputs

- CSV data frame of significantly changed genes with cluster assignments and log-fold-change (LFC) values
- STRING v11 protein-protein interaction records retrieved via STRINGdb
- UniProt functional annotations (protein type, cellular location) fetched via RCurl/HTTP

## Outputs

- Per-gene interactome network sizes (predicted interactome complexity scores)
- Combined table of interactome sizes and UniProt annotations written to interactome_annotations.csv

## How to apply

Load a CSV data frame containing significantly changed genes, their cluster assignments, and LFC values. For each gene cluster, retrieve known protein-protein interactors at the proteome level from STRING v11 using the STRINGdb R package, and query UniProt (via RCurl/HTTP) for functional annotations such as protein type and cellular location. Quantify predicted interactome complexity per gene from retrieved interaction counts together with its LFC value, following the OmicInt network-assessment approach. Assemble per-gene interactome network sizes and UniProt annotations into a single table using OmicInt v1.1.7 data-handling dependencies (dplyr, tidyr, stringr) and export to CSV. Protein type and cellular location classification provide the useful assessments of the key interactors; correctness is judged by consistent join of interaction counts, LFC, and annotations per gene.

## Related tools

- **OmicInt** (R package providing the network-assessment functions, including protein type and cellular location classification of key interactors; used at version v1.1.7) — https://github.com/AusteKan/OmicInt
- **STRINGdb** (R interface to STRING v11 used to retrieve known protein-protein interactors at the proteome level for each gene cluster)
- **RCurl** (HTTP querying of UniProt for protein type and cellular location annotations of cluster genes and their interactors)
- **UniProt** (Resource supplying functional annotations (protein type, cellular location) used to classify the key interactors)

## Examples

```
Rscript -e 'library(STRINGdb); clusters <- read.csv("gene_clusters.csv"); ...; write.csv(interactome_annotations, "interactome_annotations.csv")'
```

## Evaluation signals

- Output CSV (interactome_annotations.csv) contains one row per gene with both interactome network size and UniProt annotation columns populated
- Interaction counts per gene are consistent with STRING v11 proteome-level interactor retrieval for the queried cluster
- Protein type and cellular location fields are filled for genes and their interactors from UniProt, matching OmicInt's classification utilities
- Data assembly runs under OmicInt v1.1.7 dependency set including dplyr, tidyr, stringr, STRINGdb, and RCurl
- LFC values in the output match the input gene cluster table, confirming no loss of the score parameter

## Limitations

- Assessment relies on known interactors at the proteome level from STRING v11 and UniProt; interactions not present in these resources will not be captured
- Users must ensure the supplied data is in the right format — a CSV data frame containing all relevant experimental information — before analysis
- Scored via a previously developed method combining LFC and predicted interactome complexity; it does not directly predict novel physical interactions

## Evidence
