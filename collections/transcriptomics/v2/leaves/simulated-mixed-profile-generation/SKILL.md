---
name: simulated-mixed-profile-generation
description: Use when a cell-type deconvolution model needs supervised training data
  but no ground-truth spatial compositions exist, and a single-cell RNA-seq reference
  with cell type annotations is available.
when_to_use_negative:
- The input is already a mixed-profile matrix with known compositions — simulation
  is redundant
- The scRNA-seq reference lacks reliable cell type annotations needed to define simulation
  proportions
- The goal is direct deconvolution of real spatial spots rather than generating training
  data
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: SpatialDDLS
  role: Provides createSpatialDDLSobject, genMixedCellProp, and simMixedProfiles to
    simulate mixed transcriptional profiles with known cell composition
  repo: https://CRAN.R-project.org/package=SpatialDDLS
- name: SpatialDDLSdata
  role: Supplies the MouseDLN.SCE scRNA-seq and MouseDLN.ST spatial example datasets
    used for simulation
  repo: github.com/diegommcc/SpatialDDLSdata
- name: SingleCellExperiment
  role: Container format for the scRNA-seq reference loaded into the SpatialDDLS object
- name: SpatialExperiment
  role: Container format for the spatial transcriptomics data used to select shared
    genes
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btae072
    title: 'SpatialDDLS: an R package to deconvolute spatial transcriptomics data
      using neural networks'
  source_content_hash: sha256:283e6c21d4b7cb265f98c48e610ecd3b536197d9eccfc84d6a0cacc14339913b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - SingleCellExperiment
  - SpatialDDLS
  - SpatialDDLSdata
  - SpatialExperiment
  - ggplot2
  - ggpubr
  - keras
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5625
n_claims: 16
---

# simulated-mixed-profile-generation

## Summary

Generate simulated mixed transcriptional profiles with known cell type composition from scRNA-seq reference data using the SpatialDDLS R package, to serve as labeled training and test data for a deconvolution neural network.

## When to use

Use when a cell-type deconvolution model needs supervised training data but no ground-truth spatial compositions exist, and a single-cell RNA-seq reference with cell type annotations is available.

## When NOT to use

- The input is already a mixed-profile matrix with known compositions — simulation is redundant
- The scRNA-seq reference lacks reliable cell type annotations needed to define simulation proportions
- The goal is direct deconvolution of real spatial spots rather than generating training data

## Inputs

- SingleCellExperiment object with annotated scRNA-seq reference (e.g. MouseDLN.SCE)
- SpatialExperiment object with spatial transcriptomics data (e.g. MouseDLN.ST)
- Cell type annotations per reference cell (12 cell types in the mouse lymph node dataset)

## Outputs

- Cell composition matrices (7500 train / 2500 test spots) with Dirichlet-sampled cell type proportions
- Simulated mixed transcriptional profiles normalized to logCPMs
- A SpatialDDLS object containing the simulated data for downstream training

## How to apply

Build a SpatialDDLS object with createSpatialDDLSobject, applying logFC-based feature selection (sc.filt.genes.cluster = TRUE, sc.n.genes.per.cluster = 150, sc.min.mean.counts = 2), which after removing 11 mitochondrial genes yields 1041 shared genes from 10941 shared genes between the single-cell and spatial datasets. Call genMixedCellProp with num.sim.spots = 10000, n.cells = 50, min.zero.prop = 5, and balanced.type.cells = TRUE to split reference cells into train/test subsets and sample Dirichlet-based proportions across 12 cell types, yielding 7500 train and 2500 test composition matrices with sparsity, pure profiles, and minimum-missing-type profiles. Then run simMixedProfiles (threads = 3), which sums raw counts of the sampled cells and normalizes to logCPMs to produce the mixed transcriptional profiles used for trainDeconvModel.

## Related tools

- **SpatialDDLS** (Provides createSpatialDDLSobject, genMixedCellProp, and simMixedProfiles to simulate mixed transcriptional profiles with known cell composition) — https://CRAN.R-project.org/package=SpatialDDLS
- **SpatialDDLSdata** (Supplies the MouseDLN.SCE scRNA-seq and MouseDLN.ST spatial example datasets used for simulation) — github.com/diegommcc/SpatialDDLSdata
- **SingleCellExperiment** (Container format for the scRNA-seq reference loaded into the SpatialDDLS object)
- **SpatialExperiment** (Container format for the spatial transcriptomics data used to select shared genes)

## Examples

```
mouseDLN.SDDLS <- genMixedCellProp(mouseDLN.SDDLS, num.sim.spots = 10000, n.cells = 50, min.zero.prop = 5, balanced.type.cells = TRUE); mouseDLN.SDDLS <- simMixedProfiles(mouseDLN.SDDLS, threads = 3)
```

## Evaluation signals

- Feature selection reports 1041 genes after logFC filtering and 11 removed mitochondrial genes, from 10941 shared genes between single-cell and spatial datasets
- genMixedCellProp yields exactly 7500 train and 2500 test mixed spots across 12 cell types
- Downstream model on held-out 2500 simulated profiles reaches loss ≈ 0.0818, accuracy ≈ 0.87, MAE ≈ 0.017
- Predicted vs expected proportions on simulated test profiles give mean PCC = 0.99 and mean CCC = 0.98 (Supplementary Fig. S3b)

## Limitations

- Simulated compositions are Dirichlet-sampled approximations; performance on real spatial tissue may not match simulated-profile metrics
- Simulation requires a Python interpreter with Tensorflow/Keras installed for the downstream SpatialDDLS workflow (installTFpython sets up the spatialddls-env conda environment)
- Gene feature selection discards genes (e.g. mitochondrial genes, genes without expression in any cell), limiting the simulated profile feature space

## Evidence

- [abstract] The package takes advantage of single-cell RNA sequencing data to simulate mixed transcriptional profiles with predefined cellular composition: [evidence span withheld — anchor status: not_found]
- [methods] This is done by the `genMixedCellProp` function: "This is done by the `genMixedCellProp` function"
- [methods] mouseDLN.SDDLS <- simMixedProfiles(mouseDLN.SDDLS, threads = 3): "mouseDLN.SDDLS <- simMixedProfiles(mouseDLN.SDDLS, threads = 3)"
- [methods] sc.filt.genes.cluster = TRUE, sc.n.genes.per.cluster = 150, sc.min.mean.counts = 2: "sc.filt.genes.cluster = TRUE, 
  sc.n.genes.per.cluster = 150,
  sc.min.mean.counts = 2"
- [methods] === Number of shared genes between single-cell and spatial transcriptomics datasets: 10941: "=== Number of shared genes between single-cell and spatial transcriptomics datasets: 10941"
- [readme] spatialddls-env conda environment: [evidence span withheld — anchor status: external_doc]
