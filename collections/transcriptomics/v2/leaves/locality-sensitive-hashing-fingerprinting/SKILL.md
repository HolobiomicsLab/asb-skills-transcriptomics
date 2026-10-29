---
name: locality-sensitive-hashing-fingerprinting
description: Use when raw scRNA-seq reads must be transformed into reference-free
  cell features without genome alignment, and k-mers with similar abundance profiles
  need to be grouped via simhash fingerprints.
when_to_use_negative:
- Input is already a cell-level feature or gene expression table rather than a k-mer
  abundance matrix — the cell matrix generation module, not simhash, transforms k-mer
  abundance to CKG expression
- A complete, well-annotated reference genome exists and standard alignment-based
  gene expression features are sufficient, since CKG derivation is motivated by incomplete
  or insufficient reference genomes
- The k-mer matrix generation step has not been run — simhash requires the sparse
  k-mer matrix as its direct input
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: simhash
  role: Python package used to hash each selected k-mer's abundance vector into a
    fixed-size locality-sensitive fingerprint for CKG grouping
  repo: github.com/digi2002/scSimClassify
- name: pandas
  role: Loads and manipulates the sparse k-mer matrix and writes the ckg_lookup_table.csv
  repo: github.com/digi2002/scSimClassify
- name: numpy
  role: Numerical handling of k-mer abundance vectors prior to simhash fingerprinting
  repo: github.com/digi2002/scSimClassify
- name: jellyfish
  role: Upstream k-mer counting whose outputs feed the k-mer matrix used by this skill
  repo: github.com/digi2002/scSimClassify
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.isci.2021.102855
    title: A reference-free approach for cell type classification with scRNA-seq
  source_content_hash: sha256:c1689d9031af06ea2eb209a4cfc16ade03ffc27346fe7be462db5eb6598def5d
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
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 15
---

# locality-sensitive-hashing-fingerprinting

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Applies simhash locality-sensitive hashing to compress informative k-mer abundance vectors from scRNA-seq reads into fixed-size fingerprints, grouping k-mers with similar abundance profiles into compressed k-mer groups (CKGs) that serve as reference-free cell features.

## When to use

Use when raw scRNA-seq reads must be transformed into reference-free cell features without genome alignment, and k-mers with similar abundance profiles need to be grouped via simhash fingerprints.

## When NOT to use

- Input is already a cell-level feature or gene expression table rather than a k-mer abundance matrix — the cell matrix generation module, not simhash, transforms k-mer abundance to CKG expression
- A complete, well-annotated reference genome exists and standard alignment-based gene expression features are sufficient, since CKG derivation is motivated by incomplete or insufficient reference genomes
- The k-mer matrix generation step has not been run — simhash requires the sparse k-mer matrix as its direct input

## Inputs

- Sparse k-mer matrix (k-mer ids × cells) produced by the k-mer matrix generation module, loaded with pandas/numpy
- jellyfish-derived k-mer count files underlying the k-mer matrix

## Outputs

- ckg_lookup_table.csv — CKG look-up table mapping each CKG to its member k-mer ids
- CKG fingerprints grouping k-mers with similar abundance profiles

## How to apply

Load the sparse k-mer matrix produced by the k-mer matrix generation module (built from jellyfish files via pandas/numpy). Select informative k-mers, filtering out k-mers whose abundance profiles across cells carry no useful information. Apply the simhash Python package to hash each selected k-mer's abundance vector into a fixed-size locality-sensitive fingerprint. Group k-mers whose fingerprints collide or are near-identical into compressed k-mer groups (CKGs), then assemble a look-up table mapping each CKG to its member k-mer ids and write it as ckg_lookup_table.csv. The rationale is that CKGs, identified by simhash, contain k-mers with similar abundance profiles and serve as the cells' features without alignment to a reference genome.

## Related tools

- **simhash** (Python package used to hash each selected k-mer's abundance vector into a fixed-size locality-sensitive fingerprint for CKG grouping) — github.com/digi2002/scSimClassify
- **pandas** (Loads and manipulates the sparse k-mer matrix and writes the ckg_lookup_table.csv) — github.com/digi2002/scSimClassify
- **numpy** (Numerical handling of k-mer abundance vectors prior to simhash fingerprinting) — github.com/digi2002/scSimClassify
- **jellyfish** (Upstream k-mer counting whose outputs feed the k-mer matrix used by this skill) — github.com/digi2002/scSimClassify

## Examples

```
import pandas as pd; from simhash import Simhash; kmer_matrix = pd.read_csv('kmer_matrix.csv'); ckg_lookup = {kmer_id: Simhash(vec).value for kmer_id, vec in kmer_matrix.items()}
```

## Evaluation signals

- ckg_lookup_table.csv exists and is a parseable table
- Every CKG entry in the look-up table maps to its member k-mer ids consistent with the simhash fingerprint grouping
- k-mers grouped into the same CKG have similar (near-identical) simhash fingerprints
- Downstream cell matrix generation can transform every cell's k-mer abundances into CKG expression using only the look-up table

## Limitations

- CKGs are derived from raw reads without alignment to a reference genome, so their biological interpretability as genes is limited compared with aligned gene expression features
- Requires Python 3.6 and the simhash, keras, sklearn, pandas, numpy, and Bio packages as prerequisites
- README gives no explicit fingerprint size or similarity threshold; users must 'modify the path to use the modules', so parameter tuning is not documented

## Evidence
