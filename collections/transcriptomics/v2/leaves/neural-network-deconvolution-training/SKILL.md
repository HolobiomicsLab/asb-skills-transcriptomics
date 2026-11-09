---
name: neural-network-deconvolution-training
description: Use when paired annotated scRNA-seq and spot-based spatial transcriptomics
  data (e.g. 10x Chromium + 10x Visium from GSE173778) are available and cell type
  proportions per spot must be learned from simulated mixtures.
when_to_use_negative:
- No paired annotated scRNA-seq reference exists for the tissue — simulated mixed
  profiles with known composition cannot be generated from single-cell data alone
  is required.
- A trained model already exists and the goal is only to predict proportions on new
  spatial data (apply via deconvSpatialDDLS instead of retraining).
- Input is bulk RNA-seq or already-estimated cell proportions rather than spot-resolution
  spatial transcriptomics with a paired single-cell reference.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: SpatialDDLS
  role: 'Provides the full training pipeline: createSpatialDDLSobject, genMixedCellProp,
    simMixedProfiles, trainDeconvModel, and calculateEvalMetrics on a SpatialDDLS
    object'
  repo: https://github.com/diegommcc/SpatialDDLS
- name: SpatialDDLSdata
  role: Supplies the MouseDLN.SCE (scRNA-seq) and MouseDLN.ST (spatial) example datasets
    for the mouse lymph node workflow
  repo: https://github.com/diegommcc/SpatialDDLSdata
- name: keras
  role: Backend on which trainDeconvModel builds and trains the fully-connected network;
    requires Python TensorFlow, installed via installTFpython into the spatialddls-env
    conda environment
  repo: https://cran.r-project.org/package=keras
- name: SingleCellExperiment
  role: Container format for the annotated scRNA-seq reference (MouseDLN.SCE) used
    to simulate mixed profiles
- name: SpatialExperiment
  role: Container format for the 10x Visium spatial transcriptomics data (MouseDLN.ST)
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btae072
    title: 'SpatialDDLS: an R package to deconvolute spatial transcriptomics data
      using neural networks'
  source_content_hash: sha256:260b07e1b6be93ee1c0d9a20598a5344d8701152cab210b1766279f549a3de80
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
  - keras
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 22
---

# neural-network-deconvolution-training

## Summary

Train a fully-connected neural network (keras/TensorFlow backend) on simulated mixed transcriptional profiles with known cell composition, generated from paired scRNA-seq and spatial transcriptomics data via the SpatialDDLS R package, so the resulting model can deconvolute cell type proportions in spatial spots.

## When to use

Use when paired annotated scRNA-seq and spot-based spatial transcriptomics data (e.g. 10x Chromium + 10x Visium from GSE173778) are available and cell type proportions per spot must be learned from simulated mixtures.

## When NOT to use

- No paired annotated scRNA-seq reference exists for the tissue — simulated mixed profiles with known composition cannot be generated from single-cell data alone is required.
- A trained model already exists and the goal is only to predict proportions on new spatial data (apply via deconvSpatialDDLS instead of retraining).
- Input is bulk RNA-seq or already-estimated cell proportions rather than spot-resolution spatial transcriptomics with a paired single-cell reference.

## Inputs

- SingleCellExperiment object with annotated scRNA-seq reference counts (e.g. MouseDLN.SCE from SpatialDDLSdata, 10x Chromium data of GSE173778)
- SpatialExperiment object with spot-level spatial transcriptomics counts (e.g. MouseDLN.ST, 10x Visium)
- Shared gene space between single-cell and spatial data after logFC/mitochondrial filtering

## Outputs

- Trained fully-connected keras/TensorFlow neural network stored in the SpatialDDLS object, evaluated on 2500 held-out simulated test profiles
- Per-cell-type evaluation metrics table (MAE, MSE; e.g. model_evaluation_metrics.csv)
- Expected-vs-predicted correlation panels (PCC and CCC per cell type) and error boxplots rendered with ggplot2/ggpubr

## How to apply

Load the scRNA-seq reference (e.g. MouseDLN.SCE, a SingleCellExperiment) and spatial data (MouseDLN.ST, a SpatialExperiment) into a SpatialDDLS object with createSpatialDDLSobject, enabling logFC-based feature selection (sc.filt.genes.cluster = TRUE, sc.n.genes.per.cluster = 150, sc.min.mean.counts = 2), which for the mouse lymph node data leaves 1041 shared genes after removing 11 mitochondrial genes and 16 genes without expression. Generate cell composition matrices with genMixedCellProp (num.sim.spots = 10000, n.cells = 50, min.zero.prop = 5, balanced.type.cells = TRUE), which Dirichlet-samples 7500 training and 2500 test mixed spots across 12 cell types including pure and minimum-missing-type (sparsity) profiles, then realize the actual mixed transcriptional profiles with simMixedProfiles (threads = 3) by summing raw counts and normalizing to logCPMs. Train the fully-connected network with trainDeconvModel (two hidden layers of 200 neurons, 60 epochs) and evaluate it on the held-out 2500 simulated test profiles, since the known simulated compositions provide ground truth. Quantify performance with calculateEvalMetrics (per-cell-type MAE and MSE), distErrorPlot (error = 'AbsErr', x.by = 'CellType', type = 'boxplot'), and corrExpPredPlot (color.by = 'CellType', corr = 'both') to obtain Pearson (PCC) and Lin concordance (CCC) coefficients per cell type. Accept the model when held-out test metrics are low (vignette reference: loss ≈ 0.0818, accuracy ≈ 0.87, MAE ≈ 0.017) and mean correlations approach the reported PCC = 0.99 / CCC = 0.98 for mouse lymph node (PCC = 0.97 / CCC = 0.97 for hippocampus).

