---
name: accessible-chromatin-sequence-extraction
description: Use when scATAC-seq peak data must be converted into gene-associated
  accessible promoter/enhancer sequences as the substrate for TF motif scanning in
  CellOracle base-GRN construction.
when_to_use_negative:
- Input is scRNA-seq-only data with no chromatin accessibility information — CellOracle's
  base-GRN construction requires scATAC-seq peaks
- A pre-built base GRN is acceptable — the paper itself uses a Base-GRN generated
  from the Mouse sci-ATAC-seq Atlas rather than rebuilding one
- The goal is validating TF-target interactions rather than enumerating potential
  regulatory edges from motif hits
edam_operation: http://edamontology.org/operation_0239
edam_topics:
- http://edamontology.org/topic_0749
- http://edamontology.org/topic_0602
tools:
- name: CellOracle
  role: 'Hosts the base-GRN construction pipeline: TSS annotation, accessible sequence
    extraction, and gimmemotifs-based motif scanning that emits the TF-target edge
    list'
  repo: https://github.com/morris-lab/CellOracle
- name: Cicero
  role: R package used on the scATAC-seq atlas to compute peak co-accessibility, identifying
    distal cis-regulatory elements linked to promoters before sequence extraction
- name: gimmemotifs
  role: Provides the motif scanning engine and default CisBP motif position frequency
    matrices used by CellOracle's motif scan module
  repo: https://github.com/vanheeringen-lab/gimmemotifs
- name: Python
  role: Implementation language for the CellOracle base-GRN pipeline (developed with
    python 3.8, tested with python 3.10)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.stemcr.2022.11.010
    title: Gene regulatory network reconfiguration in direct lineage reprogramming
  source_content_hash: sha256:8b23536727b5baae0e1b158389f48813449480eec82dcb2ea4333346772684d7
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/morris-lab/CellOracle
  repo_url: https://github.com/morris-lab/CellOracle
  tools:
  - CellOracle
  - Cicero
  - Python
  - gimmemotifs
  edam_operation: http://edamontology.org/operation_0239
  edam_topics:
  - http://edamontology.org/topic_0749
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# Accessible chromatin sequence extraction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Extracts the DNA sequences of accessible promoters and Cicero-linked distal enhancers from scATAC-seq peaks so they can be scanned for TF binding motifs during CellOracle base-GRN construction.

## When to use

Use when scATAC-seq peak data must be converted into gene-associated accessible promoter/enhancer sequences as the substrate for TF motif scanning in CellOracle base-GRN construction.

## When NOT to use

- Input is scRNA-seq-only data with no chromatin accessibility information — CellOracle's base-GRN construction requires scATAC-seq peaks
- A pre-built base GRN is acceptable — the paper itself uses a Base-GRN generated from the Mouse sci-ATAC-seq Atlas rather than rebuilding one
- The goal is validating TF-target interactions rather than enumerating potential regulatory edges from motif hits

## Inputs

- scATAC-seq peak dataset (Mouse sci-ATAC-seq Atlas peak table)
- Peak-to-TSS annotations (accessible proximal promoter/enhancer regions around TSS)
- Cicero co-accessibility scores linking distal peaks to promoters
- TF binding motif position frequency matrices (Cis-BP defaults shipped with gimmemotifs/CellOracle)

## Outputs

- Accessible promoter and enhancer DNA sequences per gene
- TF-to-gene motif hit assignments
- base_grn_edge_list.csv: TF-target edge list defining the CellOracle base GRN

## How to apply

Start from the scATAC-seq peak dataset (in this study, the Mouse sci-ATAC-seq Atlas) and annotate peaks against transcription start sites to retain accessible proximal promoter and enhancer sequence around each TSS. Run Cicero on the atlas data to compute co-accessibility scores between peaks, linking distal cis-regulatory elements to promoters. Preprocess the annotated peak set to extract the accessible DNA sequences of promoters and their Cicero-linked enhancer elements per gene. Scan these accessible regulatory sequences for TF binding motifs using the gimmemotifs-based CellOracle motif scan module with the default CisBP motif position frequency matrices, then connect each TF to genes whose accessible regulatory elements contain hits for that TF's motif. The resulting edge list is emitted as potential regulatory connections, not validated interactions.

## Related tools

- **CellOracle** (Hosts the base-GRN construction pipeline: TSS annotation, accessible sequence extraction, and gimmemotifs-based motif scanning that emits the TF-target edge list) — https://github.com/morris-lab/CellOracle
- **Cicero** (R package used on the scATAC-seq atlas to compute peak co-accessibility, identifying distal cis-regulatory elements linked to promoters before sequence extraction)
- **gimmemotifs** (Provides the motif scanning engine and default CisBP motif position frequency matrices used by CellOracle's motif scan module) — https://github.com/vanheeringen-lab/gimmemotifs
- **Python** (Implementation language for the CellOracle base-GRN pipeline (developed with python 3.8, tested with python 3.10))

## Examples

```
import pandas as pd; base_grn = pd.read_csv("base_grn_edge_list.csv"); assert {"TF", "target"}.issubset(base_grn.columns)
```

## Evaluation signals

- base_grn_edge_list.csv exists, is a parseable CSV, and contains TF and target gene columns with non-empty rows
- Every emitted TF-target edge corresponds to a motif hit for that TF within the accessible promoter or Cicero-linked enhancer sequence of the target gene
- All peaks feeding sequence extraction were TSS-annotated, and distal enhancers are justified by Cicero co-accessibility links
- Motif scanning used the default CisBP position frequency matrices (no undocumented custom motif set)
- Edges are labeled as potential regulatory connections, not validated interactions

## Limitations

- The base GRN consists of potential TF-target edges inferred from motif hits, not experimentally validated regulatory interactions
- Edge quality depends on TSS annotation accuracy and Cicero co-accessibility linking of distal peaks to promoters
- Results are tied to the default CisBP motif matrices and to the Mouse sci-ATAC-seq Atlas input used in this study

## Evidence

- [methods] Base GRN from scATAC-seq atlas: "Base-GRN: Base-GRN generated from `the Mouse sci-ATAC-seq Atlas"
- [abstract] CellOracle infers GRNs from transcriptome and epigenome data: "CellOracle, a computational method to infer GRNs from single-cell transcriptome and epigenome data"
- [intro] CellOracle integrates single-cell data modalities for GRN inference: "We previously described CellOracle, a computational pipeline for GRN inference via integrating different single-cell data modalities (Kamimoto et al., 2020)."
- [methods] CellOracle is open source on GitHub: "CellOracle code is open source and available on GitHub: (https://github.com/morris-lab/CellOracle)"
- [methods] Python version requirements for CellOracle: "CellOracle was developed using python 3.8. It is also tested with python 3.10."
