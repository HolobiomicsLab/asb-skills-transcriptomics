---
name: kmer-to-ckg-feature-aggregation
description: Use when you already have a per-cell k-mer abundance matrix (from jellyfish
  counts) and a CKG look-up table mapping k-mer ids to compressed k-mer groups, and
  need CKG-level cell features.
when_to_use_negative:
- Input is already a cell-level feature matrix (e.g. gene expression counts); this
  skill is only for transforming k-mer abundances into CKG features.
- No CKG look-up table exists yet — the CKG generation module (informative k-mer selection
  and simhash fingerprinting) must run first.
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: pandas
  role: Loads the k-mer abundance matrix and look-up table and assembles the cells-by-CKGs
    feature matrix
  repo: github.com/digi2002/scSimClassify
- name: numpy
  role: Performs aggregation (summing k-mer abundances per CKG) and depth normalization
    of cell vectors
  repo: github.com/digi2002/scSimClassify
- name: jellyfish
  role: Produces the per-cell k-mer count files from which the sparse k-mer abundance
    matrix is generated
  repo: github.com/digi2002/scSimClassify
- name: simhash
  role: Used upstream to hash k-mer vectors into fingerprints that define the CKG
    identifiers in the look-up table
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
  source_content_hash: sha256:f8afd95964b42988616c89bd538ba64c9a5c03dc859a30633514ee3f37e1b599
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
  - simhash
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4
n_claims: 10
---

# kmer-to-ckg-feature-aggregation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Transforms a per-cell sparse k-mer abundance matrix into a cells-by-CKGs feature matrix by summing k-mer abundances according to a simhash-derived CKG look-up table and normalizing each cell by its sequencing depth, as done in scSimClassify for reference-free cell type classification.

## When to use

Use when you already have a per-cell k-mer abundance matrix (from jellyfish counts) and a CKG look-up table mapping k-mer ids to compressed k-mer groups, and need CKG-level cell features.

## When NOT to use

- Input is already a cell-level feature matrix (e.g. gene expression counts); this skill is only for transforming k-mer abundances into CKG features.
- No CKG look-up table exists yet — the CKG generation module (informative k-mer selection and simhash fingerprinting) must run first.

## Inputs

- CKG look-up table (CSV/TSV) mapping informative k-mer ids to CKG identifiers, produced by the simhash-based CKG generation module
- Per-cell sparse k-mer abundance matrix generated from jellyfish k-mer count files (pandas/numpy)
- Cell identifiers corresponding to rows of the k-mer matrix

## Outputs

- cell_matrix.csv: normalized cells-by-CKGs feature matrix with cell identifiers as rows and CKG identifiers as columns
- Per-cell sequencing depth (library size) used for normalization

## How to apply

Load the CKG look-up table produced by the CKG generation module, which maps informative k-mer ids to simhash-derived compressed k-mer group (CKG) identifiers. Load the per-cell sparse k-mer abundance matrix (built in pandas/numpy from jellyfish k-mer counts). Aggregate k-mer abundances into CKG expression values by summing the abundances of all k-mers assigned to each CKG according to the look-up table. Normalize each cell's CKG abundance vector by its sequencing depth (library size) so cells with variable depth remain comparable. Write the resulting cells-by-CKGs feature matrix to cell_matrix.csv with cell identifiers as rows and CKG identifiers as columns; the output then feeds the scSimClassify classification module.

## Related tools

- **pandas** (Loads the k-mer abundance matrix and look-up table and assembles the cells-by-CKGs feature matrix) — github.com/digi2002/scSimClassify
- **numpy** (Performs aggregation (summing k-mer abundances per CKG) and depth normalization of cell vectors) — github.com/digi2002/scSimClassify
- **jellyfish** (Produces the per-cell k-mer count files from which the sparse k-mer abundance matrix is generated) — github.com/digi2002/scSimClassify
- **simhash** (Used upstream to hash k-mer vectors into fingerprints that define the CKG identifiers in the look-up table) — github.com/digi2002/scSimClassify
- **Python 3.6** (Runtime environment for the cell matrix generation module) — github.com/digi2002/scSimClassify

## Examples

```
python cell_matrix_generation.py --lookup ckg_lookup_table.csv --kmer_matrix kmer_matrix.csv --out cell_matrix.csv
```

## Evaluation signals

- cell_matrix.csv exists after the module runs
- cell_matrix.csv contains exactly one row per input cell and one column per CKG in the look-up table
- Normalized per-cell CKG abundance values fall within the [0, 1] range expected of depth-normalized fractions
- Each CKG column value equals the sum of the abundances of all k-mers mapped to that CKG by the look-up table, divided by the cell's library size

## Limitations

- Depends on an upstream CKG look-up table; without informative k-mer selection and simhash fingerprinting, no valid CKG mapping exists.
- Depth normalization accounts for variable sequencing depth across cells but the skill does not address other batch or quality effects.
- The repository README only says to 'modify the path to use the modules' — no CLI is documented, so integration requires editing module paths.

## Evidence