## Related tools

- **SpatialDDLS** (Provides the full training pipeline: createSpatialDDLSobject, genMixedCellProp, simMixedProfiles, trainDeconvModel, and calculateEvalMetrics on a SpatialDDLS object) — https://github.com/diegommcc/SpatialDDLS
- **SpatialDDLSdata** (Supplies the MouseDLN.SCE (scRNA-seq) and MouseDLN.ST (spatial) example datasets for the mouse lymph node workflow) — https://github.com/diegommcc/SpatialDDLSdata
- **keras** (Backend on which trainDeconvModel builds and trains the fully-connected network; requires Python TensorFlow, installed via installTFpython into the spatialddls-env conda environment) — https://cran.r-project.org/package=keras
- **SingleCellExperiment** (Container format for the annotated scRNA-seq reference (MouseDLN.SCE) used to simulate mixed profiles)
- **SpatialExperiment** (Container format for the 10x Visium spatial transcriptomics data (MouseDLN.ST))

## Examples

```
library("SpatialDDLS"); mouseDLN.SDDLS <- simMixedProfiles(mouseDLN.SDDLS, threads = 3); mouseDLN.SDDLS <- trainDeconvModel(mouseDLN.SDDLS)
```

## Evaluation signals

- Feature-selection console output matches the paper: '=== Number of genes after filtering based on logFC: 1041' and '=== Number of removed mitochondrial genes: 11', plus 16 genes without expression removed.
- genMixedCellProp splits reference cells into 7500 training and 2500 test simulated spots across 12 cell types (Dirichlet, pure, and minimum-missing-type profiles).
- Held-out test metrics on the 2500 simulated profiles are low: loss ≈ 0.0818, accuracy ≈ 0.87, mean_absolute_error ≈ 0.017.
- corrExpPredPlot with corr = 'both' yields per-cell-type PCC/CCC with means close to the reported PCC = 0.99 / CCC = 0.98 for lymph node (Supplementary Fig. S3b) and PCC = 0.97 / CCC = 0.97 for hippocampus (Supplementary Fig. S1b).
- distErrorPlot boxplots of AbsErr grouped by CellType show low, compact per-cell-type error distributions, and predicted proportions colocalize with known cell type marker expression.

## Limitations

- Reported accuracy (mean PCC 0.97–0.99, CCC 0.97–0.98) is measured on simulated mixed transcriptional profiles with known composition, not on experimentally quantified real tissue compositions.
- The keras/TensorFlow backend needs a working Python interpreter; the README recommends installTFpython(install.conda = TRUE) to build the spatialddls-env conda environment.
- Results depend on filtering and simulation parameters (sc.n.genes.per.cluster = 150, sc.min.mean.counts = 2, min.zero.prop = 5, n.cells = 50); deviating from them changes the 1041-gene shared set and the train/test composition split.

## Evidence

- [abstract] we introduce SpatialDDLS, a fast neural network-based algorithm for cell type deconvolution of spatial transcriptomics data: "we introduce SpatialDDLS, a fast neural network-based algorithm for cell type deconvolution of spatial transcriptomics data"
- [intro] The package takes advantage of single-cell RNA sequencing data to simulate mixed transcriptional profiles: "The package takes advantage of single-cell RNA sequencing (scRNA-seq) data to simulate mixed transcriptional profiles with known cell composition"
- [methods] we can then train a neural network using the training subset and evaluate the model by predicting cell type proportions on the test subset: "we can then train a neural network using the training subset and evaluate the model by predicting cell type proportions on the test subset"
- [methods] This function computes mean absolute error (MAE) and mean squared error (MSE) metrics per cell type: "This function computes mean absolute error (MAE) and mean squared error (MSE) metrics per cell type"
- [methods] === Number of genes after filtering based on logFC: 1041: "=== Number of genes after filtering based on logFC: 1041"
- [readme] The `installTFpython` function provides an easy way to install a conda environment named `spatialddls-env` with all necessary dependencies covered.: [evidence span withheld — anchor status: external_doc]
