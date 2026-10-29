---
name: sparse-kmer-matrix-processing
description: Use when a sparse k-mer abundance matrix derived from scRNA-seq raw reads
  (BAM/FASTA via jellyfish) must be converted into reference-free CKG features because
  the reference genome is incomplete or insufficient.
when_to_use_negative:
- The input is already a gene expression feature table or an existing cell-by-feature
  matrix rather than a k-mer abundance matrix.
- A complete reference genome is available and standard alignment-based gene-level
  feature extraction is sufficient, since this skill targets the reference-free case.
- The CKG look-up table already exists and the task is the downstream cell matrix
  generation step (transforming k-mer abundance into CKG expression).
edam_operation: http://edamontology.org/operation_2409
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: simhash
  role: Hashes each selected k-mer's abundance vector into a fixed-size locality-sensitive
    fingerprint so k-mers with similar abundance profiles can be grouped into CKGs
  repo: github.com/digi2002/scSimClassify
- name: pandas
  role: Loads and manipulates the sparse k-mer matrix and assembles/writes the CKG
    look-up table
  repo: github.com/digi2002/scSimClassify
- name: numpy
  role: Handles the numerical k-mer abundance vectors prior to simhash fingerprinting
  repo: github.com/digi2002/scSimClassify
- name: jellyfish
  role: Upstream source of k-mer files from which the sparse k-mer matrix (the input
    to this skill) is generated
  repo: github.com/digi2002/scSimClassify
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.isci.2021.102855
    title: A reference-free approach for cell type classification with scRNA-seq
  source_content_hash: sha256:92171a6663eedd165302a565a51c7dea64c88bf76c740ca30ce4ac87586b2646
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
  edam_operation: http://edamontology.org/operation_2409
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# sparse-kmer-matrix-processing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Processes the sparse k-mer abundance matrix built from scRNA-seq raw reads (jellyfish k-mer files) into compressed k-mer groups (CKGs) by selecting informative k-mers and simhashing their abundance vectors into locality-sensitive fingerprints, without alignment to a reference genome.

## When to use

Use when a sparse k-mer abundance matrix derived from scRNA-seq raw reads (BAM/FASTA via jellyfish) must be converted into reference-free CKG features because the reference genome is incomplete or insufficient.

## When NOT to use

- The input is already a gene expression feature table or an existing cell-by-feature matrix rather than a k-mer abundance matrix.
- A complete reference genome is available and standard alignment-based gene-level feature extraction is sufficient, since this skill targets the reference-free case.
- The CKG look-up table already exists and the task is the downstream cell matrix generation step (transforming k-mer abundance into CKG expression).

## Inputs

- Sparse k-mer matrix produced by the k-mer matrix generation module (from jellyfish files)
- BAM or FASTA raw read files (upstream of the k-mer matrix)

## Outputs

- ckg_lookup_table.csv — a look-up table mapping each compressed k-mer group (CKG) to its member k-mer ids
- Fixed-size simhash fingerprints for selected k-mers

## How to apply

Load the sparse k-mer matrix produced by the k-mer matrix generation module using pandas and numpy. First filter out non-informative k-mers whose abundance profiles across cells carry no useful information. Then apply the simhash Python package to compress each retained k-mer's abundance vector into a fixed-size locality-sensitive fingerprint. Group k-mers whose fingerprints collide or are near-identical into compressed k-mer groups (CKGs) — these groups capture k-mers with similar abundance profiles across cells — and assemble a CKG look-up table mapping each CKG to its member k-mer ids, written as ckg_lookup_table.csv. The rationale is that CKGs derived from raw reads without reference-genome alignment serve as cell features downstream in the cell matrix generation and classification modules.

## Related tools

- **simhash** (Hashes each selected k-mer's abundance vector into a fixed-size locality-sensitive fingerprint so k-mers with similar abundance profiles can be grouped into CKGs) — github.com/digi2002/scSimClassify
- **pandas** (Loads and manipulates the sparse k-mer matrix and assembles/writes the CKG look-up table) — github.com/digi2002/scSimClassify
- **numpy** (Handles the numerical k-mer abundance vectors prior to simhash fingerprinting) — github.com/digi2002/scSimClassify
- **jellyfish** (Upstream source of k-mer files from which the sparse k-mer matrix (the input to this skill) is generated) — github.com/digi2002/scSimClassify

## Examples

```
import pandas as pd; from simhash import Simhash; X = pd.read_csv('kmer_matrix.csv'); fps = {i: Simhash(row).value for i, row in enumerate(X.to_numpy())}
```

## Evaluation signals

- ckg_lookup_table.csv exists, is a parseable table, and every CKG entry maps to its member k-mer ids
- The k-mer-to-CKG membership is consistent with simhash fingerprint grouping (colliding or near-identical fingerprints share a CKG)
- All output fingerprints have the same fixed size, as expected from the simhash package
- Non-informative k-mers (flat/noisy abundance profiles across cells) are excluded from the look-up table
- Downstream cell matrix generation can consume the look-up table to transform k-mer abundance into CKG expression

## Limitations

- Requires Python 3.6 and the simhash, keras, sklearn, pandas, numpy, and Bio packages; README notes 'Please modify the path to use the modules', so hard-coded paths must be adjusted.
- CKG features lend themselves to better classification accuracy than gene expression features only 'in the majority of experimental cases', so gains are not guaranteed for every dataset.
- Only k-mers with similar abundance profiles are compressed together; k-mers without useful abundance-profile information are discarded by the informative-k-mer filter.

## Evidence
