---
name: k-mer-compressed-group-feature-generation
description: Use when scRNA-seq data must be classified without aligning reads to
  a reference genome, or when the reference genome is incomplete or insufficient to
  represent subject genomes — convert k-mer abundance profiles into CKG features.
when_to_use_negative:
- A complete, high-quality reference genome is available and gene-expression feature
  matrices are already suitable for the classifier
- The input is already a cell feature table (e.g., precomputed CKG or gene-expression
  matrix) — skip directly to the classification module
edam_operation: http://edamontology.org/operation_3472
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: simhash
  role: hashes informative k-mer abundance vectors into fingerprints that define compressed
    k-mer groups (CKGs)
  repo: github.com/digi2002/scSimClassify
- name: jellyfish
  role: produces the k-mer files from which the sparse k-mer matrix is generated
- name: numpy
  role: numeric handling of k-mer and CKG feature matrices
- name: pandas
  role: loading and writing feature matrices and cell labels
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.isci.2021.102855
    title: A reference-free approach for cell type classification with scRNA-seq
  source_content_hash: sha256:d5cc00ecfddeef4df6956c74c9cca23eaac928090bac21fdf3f6084bee1f2714
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/digi2002/scSimClassify
  tools:
  - jellyfish
  - numpy
  - pandas
  - simhash
  - sklearn
  edam_operation: http://edamontology.org/operation_3472
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# k-mer-compressed-group-feature-generation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Generates compressed k-mer group (CKG) features for scRNA-seq cells by selecting informative k-mers and simhashing k-mer vectors into fingerprints, producing a CKG look-up table that transforms raw-read k-mer abundances into reference-free cell features.

## When to use

Use when scRNA-seq data must be classified without aligning reads to a reference genome, or when the reference genome is incomplete or insufficient to represent subject genomes — convert k-mer abundance profiles into CKG features.

## When NOT to use

- A complete, high-quality reference genome is available and gene-expression feature matrices are already suitable for the classifier
- The input is already a cell feature table (e.g., precomputed CKG or gene-expression matrix) — skip directly to the classification module

## Inputs

- jellyfish k-mer files derived from scRNA-seq BAM or FASTA reads
- sparse k-mer matrix (k-mer IDs x cells abundance matrix)

## Outputs

- CKG look-up table mapping informative k-mers to simhash fingerprints
- cell-by-CKG feature matrix (transformed expression of CKGs per cell)

## How to apply

Start from sparse k-mer matrices produced from jellyfish output (BAM or FASTA inputs; steps include indexing k-mers, converting k-mer sequences to k-mer IDs, and generating the k-mer matrix). In the CKG generation module, select informative k-mers and apply simhash to hash k-mer abundance vectors into fingerprints, grouping k-mers with similar abundance profiles into CKGs and producing a CKG look-up table. Then, in the cell matrix generation module, use the look-up table to transform each cell's k-mer abundances into CKG expression values, yielding the feature matrix fed to a Keras neural-network classifier. The rationale is that CKGs are derived from raw reads without reference alignment, so they remain valid when reference genomes are incomplete; the resulting features should outperform raw gene-expression features in classification accuracy in the majority of cases.

## Related tools

- **simhash** (hashes informative k-mer abundance vectors into fingerprints that define compressed k-mer groups (CKGs)) — github.com/digi2002/scSimClassify
- **jellyfish** (produces the k-mer files from which the sparse k-mer matrix is generated)
- **numpy** (numeric handling of k-mer and CKG feature matrices)
- **pandas** (loading and writing feature matrices and cell labels)

## Examples

```
python generate_ckg.py --kmer_matrix jellyfish_kmer_matrix.tsv --output ckg_lookup_table.csv  # then transform cells: python generate_cell_matrix.py --lookup ckg_lookup_table.csv --kmer_matrix jellyfish_kmer_matrix.tsv --out cell_ckg_matrix.csv
```

## Evaluation signals

- CKG look-up table groups only k-mers with similar abundance profiles into the same compressed k-mer group (same simhash fingerprint)
- The resulting cell matrix has one row per cell and one column per CKG, with values transformed from k-mer abundances via the look-up table
- Downstream classification accuracy and weighted F1 on CKG features meet or exceed the gene-expression-feature baseline in the majority of experimental cases
- Features are generated without aligning reads to a reference genome (reference-free invariant)

## Limitations

- CKG quality depends on selecting informative k-mers before simhashing; uninformative k-mers degrade the fingerprints
- Pipeline requires python 3.6 with simhash, keras, sklearn, pandas, numpy, and Bio packages, and the README only says 'modify the path to use the modules' — no turnkey CLI
- Benefit over gene-expression features holds in 'the majority of experimental cases', not universally, so a baseline comparison is needed

## Evidence
