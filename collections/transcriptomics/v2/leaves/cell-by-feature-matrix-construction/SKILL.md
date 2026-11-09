---
name: cell-by-feature-matrix-construction
description: Use when you have a per-cell sparse k-mer abundance matrix (from jellyfish
  counts) and a CKG look-up table (from simhash), and need a normalized cells-by-CKGs
  feature matrix for downstream cell type classification.
when_to_use_negative:
- The input is already a gene expression or other feature matrix — this skill specifically
  transforms k-mer abundances into CKG features via the look-up table
- No CKG look-up table exists yet — the CKG generation module (informative k-mer selection
  and simhashing into fingerprints) must run first
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: pandas
  role: Load and manipulate the per-cell sparse k-mer abundance matrix and assemble
    the cells-by-CKGs feature matrix
  repo: github.com/digi2002/scSimClassify
- name: numpy
  role: Aggregate k-mer abundances into CKG sums and normalize per-cell CKG vectors
    by library size
  repo: github.com/digi2002/scSimClassify
- name: jellyfish
  role: Upstream source of per-cell k-mer counts that are aggregated into CKG features
  repo: github.com/digi2002/scSimClassify
- name: Python 3.6
  role: Runtime environment for the cell matrix generation module
  repo: github.com/digi2002/scSimClassify
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.isci.2021.102855
    title: A reference-free approach for cell type classification with scRNA-seq
  source_content_hash: sha256:a78503e2ad317b26b9b6a958237914536fde3e4773b6ca86e1b1a08fffde1168
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
  - Python 3.6
  - jellyfish
  - numpy
  - pandas
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# cell-by-feature-matrix-construction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Transform per-cell k-mer abundances into a cells-by-CKG feature matrix by aggregating k-mers via the pre-built CKG look-up table and depth-normalizing each cell, so that compressed k-mer groups (not genes) serve as cell features for classification.

## When to use

Use when you have a per-cell sparse k-mer abundance matrix (from jellyfish counts) and a CKG look-up table (from simhash), and need a normalized cells-by-CKGs feature matrix for downstream cell type classification.

## When NOT to use

- The input is already a gene expression or other feature matrix — this skill specifically transforms k-mer abundances into CKG features via the look-up table
- No CKG look-up table exists yet — the CKG generation module (informative k-mer selection and simhashing into fingerprints) must run first

## Inputs

- CKG look-up table mapping informative k-mer ids to CKG identifiers (output of the CKG generation module)
- Per-cell sparse k-mer abundance matrix generated from jellyfish k-mer counts
- Cell identifiers (e.g., from BAM or FASTA inputs processed by the k-mer matrix generation module)

## Outputs

- cell_matrix.csv: a cells-by-CKGs feature matrix with cell identifiers as rows, CKG identifiers as columns, and depth-normalized CKG abundance values in [0, 1]

## How to apply

Load the CKG look-up table produced by the CKG generation module, which maps informative k-mer ids to simhash-derived compressed k-mer group (CKG) identifiers, and load the per-cell sparse k-mer abundance matrix built with pandas and numpy from jellyfish files. Aggregate k-mer abundances into CKG expression values by summing the abundances of every k-mer assigned to each CKG according to the look-up table. Normalize each cell's CKG abundance vector by its library size (sequencing depth) so cells are comparable despite variable depth. Write the resulting feature matrix to cell_matrix.csv with cell identifiers as rows and CKG identifiers as columns. This module sits between CKG generation and the classification module in the scSimClassify pipeline.

## Related tools

- **pandas** (Load and manipulate the per-cell sparse k-mer abundance matrix and assemble the cells-by-CKGs feature matrix) — github.com/digi2002/scSimClassify
- **numpy** (Aggregate k-mer abundances into CKG sums and normalize per-cell CKG vectors by library size) — github.com/digi2002/scSimClassify
- **jellyfish** (Upstream source of per-cell k-mer counts that are aggregated into CKG features) — github.com/digi2002/scSimClassify
- **Python 3.6** (Runtime environment for the cell matrix generation module) — github.com/digi2002/scSimClassify

## Examples

```
import pandas, numpy; lut = pandas.read_csv('ckg_lookup_table.csv'); kmers = pandas.read_csv('kmer_matrix.csv', index_col=0); ckg = kmers.T.groupby(lut.set_index('kmer_id')['ckg_id']).sum().T; cell_matrix = ckg.div(ckg.sum(axis=1), axis=0); cell_matrix.to_csv('cell_matrix.csv')
```

## Evaluation signals

- cell_matrix.csv exists and can be loaded
- cell_matrix.csv contains one row per input cell and one column per CKG in the look-up table
- Normalized per-cell CKG abundance values fall within the [0, 1] range expected of depth-normalized fractions
- Each CKG column value equals the sum of the abundances of the k-mers assigned to that CKG in the look-up table (before normalization)

## Limitations

- Requires the CKG look-up table from the upstream CKG generation module; cannot run on raw k-mer counts alone
- Depth normalization assumes per-cell library size correction is sufficient for cross-cell comparability
- The README only says 'Please modify the path to use the modules' — no CLI parameters are documented

## Evidence

- [methods] Based on CKG look-up table, the features of cells is transformed to the expression of CKGs from abundance of k-mers.: "Based on CKG look-up table, the features of cells is transformed to the expression of CKGs from abundance of k-mers."
- [other] The cell matrix generation step aggregates k-mer abundances into CKG features via the pre-built CKG look-up table and normalizes to account for variable sequencing depth.: [evidence span withheld — anchor status: not_found]
- [abstract] Compressed k-mer groups (CKGs), identified by the simhash method, contain k-mers with similar abundance profiles and serve as the cells' features.: "The compressed k -mer groups (CKGs), identified by the simhash method, contain k -mers with similar abundance profiles and serve as the cells' features."
- [readme] Sparse k-mer matrix is build by several steps, such as indexing k-mer, convert k-mer sequence to k-mer id, and generate k-mer matrix from jellyfish files.: "Sparse k-mer matrix is build by several steps, such as indexing k-mer, convert k-mer sequence to k-mer id, and generate k-mer matrix from jellyfish files."
