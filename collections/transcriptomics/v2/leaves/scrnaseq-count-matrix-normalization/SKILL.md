---
name: scrnaseq-count-matrix-normalization
description: Use when you have a per-cell sparse k-mer abundance matrix (from jellyfish
  counts) and a pre-built CKG look-up table, and need reference-free CKG-level cell
  features for downstream cell type classification.
when_to_use_negative:
- The input is already a gene expression matrix or another feature table — this skill
  transforms k-mer abundances into CKG features, not gene-level features
- No CKG look-up table exists yet — the CKG generation module (informative k-mer selection
  and simhashing) must run first
- Raw reads need k-mer counting — the k-mer matrix generation module (indexing k-mers,
  converting k-mer sequence to k-mer id, generating the matrix from jellyfish files)
  must run first
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: pandas
  role: Loads and manipulates the sparse k-mer abundance matrix and writes the cells-by-CKGs
    feature table
  repo: github.com/digi2002/scSimClassify
- name: numpy
  role: Numeric aggregation of k-mer abundances into CKG sums and depth normalization
    of per-cell vectors
  repo: github.com/digi2002/scSimClassify
- name: jellyfish
  role: Upstream k-mer counter whose output files are the source of the per-cell k-mer
    abundance matrix
  repo: github.com/digi2002/scSimClassify
- name: simhash
  role: 'Indirectly used: the CKG look-up table consumed here is produced by simhashing
    k-mer vectors into fingerprints'
  repo: github.com/digi2002/scSimClassify
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.isci.2021.102855
    title: A reference-free approach for cell type classification with scRNA-seq
  source_content_hash: sha256:eb37be9983ffb6d3865480a875ed64e6b4f1bc0a961f686b28a9ea0d60c27b43
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
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# scrnaseq-count-matrix-normalization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Transforms per-cell k-mer abundance matrices into a cells-by-CKG feature matrix by aggregating k-mers into compressed k-mer groups (CKGs) via a simhash-derived look-up table and normalizing by sequencing depth, as in scSimClassify.

## When to use

Use when you have a per-cell sparse k-mer abundance matrix (from jellyfish counts) and a pre-built CKG look-up table, and need reference-free CKG-level cell features for downstream cell type classification.

## When NOT to use

- The input is already a gene expression matrix or another feature table — this skill transforms k-mer abundances into CKG features, not gene-level features
- No CKG look-up table exists yet — the CKG generation module (informative k-mer selection and simhashing) must run first
- Raw reads need k-mer counting — the k-mer matrix generation module (indexing k-mers, converting k-mer sequence to k-mer id, generating the matrix from jellyfish files) must run first

## Inputs

- CKG look-up table mapping informative k-mer ids to simhash-derived CKG identifiers (from the CKG generation module)
- Per-cell sparse k-mer abundance matrix generated from jellyfish k-mer counts (pandas/numpy dataframe)
- Cell identifiers

## Outputs

- cell_matrix.csv: a cells-by-CKGs feature matrix with cell identifiers as rows and CKG identifiers as columns, containing depth-normalized CKG abundance values

## How to apply

Load the CKG look-up table produced by the CKG generation module, which maps informative k-mer ids to simhash-derived CKG identifiers. Load the per-cell sparse k-mer abundance matrix built with pandas and numpy from jellyfish k-mer counts. Aggregate k-mer abundances into CKG expression values by summing the abundance of every k-mer assigned to each CKG according to the look-up table. Normalize each cell's CKG abundance vector by its sequencing depth (library size) so cells with variable sequencing depth are comparable. Write the resulting cells-by-CKGs matrix to cell_matrix.csv with cell identifiers as rows and CKG identifiers as columns.

## Related tools

- **pandas** (Loads and manipulates the sparse k-mer abundance matrix and writes the cells-by-CKGs feature table) — github.com/digi2002/scSimClassify
- **numpy** (Numeric aggregation of k-mer abundances into CKG sums and depth normalization of per-cell vectors) — github.com/digi2002/scSimClassify
- **jellyfish** (Upstream k-mer counter whose output files are the source of the per-cell k-mer abundance matrix) — github.com/digi2002/scSimClassify
- **simhash** (Indirectly used: the CKG look-up table consumed here is produced by simhashing k-mer vectors into fingerprints) — github.com/digi2002/scSimClassify

## Evaluation signals

- cell_matrix.csv exists and contains one row per input cell
- cell_matrix.csv contains one column per CKG in the look-up table
- Normalized per-cell CKG abundance values fall within the [0, 1] range expected of depth-normalized fractions
- Each k-mer's abundance is credited to exactly the CKG its look-up table row assigns it to (aggregation is complete and non-duplicated)
- Cells with higher sequencing depth show proportionally scaled raw CKG sums before normalization, confirming library-size correction

## Limitations

- Depends on the quality of the upstream CKG look-up table; if the simhash-based grouping of k-mers is poor, CKG features will be noisy
- Derived from raw reads without alignment to a reference genome, so features are not directly interpretable as gene expressions
- README provides no CLI and instructs users to 'modify the path to use the modules', so paths must be configured manually

## Evidence

- [other] The cell matrix generation step aggregates k-mer abundances into CKG features via the pre-built CKG look-up table and normalizes to account for variable sequencing depth across cells.: [evidence span withheld — anchor status: not_found]
