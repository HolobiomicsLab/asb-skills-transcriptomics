---
name: synthetic-scrnaseq-simulation-data-handling
description: Use when a simulated scRNA-seq count/expression matrix with synthetic
  cell-type labels (dozens of classes, thousands of genes) must be analyzed to measure
  how marker-panel size affects label recovery and wall-clock runtime.
when_to_use_negative:
- Cells carry continuous or manifold structure (e.g., trajectory pseudotime or spatial
  locations) rather than categorical labels — scGeneFit only supports categorical
  labeling per the paper's discussion
- Cell-type classes are expected to be nonlinearly separable in the reduced space
  — the method assumes linear separability, which the authors note may not hold in
  lower dimensions
- The input is real experimental data without a hierarchical label partition; the
  skill targets the synthetic simulation spec shipped with the scGeneFit release
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scGeneFit
  role: Solver that jointly selects the marker set via label-aware compressive classification
    on the simulated expression matrix and label vector; its fit call is timed and
    its selected gene indices extracted
  repo: github.com/solevillar/scGeneFit
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-21453-4
    title: Optimal marker gene selection for cell type discrimination in single cell
      analyses
  source_content_hash: sha256:8e335ecda350b7aec00c3ed0ca7cadee053f8cd8f7e4d7071c5ec17c3ef74e08
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - scGeneFit
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# synthetic-scrnaseq-simulation-data-handling

## Summary

Run scGeneFit marker selection on large synthetic scRNA-seq simulations (e.g., 15 functional groups, 40 classes, 1000 cells, 10,000 genes) under multivariate-normal and Poisson-linked generalized models, timing the fit and extracting the selected marker panel. This benchmarks scalability of label-aware compressive classification as the allowed marker count grows.

## When to use

Use when a simulated scRNA-seq count/expression matrix with synthetic cell-type labels (dozens of classes, thousands of genes) must be analyzed to measure how marker-panel size affects label recovery and wall-clock runtime.

## When NOT to use

- Cells carry continuous or manifold structure (e.g., trajectory pseudotime or spatial locations) rather than categorical labels — scGeneFit only supports categorical labeling per the paper's discussion
- Cell-type classes are expected to be nonlinearly separable in the reduced space — the method assumes linear separability, which the authors note may not hold in lower dimensions
- The input is real experimental data without a hierarchical label partition; the skill targets the synthetic simulation spec shipped with the scGeneFit release

## Inputs

- Simulated scRNA-seq expression matrix (cells x genes, e.g., 10,000 x 10,000) under a multivariate normal model
- Simulated expression matrix under a generalized multivariate normal model with a Poisson link
- Cell-type annotation vector with 40 classes across 15 functional groups
- Target marker set size (e.g., 50 markers)

## Outputs

- Indices of the selected marker genes (panel of exactly the target size, e.g., 50)
- Wall-clock runtime of the fit call in seconds (~15 minutes on laptop-class hardware for the 50-marker run)
- Classification accuracy as a function of the number of allowed markers

## How to apply

Load the simulated expression matrix (e.g., 10,000 cells x 10,000 genes) and its 40-label annotation vector generated per the simulation spec shipped with the scGeneFit release. Instantiate the scGeneFit solver on the matrix and label vector with the target marker set size set to the desired panel size (e.g., 50 markers), wrapping the fit call in a wall-clock timer. scGeneFit jointly optimizes cell label recovery using label-aware compressive classification rather than testing each gene separately, which is the rationale for using it instead of one-vs-all methods like COMET on high-dimensional synthetic data. After fitting, extract the selected gene indices and confirm the panel size matches the target. Repeat under both a multivariate normal model and a generalized multivariate normal model with a Poisson link, recording classification accuracy as a function of marker count.

## Related tools

- **scGeneFit** (Solver that jointly selects the marker set via label-aware compressive classification on the simulated expression matrix and label vector; its fit call is timed and its selected gene indices extracted) — github.com/solevillar/scGeneFit

## Examples

```
from scGeneFit import get_markers; markers = get_markers(X, labels, num_markers=50)  # X: 10000x10000 simulated matrix, labels: 40-class vector
```

## Evaluation signals

- The selected panel contains exactly the target number of markers (e.g., 50 for the scalability run)
- Measured wall-clock runtime is on the order of ~15 minutes on laptop-class hardware for the 50-marker, 10,000-cell, 10,000-gene run
- Classification accuracy on the 40-class simulation improves substantially and monotonically as the number of allowed markers increases, under both the multivariate normal and Poisson-link models
- Serialized JSON record contains both the runtime in seconds and the marker count (50), and the marker indices resolve to valid gene columns of the input matrix

## Limitations

- A key assumption is that cell type classes are linearly separable, which is reasonable in high dimensions but may not hold in lower dimensions
- No nonlinear dimension reduction methods are implemented; the authors plan to relax the linear separability assumption in future work
- Marker selection is limited to categorical labeling and does not support placing unlabeled cells on trajectories or in spatial assays
- Markers can be redundant when the latent dimension of cell-type expression profiles is lower than the number of cell labels

## Evidence

- [abstract] scGeneFit selects gene markers that jointly optimize cell label recovery using label-aware compressive classification methods.: "scGeneFit selects gene markers that jointly optimize cell label recovery using label-aware compressive classification methods."
- [results] Spectral toy model marker recovery: "scGeneFit selected as markers the two features representing the circle coordinates whenever the size of the target marker set was ≥2"
- [discussion] Linear separability assumption: "cell type classes are linearly separable, which is a reasonable assumption in high dimensions, but may not"
- [intro] Redundancy when latent dimension is low: "these markers will be redundant if the latent dimension of the space spanned by the gene expression profiles of each cell type is lower than the number of cell labels"
