---
name: scrnaseq-mixed-transcriptional-profile-simulation
description: Use when a paired scRNA-seq and spatial transcriptomics dataset is available
  and cell type proportions must be simulated with known ground truth to train or
  benchmark a deconvolution model (e.g., SpatialDDLS on 10x Visium/Chromium data).
when_to_use_negative:
- The input is already a mixed bulk or spatial profile with measured cell type proportions
  rather than a scRNA-seq reference needing simulation
- The scRNA-seq and spatial datasets share too few genes after filtering to support
  meaningful profile mixing (shared-gene intersection is the limiting step)
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: SpatialDDLS
  role: Provides genMixedCellProp and simMixedProfiles to generate cell composition
    matrices and simulate mixed transcriptional profiles from scRNA-seq
  repo: https://github.com/diegommcc/SpatialDDLS
- name: SpatialDDLSdata
  role: Supplies the MouseDLN.SCE and MouseDLN.ST example datasets used to demonstrate
    profile simulation
  repo: https://github.com/diegommcc/SpatialDDLSdata
- name: SingleCellExperiment
  role: Container for the scRNA-seq reference used as input to createSpatialDDLSobject
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btae072
    title: 'SpatialDDLS: an R package to deconvolute spatial transcriptomics data
      using neural networks'
  source_content_hash: sha256:3fbd2e76e1b8bb7118805106584280d0b431b2274864cfe670e32e516bec07a1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/diegommcc/SpatialDDLS
  tools:
  - R
  - SingleCellExperiment
  - SpatialDDLS
  - SpatialDDLSdata
  - SpatialExperiment
  - devtools
  - ggplot2
  - ggpubr
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# scrnaseq-mixed-transcriptional-profile-simulation

## Summary

Simulate mixed transcriptional profiles with known cell type composition from scRNA-seq reference data (e.g., MouseDLN.SCE from GEO GSE173778) using SpatialDDLS's genMixedCellProp and simMixedProfiles, producing labeled train/test spots for training and evaluating a cell-type deconvolution neural network.

## When to use

Use when a paired scRNA-seq and spatial transcriptomics dataset is available and cell type proportions must be simulated with known ground truth to train or benchmark a deconvolution model (e.g., SpatialDDLS on 10x Visium/Chromium data).

## When NOT to use

- The input is already a mixed bulk or spatial profile with measured cell type proportions rather than a scRNA-seq reference needing simulation
- The scRNA-seq and spatial datasets share too few genes after filtering to support meaningful profile mixing (shared-gene intersection is the limiting step)

## Inputs

- SingleCellExperiment object with paired scRNA-seq data (e.g., MouseDLN.SCE from GEO GSE173778)
- SpatialExperiment object with spatial transcriptomics data (e.g., MouseDLN.ST, 10x Visium slides)
- SpatialDDLS object created via createSpatialDDLSobject

## Outputs

- Simulated mixed transcriptional profiles (logCPM-normalized) with known cell type composition
- Cell composition matrices split into train (7500 spots) and test (2500 spots) subsets
- SpatialDDLS object populated with simulated.profiles and cell.comp matrices ready for trainDeconvModel

## How to apply

Load the scRNA-seq reference as a SingleCellExperiment and the ST data as a SpatialExperiment into a SpatialDDLS object with createSpatialDDLSobject (sc.filt.genes.cluster=TRUE, sc.n.genes.per.cluster=150, sc.min.mean.counts=2), which removes mitochondrial genes and genes without expression, selects shared genes (1041 after logFC filtering in the mouse example), and yields ~1041 shared filtered genes. Call genMixedCellProp (num.sim.spots=10000, n.cells=50, min.zero.prop=5, balanced.type.cells=TRUE) to generate cell composition matrices via Dirichlet, pure, and minimum-sparsity sampling strategies, splitting into 7500 train and 2500 test spots. Then run simMixedProfiles (threads=3) which sums raw counts across the sampled cells and normalizes to logCPMs, producing simulated mixed profiles with known proportions for network training and held-out evaluation.

## Related tools

- **SpatialDDLS** (Provides genMixedCellProp and simMixedProfiles to generate cell composition matrices and simulate mixed transcriptional profiles from scRNA-seq) — https://github.com/diegommcc/SpatialDDLS
- **SpatialDDLSdata** (Supplies the MouseDLN.SCE and MouseDLN.ST example datasets used to demonstrate profile simulation) — https://github.com/diegommcc/SpatialDDLSdata
- **SingleCellExperiment** (Container for the scRNA-seq reference used as input to createSpatialDDLSobject)

## Examples

```
library(SpatialDDLS); mouseDLN.SDDLS <- simMixedProfiles(mouseDLN.SDDLS, threads = 3)
```

## Evaluation signals

- Simulated profiles sum to the expected spot count: 10000 total spots split into 7500 train / 2500 test subsets
- Gene space matches the filtered shared-gene set (1041 genes in the mouse hippocampus/lymph node example after logFC filtering and removal of 11 mitochondrial and 16 zero-expression genes)
- Test-set MAE of a downstream model trained on these profiles is around 0.017
- Model evaluated on the held-out simulated test profiles reaches mean PCC ≈ 0.97 and mean CCC ≈ 0.97 (mouse hippocampus), matching Supplementary Fig. S1b

## Limitations

- Simulated profiles assume cell type proportions sampled via Dirichlet, pure, and minimum-sparsity strategies; proportions with a minimum number of missing cell types are imposed via min.zero.prop, which may not reflect all real tissue compositions
- Simulation is limited to genes shared between the single-cell and spatial transcriptomics datasets (10941 shared genes before filtering in the example)
- Requires a working Python interpreter with TensorFlow installed (via installTFpython) because training depends on tensorflow/keras R packages

## Evidence

- [readme] The package takes advantage of single-cell RNA sequencing (scRNA-seq) data to simulate mixed transcriptional profiles with known cell composition and train fully-connected neural networks to predict: "The package takes advantage of single-cell RNA sequencing (scRNA-seq) data to simulate mixed transcriptional profiles with known cell composition"
- [methods] mouseDLN.SDDLS <- simMixedProfiles(mouseDLN.SDDLS, threads = 3): [evidence span withheld — anchor status: not_found]
- [methods] This is done by the `genMixedCellProp` function: [evidence span withheld — anchor status: not_found]
- [methods] === Number of genes after filtering based on logFC: 1041: "=== Number of genes after filtering based on logFC: 1041"
- [abstract] SpatialDDLS leverages single-cell RNA sequencing data to simulate mixed transcriptional profiles with predefined cellular composition: "SpatialDDLS leverages single-cell RNA sequencing data to simulate mixed transcriptional profiles with predefined cellular composition"
