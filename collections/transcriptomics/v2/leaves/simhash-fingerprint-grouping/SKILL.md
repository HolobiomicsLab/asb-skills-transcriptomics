---
name: simhash-fingerprint-grouping
description: Use when raw scRNA-seq reads (BAM/FASTA) have been converted to a sparse
  k-mer matrix and cell features must be compressed, reference-free, into CKG groups
  before classification.
when_to_use_negative:
- Input is already a gene expression feature table or the reference genome is complete
  and sufficient — the paper's advantage is specifically when the reference is incomplete
  or insufficient
- The k-mer matrix has not yet been generated from jellyfish files; run the k-mer
  matrix generation module first
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: simhash
  role: Hashes each selected k-mer's abundance vector into a fixed-size locality-sensitive
    fingerprint used to group k-mers into CKGs
  repo: github.com/digi2002/scSimClassify
- name: pandas
  role: Loads the sparse k-mer matrix and assembles/writes the CKG look-up table (ckg_lookup_table.csv)
  repo: github.com/digi2002/scSimClassify
- name: numpy
  role: Handles the k-mer abundance vectors passed to simhash fingerprinting
  repo: github.com/digi2002/scSimClassify
- name: jellyfish
  role: 'Upstream: provides the k-mer counting files from which the k-mer matrix (indexed
    k-mer sequences to k-mer ids) is generated'
  repo: github.com/digi2002/scSimClassify
provenance:
  source_task_ids:
  - task_002
  - task_005
  source_papers:
  - doi: 10.1016/j.isci.2021.102855
    title: A reference-free approach for cell type classification with scRNA-seq
  source_content_hash: sha256:dbb784c0095b0c0c7716afc23cf9dc187a49e86c05fa08d412064894d1f2dff1
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/digi2002/scSimClassify
  tools:
  - Python
  - jellyfish
  - numpy
  - pandas
  - simhash
  - sklearn
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# simhash-fingerprint-grouping

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generates compressed k-mer groups (CKGs) for scSimClassify by selecting informative k-mers from a sparse k-mer matrix and simhashing their abundance vectors into fixed-size locality-sensitive fingerprints, grouping k-mers with similar abundance profiles across cells into a CKG look-up table.

## When to use

Use when raw scRNA-seq reads (BAM/FASTA) have been converted to a sparse k-mer matrix and cell features must be compressed, reference-free, into CKG groups before classification.

## When NOT to use

- Input is already a gene expression feature table or the reference genome is complete and sufficient — the paper's advantage is specifically when the reference is incomplete or insufficient
- The k-mer matrix has not yet been generated from jellyfish files; run the k-mer matrix generation module first

## Inputs

- Sparse k-mer matrix from the k-mer matrix generation module (indexed k-mer sequences converted to k-mer ids from jellyfish files)
- Choice of k-mer size (e.g. k=21 or k=16)
- Fixed simhash fingerprint size

## Outputs

- ckg_lookup_table.csv — CKG look-up table mapping each CKG to its member k-mer ids
- CKG feature groups usable for cell feature transformation in the cell matrix generation module

## How to apply

Load the sparse k-mer matrix produced by the k-mer matrix generation module (built from jellyfish files with pandas/numpy). First filter the matrix to retain only informative k-mers whose abundance profiles carry useful information across cells. Then apply the simhash Python package to compress each selected k-mer's abundance vector into a fixed-size locality-sensitive fingerprint; fingerprint size is a fixed parameter that must be kept identical when comparing variants (e.g. the 21-mer vs 16-mer comparison). Group k-mers whose fingerprints collide or are near-identical into compressed k-mer groups (CKGs), since simhash guarantees that vectors with similar abundance profiles produce similar fingerprints. Finally assemble a look-up table mapping each CKG to its member k-mer ids and write ckg_lookup_table.csv, which downstream cell matrix generation consumes to transform k-mer abundance into CKG expression features.

## Related tools

- **simhash** (Hashes each selected k-mer's abundance vector into a fixed-size locality-sensitive fingerprint used to group k-mers into CKGs) — github.com/digi2002/scSimClassify
- **pandas** (Loads the sparse k-mer matrix and assembles/writes the CKG look-up table (ckg_lookup_table.csv)) — github.com/digi2002/scSimClassify
- **numpy** (Handles the k-mer abundance vectors passed to simhash fingerprinting) — github.com/digi2002/scSimClassify
- **jellyfish** (Upstream: provides the k-mer counting files from which the k-mer matrix (indexed k-mer sequences to k-mer ids) is generated) — github.com/digi2002/scSimClassify

## Evaluation signals

- ckg_lookup_table.csv exists and is a parseable table
- Every CKG entry in ckg_lookup_table.csv maps to member k-mer ids consistent with simhash fingerprint grouping
- K-mers grouped into the same CKG show similar abundance profiles across cells
- The fingerprint size parameter matches the fixed size used in any comparison runs (e.g. across k=21 and k=16 variants)
- Downstream cell matrix generation can transform k-mer abundance into CKG expression using the look-up table without missing keys

## Limitations

- CKG quality depends on prior filtering for informative k-mers; uninformative k-mers must be removed before simhashing
- Fingerprint size is fixed and must be held constant for fair comparisons between k-mer sizes; changing it alters grouping granularity
- CKGs are derived from raw reads without alignment to a reference genome, so groups are reference-free and not directly gene-annotated

## Evidence
