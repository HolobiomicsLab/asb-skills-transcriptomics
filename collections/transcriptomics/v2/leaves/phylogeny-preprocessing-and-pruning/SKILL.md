---
name: phylogeny-preprocessing-and-pruning
description: Use when a single-cell phylogeny and an expression matrix must be reconciled
  before lineage-aware analysis — e.g., cells present in one modality but not the
  other, unifurcations in the tree, or raw UMI counts needing normalization.
when_to_use_negative:
- The input is already a normalized, filtered expression matrix paired with a binary
  tree free of unifurcations whose leaves exactly match the cell barcodes
- The analysis goal is pure tree visualization without joining expression data (iTOL/CeLaVi-style
  overlays suffice)
- The tree leaves do not correspond to individual cells (e.g., a species-level phylogeny)
  — the barcode-matching preprocessing is meaningless
edam_operation: http://edamontology.org/operation_0326
edam_topics:
- http://edamontology.org/topic_0194
- http://edamontology.org/topic_3170
tools:
- name: ape
  role: R package whose collapse.singles function collapses unifurcations in the reconstructed
    tumor phylogeny before analysis
- name: VISION
  role: Provides filterGenesFano (default parameters) to select informative projection
    genes and the front-end used by PhyloVision; PhyloVision itself is distributed
    via the VISION repository
  repo: https://github.com/YosefLab/VISION
- name: Cassiopeia
  role: Package that originally inferred the single-cell phylogenies being preprocessed
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1016/j.crmeth.2022.100200
    title: Interactive, integrated analysis of single-cell transcriptomic and phylogenetic
      data with PhyloVision
  source_content_hash: sha256:d5a3a33464c6841bbeb49804f2d120947c621ff7898e81eb5a7c5ba309c808b8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/YosefLab/VISION
  tools:
  - Cassiopeia
  - PhyloVision
  - R
  - VISION
  - ape
  edam_operation: http://edamontology.org/operation_0326
  edam_topics:
  - http://edamontology.org/topic_0194
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# phylogeny-preprocessing-and-pruning

## Summary

Prepare a single-cell phylogeny (e.g., a Cassiopeia-reconstructed tumor tree from Quinn et al. 2021 clone CP007) and its matched scRNA-seq data for downstream PhyloVision/Hotspot analysis by pruning to shared cells, collapsing unifurcations, library-size normalizing counts, and selecting informative genes.

## When to use

Use when a single-cell phylogeny and an expression matrix must be reconciled before lineage-aware analysis — e.g., cells present in one modality but not the other, unifurcations in the tree, or raw UMI counts needing normalization.

## When NOT to use

- The input is already a normalized, filtered expression matrix paired with a binary tree free of unifurcations whose leaves exactly match the cell barcodes
- The analysis goal is pure tree visualization without joining expression data (iTOL/CeLaVi-style overlays suffice)
- The tree leaves do not correspond to individual cells (e.g., a species-level phylogeny) — the barcode-matching preprocessing is meaningless

## Inputs

- Raw UMI count matrix (cells x genes) from NCBI GEO (series GSE161363, dataset GSM4905334)
- Cassiopeia-reconstructed single-cell phylogeny (Newick/tree object) from Quinn et al. 2021
- Per-cell meta-data table keyed by cell barcode

## Outputs

- Pruned, unifurcation-collapsed phylogeny containing only cells shared with the expression matrix
- Library-size-normalized expression matrix (scaled to the median UMI count of the clone)
- Set of informative genes (filterGenesFano-selected) suitable for projections and Hotspot autocorrelation

## How to apply

Restrict both modalities to the intersection of cells present in the expression matrix and the lineage, ensuring tree leaf names match cell barcodes in the counts matrix. Collapse unifurcations in the tree using the collapse.singles function in the R package ape so internal nodes represent real branching events. Normalize raw UMI counts to the median number of UMIs in the clone so that downstream signature Z-scores and Hotspot autocorrelations are depth-comparable. Select informative genes for projection using VISION's filterGenesFano with default parameters. Finally, supply the pruned tree as the latent space (or tree input) to PhyloVision or Hotspot, building a KNN graph from cophenetic distances with n_neighbors = 30.

## Related tools

- **ape** (R package whose collapse.singles function collapses unifurcations in the reconstructed tumor phylogeny before analysis)
- **VISION** (Provides filterGenesFano (default parameters) to select informative projection genes and the front-end used by PhyloVision; PhyloVision itself is distributed via the VISION repository) — https://github.com/YosefLab/VISION
- **Cassiopeia** (Package that originally inferred the single-cell phylogenies being preprocessed)

## Examples

```
library(ape); tree <- collapse.singles(tree)
```

## Evaluation signals

- Every tree leaf name corresponds to a cell barcode column in the (normalized) counts matrix, and no cell in the expression matrix is missing from the tree (or vice versa)
- The collapsed tree contains no unifurcations (no internal nodes with exactly one child) after collapse.singles
- Per-cell UMI totals in the normalized matrix are all equal to the median UMI count of the clone (e.g., 1,127-cell CP007 clone)
- The filtered gene set is non-empty and restricted to genes passing filterGenesFano's default informativeness criterion, suitable as projection_genes
- Downstream KNN graph built from cophenetic distances on the pruned tree has n_neighbors = 30 neighbors per cell without errors from dangling leaves

## Limitations

- PhyloVision is implemented in R, which may pose challenges for interoperating with Python-only software such as Cassiopeia
- Leaf nodes in the tree must have names matching the cell barcodes in the counts matrix; mismatches silently break the cell-metric linkage
- Preprocessing steps (pruning, collapsing, normalization choices) are dataset-specific — e.g., normalization to the median UMI count of a clone — and may not transfer directly to other datasets

## Evidence

- [methods] collapsed using the collapse.singles function in the R package ape: "collapsed using the collapse.singles function in the R package ape"
- [methods] filterGenesFano informative gene selection: "Informative genes were found using the filterGenesFano function in VISION with default parameters"
- [methods] GEO source of expression and meta-data: "Meta data corresponding to each cell was downloaded from NCBI GEO, series GSE161363 and dataset GSM4905334"
- [readme] tree leaves must match cell barcode labels: [evidence span withheld — anchor status: external_doc]
- [results] Cassiopeia-inferred phylogenies as input: "reconstructed single-cell phylogenies from the original study, which were inferred with the Cassiopeia package"
