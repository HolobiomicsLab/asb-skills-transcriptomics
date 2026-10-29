---
name: single-cell-immune-repertoire-object-manipulation
description: Use when single-cell VDJ-seq data (SingleCellExperiment with TCR information)
  must be pseudobulked into a VDJ feature space of V/D/J gene usage counts per miloR
  neighbourhood for downstream trajectory analysis.
when_to_use_negative:
- Input is already a normalized feature table or pseudobulk matrix rather than a cell-level
  SingleCellExperiment/milo object with VDJ annotations
- The dataset lacks paired single-cell VDJ sequencing (e.g. Baccin et al./Bandyopadhyay
  et al. lack BCR sequencing) or has too few cells for miloR neighbourhood construction
  (e.g. only 972 healthy B cells)
- The target trajectory is cyclic (e.g. B cell receptor editing), which cannot be
  represented by absorbing Markov chains downstream of the feature space
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3168
- http://edamontology.org/topic_2229
tools:
- name: dandelionR
  role: Provides vdjPseudobulk and .normalizeFeatureSpace to tabulate V/D/J gene usage
    per pseudobulk and normalise the feature space
  repo: github.com/tuonglab/dandelionR
- name: miloR
  role: Builds the neighbourhood/pseudobulk structure on which VDJ gene usage is tabulated
    (buildGraph, makeNhoods)
- name: scRepertoire
  role: Interfaces immune receptor contigs into the SingleCellExperiment object (combineTCR,
    combineExpression) upstream of pseudobulking
  repo: github.com/ncborcherding/scRepertoire
- name: Dandelion
  role: Original Python implementation of the VDJ-feature space method that dandelionR
    replicates
  repo: github.com/zktuong/dandelion
- name: SingleCellExperiment
  role: Object container carrying the retained TCR-complete cells and the Feature_space
    assay
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2025.06.047
    title: '<i>dandelionR</i> : Single-cell immune repertoire trajectory analysis
      in R'
  source_content_hash: sha256:d186807db29707e91d5a167a1c0cf8de6ed41976a2b6bfd962b89463feb6096f
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Dandelion
  - R
  - SingleCellExperiment
  - dandelionR
  - miloR
  - scRepertoire
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3168
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# single-cell-immune-repertoire-object-manipulation

## Summary

Constructs and manipulates pseudobulked VDJ feature spaces in R by combining dandelionR, miloR, and SingleCellExperiment objects, converting single-cell immune receptor data into a continuous V/D/J gene-usage matrix suitable for trajectory analysis.

## When to use

Use when single-cell VDJ-seq data (SingleCellExperiment with TCR information) must be pseudobulked into a VDJ feature space of V/D/J gene usage counts per miloR neighbourhood for downstream trajectory analysis.

## When NOT to use

- Input is already a normalized feature table or pseudobulk matrix rather than a cell-level SingleCellExperiment/milo object with VDJ annotations
- The dataset lacks paired single-cell VDJ sequencing (e.g. Baccin et al./Bandyopadhyay et al. lack BCR sequencing) or has too few cells for miloR neighbourhood construction (e.g. only 972 healthy B cells)
- The target trajectory is cyclic (e.g. B cell receptor editing), which cannot be represented by absorbing Markov chains downstream of the feature space

## Inputs

- Preprocessed SingleCellExperiment/milo object with complete TCR information (from setupVdjPseudobulk, Suo et al. single-cell VDJ data)
- miloR neighbourhood structure built on X_scvi reduced dimensions (k = 30, d = 20, prop = 0.3)

## Outputs

- milo object containing a Feature_space assay of V/D/J gene usage counts per pseudobulk (160 V/D/J genes x 1516 pseudobulks)
- PCA embedding of the feature space (runPCA, ncomponents = 20)
- RDS file of the pb.milo object

## How to apply

Start from a preprocessed SingleCellExperiment/milo object produced by setupVdjPseudobulk (e.g. 17308 Suo et al. cells with complete TCR information retained from 65102). Call dandelionR::vdjPseudobulk on the milo object with mode_option = "abT" so VDJ gene columns carrying the abT prefix are tabulated as usage counts per pseudobulk neighbourhood. Apply the package's feature-space normalisation (.normalizeFeatureSpace with default min_count and renormalize settings) to the tabulated counts. Store the result as a milo object with a Feature_space assay and run runPCA with ncomponents = 20 for downstream diffusion-map trajectory use; validate that the feature space is 160 V/D/J genes x 1516 pseudobulks for the Suo et al. abT data.

## Related tools

- **dandelionR** (Provides vdjPseudobulk and .normalizeFeatureSpace to tabulate V/D/J gene usage per pseudobulk and normalise the feature space) — github.com/tuonglab/dandelionR
- **miloR** (Builds the neighbourhood/pseudobulk structure on which VDJ gene usage is tabulated (buildGraph, makeNhoods))
- **scRepertoire** (Interfaces immune receptor contigs into the SingleCellExperiment object (combineTCR, combineExpression) upstream of pseudobulking) — github.com/ncborcherding/scRepertoire
- **Dandelion** (Original Python implementation of the VDJ-feature space method that dandelionR replicates) — github.com/zktuong/dandelion
- **SingleCellExperiment** (Object container carrying the retained TCR-complete cells and the Feature_space assay)

## Examples

```
pb.milo <- vdjPseudobulk(milo_object, mode_option = "abT"); pb.milo <- runPCA(pb.milo, ncomponents = 20)
```

## Evaluation signals

- Feature-space dimensionality matches 160 V/D/J genes x 1516 pseudobulks for the Suo et al. abT data
- Cell counts agree with the reference: dandelionR output 17,281 cells vs original Dandelion 17,234, with 17,208 shared
- Feature_space assay exists in the pb.milo object and PCA is computed with ncomponents = 20
- All VDJ gene columns carry the abT prefix consistent with mode_option = "abT"
- Normalised counts respect the default min_count filtering and renormalization of .normalizeFeatureSpace

## Limitations

- Filtering of non-productive receptors and ambiguous VDJ chain status discards cells: 65102 Suo et al. cells reduced to 17308 with complete TCR information
- Numerical discrepancies with the Python Dandelion (cell counts differ by tens of cells) partly due to fundamental discrepancies in diffusion map and KNN graph construction between R and Python packages
- Datasets with insufficient cells (e.g. only 972 healthy B cells) may fail miloR neighbourhood construction

## Evidence

- [results] vdjPseudobulk tabulates VDJ gene usage per pseudobulk: [evidence span withheld — anchor status: not_found]
- [results] Feature space dimensionality: "With 160 V/D/J genes and 1516 pseudobulks, the VDJ feature space captured features"
- [results] Filtering rationale: "Out of 65102 cells in the data, 17308 cells were retained after filtering due to having complete TCR information"
- [methods] Pseudobulking tool: "Pseudobulking can be achieved through the miloR (v2.0.0) package [7]."
- [readme] Interface with scRepertoire: "It provides all the necessary tools to interface with [`scRepertoire`](https://github.com/ncborcherding/scRepertoire) and a custom implementation of absorbing markov chain for pseudotime inference"
