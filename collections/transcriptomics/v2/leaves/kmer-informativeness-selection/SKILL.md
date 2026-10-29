---
name: kmer-informativeness-selection
description: 'Use when raw scRNA-seq reads have been k-merized (e.g. jellyfish) into
  a sparse k-mer matrix and you need reference-free, alignment-free cell features:
  filter uninformative k-mers, then simhash their abundance vectors into fingerprints
  to form CKGs.'
when_to_use_negative:
- Input is already a gene expression feature matrix or a reference-genome alignment
  is available and preferred — scSimClassify's CKG pipeline is designed for reference-free
  settings where the reference genome is incomplete or insufficient
- Input is raw BAM/FASTA reads rather than a k-mer matrix — the k-mer matrix generation
  module (jellyfish indexing, k-mer id conversion) must run first
- The goal is de novo transcript/sequence assembly rather than cell type classification
  features
edam_operation: http://edamontology.org/operation_3472
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: simhash
  role: Hashes each informative k-mer's abundance vector into a fixed-size locality-sensitive
    fingerprint; fingerprint collisions group k-mers with similar abundance profiles
    into CKGs
  repo: github.com/digi2002/scSimClassify
- name: pandas
  role: Loads and manipulates the sparse k-mer matrix and assembles the CKG look-up
    table
  repo: github.com/digi2002/scSimClassify
- name: numpy
  role: Numerical handling of k-mer abundance vectors prior to simhash fingerprinting
  repo: github.com/digi2002/scSimClassify
- name: jellyfish
  role: Upstream k-mer counting whose output files feed the sparse k-mer matrix that
    this skill filters
  repo: github.com/digi2002/scSimClassify
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.isci.2021.102855
    title: A reference-free approach for cell type classification with scRNA-seq
  source_content_hash: sha256:3b106818b9f8be71197ca8d76ec8a4a8de6c723908054bdc96cd0fcb2fa98669
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
  edam_operation: http://edamontology.org/operation_3472
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 18
---

# kmer-informativeness-selection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Selects informative k-mers from a sparse k-mer abundance matrix and compresses similar k-mers into compressed k-mer groups (CKGs) using the simhash locality-sensitive hashing method, producing reference-free cell features for scRNA-seq classification.

## When to use

Use when raw scRNA-seq reads have been k-merized (e.g. jellyfish) into a sparse k-mer matrix and you need reference-free, alignment-free cell features: filter uninformative k-mers, then simhash their abundance vectors into fingerprints to form CKGs.

## When NOT to use

- Input is already a gene expression feature matrix or a reference-genome alignment is available and preferred — scSimClassify's CKG pipeline is designed for reference-free settings where the reference genome is incomplete or insufficient
- Input is raw BAM/FASTA reads rather than a k-mer matrix — the k-mer matrix generation module (jellyfish indexing, k-mer id conversion) must run first
- The goal is de novo transcript/sequence assembly rather than cell type classification features

## Inputs

- Sparse k-mer matrix produced by the k-mer matrix generation module (from jellyfish k-mer counts)
- k-mer ids from indexing k-mer sequences (BAM or FASTA inputs)
- cell abundance profiles (per-k-mer abundance vectors across cells)

## Outputs

- ckg_lookup_table.csv — CKG look-up table mapping each CKG to its member k-mer ids
- Set of selected informative k-mers with simhash fingerprints
- Compressed k-mer groups (CKGs) usable as cell features for the classification module

## How to apply

Load the sparse k-mer matrix produced by the k-mer matrix generation module (built from jellyfish files via indexing k-mers, converting k-mer sequences to k-mer ids) using pandas/numpy in Python 3.6. Filter out k-mers whose abundance profiles across cells do not carry useful information, keeping only informative k-mers. Apply the simhash Python package to compress each selected k-mer's abundance vector into a fixed-size locality-sensitive fingerprint. Group k-mers whose fingerprints collide or are near-identical into compressed k-mer groups (CKGs), which by construction contain k-mers with similar abundance profiles and serve as the cells' features. Assemble a CKG look-up table mapping each CKG to its member k-mer ids and write it as ckg_lookup_table.csv for the downstream cell matrix generation module.

## Related tools

- **simhash** (Hashes each informative k-mer's abundance vector into a fixed-size locality-sensitive fingerprint; fingerprint collisions group k-mers with similar abundance profiles into CKGs) — github.com/digi2002/scSimClassify
- **pandas** (Loads and manipulates the sparse k-mer matrix and assembles the CKG look-up table) — github.com/digi2002/scSimClassify
- **numpy** (Numerical handling of k-mer abundance vectors prior to simhash fingerprinting) — github.com/digi2002/scSimClassify
- **jellyfish** (Upstream k-mer counting whose output files feed the sparse k-mer matrix that this skill filters) — github.com/digi2002/scSimClassify

## Examples

```
import pandas as pd, numpy as np; X = pd.read_csv('kmer_matrix.csv'); from simhash import Simhash; fps = [Simhash(vec).value for vec in informative_kmer_vectors(X)]
```

## Evaluation signals

- ckg_lookup_table.csv exists and is a parseable table
- Every CKG entry in the look-up table maps consistently to its member k-mer ids according to simhash fingerprint grouping
- Selected k-mers carry non-trivial abundance profiles across cells (uninformative k-mers were removed)
- CKGs contain k-mers with similar abundance profiles, so downstream cell matrix generation can transform k-mer abundances into CKG expression features
- Downstream classification with CKG features should match or beat gene-expression features, as reported in the paper's experiments

## Limitations

- CKGs are derived from raw reads without alignment to a reference genome, so individual k-mers lose sequence identity/annotation when compressed into CKGs
- The informativeness filtering step's exact criteria are not detailed in the methods text; reproducibility depends on the repository code (README says only 'modify the path to use the modules')
- Requires Python 3.6 and the specific package set (simhash, keras, sklearn, pandas, numpy, Bio); upstream k-mer counts must come from jellyfish files

## Evidence
