---
name: neural-network-proportion-prediction
description: Use when a trained SpatialDDLS model must predict cell type composition
  for 10x Visium spots, or when per-spot proportions need neighbor-aware spatial regularization
  using k.spots = 6 extrinsic profiles.
when_to_use_negative:
- The spatial dataset lacks coherent spatial structure or neighbors, since the spatial
  regularization assumes continuity that may not hold for all datasets (regularization
  should then be skipped).
- No scRNA-seq reference from the same study is available to simulate mixed transcriptional
  profiles and train the deconvolution network.
- The spatial platform is not spot-based (e.g. single-cell resolution spatial data),
  where per-spot composition prediction is not the target.
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_3170
tools:
- name: SpatialDDLS
  role: R package providing the neural network training (trainDeconvModel) and deconvolution
    with spatial regularization (deconvSpatialDDLS) used to predict per-spot cell
    type proportions
  repo: https://github.com/diegommcc/SpatialDDLS
- name: SpatialDDLSdata
  role: Provides the example MouseDLN.SCE scRNA-seq and MouseDLN.ST spatial datasets
    used to demonstrate proportion prediction
  repo: https://github.com/diegommcc/SpatialDDLSdata
- name: SpatialExperiment
  role: Holds the spatial transcriptomics input data loaded into the SpatialDDLS object
- name: tensorflow
  role: Backend dependency required to run the fully-connected neural network for
    proportion prediction
  repo: https://cran.r-project.org/package=tensorflow
- name: keras
  role: Backend dependency required to run the fully-connected neural network for
    proportion prediction
  repo: https://cran.r-project.org/package=keras
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btae072
    title: 'SpatialDDLS: an R package to deconvolute spatial transcriptomics data
      using neural networks'
  source_content_hash: sha256:d8a4a6628eddc6b02f0e3d0fd15ce345fb79406ceebf52391b50962af755724d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/diegommcc/SpatialDDLS
  tools:
  - R
  - SpatialDDLS
  - SpatialDDLSdata
  - SpatialExperiment
  - keras
  - tensorflow
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# neural-network-proportion-prediction

## Summary

Train a fully-connected neural network on simulated mixed transcriptional profiles derived from scRNA-seq data, then apply the trained model to spatial transcriptomics spots to predict per-spot cell type proportions, optionally refining them via spatial regularization using extrinsic (k-nearest-spot) predictions.

## When to use

Use when a trained SpatialDDLS model must predict cell type composition for 10x Visium spots, or when per-spot proportions need neighbor-aware spatial regularization using k.spots = 6 extrinsic profiles.

## When NOT to use

- The spatial dataset lacks coherent spatial structure or neighbors, since the spatial regularization assumes continuity that may not hold for all datasets (regularization should then be skipped).
- No scRNA-seq reference from the same study is available to simulate mixed transcriptional profiles and train the deconvolution network.
- The spatial platform is not spot-based (e.g. single-cell resolution spatial data), where per-spot composition prediction is not the target.

## Inputs

- Trained SpatialDDLS model object (keras/tensorflow fully-connected neural network)
- SpatialExperiment object with 10x Visium spatial transcriptomics data (e.g. MouseDLN.ST from GSE173778)
- SingleCellExperiment scRNA-seq reference (e.g. MouseDLN.SCE from 10x Chromium) used for training simulations
- k.spots parameter (k = 6) and PCA settings (fast.pca = TRUE, 50 PCs) for extrinsic profile simulation

## Outputs

- Spot-by-cell-type matrix of unregularized intrinsic predicted proportions
- Extrinsic (neighbor-simulated) predicted proportions
- Spatially regularized ('Intrinsic-regularized') proportions, e.g. exported to deconvoluted_proportions.csv

## How to apply

Load the trained SpatialDDLS model object and the SpatialExperiment spatial data (e.g. MouseDLN.ST from GSE173778) via createSpatialDDLSobject, normalizing to logCPM as deconvSpatialDDLS does (normalize = TRUE). Predict intrinsic proportions for every spot with the trained neural network. For regularization, simulate extrinsic transcriptional profiles from the k.spots = 6 nearest spots of each spot in PCA space (fast.pca = TRUE, 50 PCs) and predict extrinsic proportions with the same network. Compute distances in PCA space between each spot's intrinsic and extrinsic profiles; spots whose distance falls below the mean distance are regularized toward their nearest-neighbor-based predictions to yield 'Intrinsic-regularized' proportions. The regularization step is optional because the spatial continuity assumption may not hold for all datasets, so export both regularized and unregularized intrinsic (and extrinsic) proportions for downstream comparison.

## Related tools

- **SpatialDDLS** (R package providing the neural network training (trainDeconvModel) and deconvolution with spatial regularization (deconvSpatialDDLS) used to predict per-spot cell type proportions) — https://github.com/diegommcc/SpatialDDLS
- **SpatialDDLSdata** (Provides the example MouseDLN.SCE scRNA-seq and MouseDLN.ST spatial datasets used to demonstrate proportion prediction) — https://github.com/diegommcc/SpatialDDLSdata
- **SpatialExperiment** (Holds the spatial transcriptomics input data loaded into the SpatialDDLS object)
- **tensorflow** (Backend dependency required to run the fully-connected neural network for proportion prediction) — https://cran.r-project.org/package=tensorflow
- **keras** (Backend dependency required to run the fully-connected neural network for proportion prediction) — https://cran.r-project.org/package=keras

## Examples

```
library(SpatialDDLS); mouseDLN.SDDLS <- createSpatialDDLSobject(...); mouseDLN.SDDLS <- trainDeconvModel(object = mouseDLN.SDDLS); deconvSpatialDDLS(mouseDLN.SDDLS, normalize = TRUE, k.spots = 6, fast.pca = TRUE)
```

## Evaluation signals

- Output must contain predicted proportions for all spots and all 12 cell types of the reference.
- Extrinsic predictions must behave as a smoothed version of the final (intrinsic) predictions.
- Spots with intrinsic-extrinsic PCA distances less than the mean distance must show regularization toward their neighbors' predictions.
- Model performance should be checked with calculateEvalMetrics (MAE and MSE per cell type); overall performance must be satisfactory.
- Predicted proportions must colocalize with expression of known cell type markers.

## Limitations

- Spatial regularization assumes spatial continuity, which may not hold for all datasets; the regularization step is optional and can be skipped.
- Requires a working Python interpreter with the TensorFlow library installed (installTFpython provides a conda environment 'spatialddls-env').
- Prediction quality depends on the scRNA-seq reference and on gene filtering parameters (sc.n.genes.per.cluster, sc.min.mean.counts, logFC filtering) and on the number of shared genes between the single-cell and spatial datasets.

## Evidence

- [intro] Neural network trained on simulated mixtures predicts spot compositions: "The package takes advantage of single-cell RNA sequencing (scRNA-seq) data to simulate mixed transcriptional profiles with known cell composition and train fully-connected neural networks to predict"
- [methods] Deconvolution with spatial regularization: "We refer to this process as spatial regularization."
- [methods] Extrinsic predictions are smoothed versions of final predictions: "the extrinsic predictions are a smoothed version of the final ones"
- [methods] Prediction quality depends on gene filtering and shared genes: "=== Number of shared genes between single-cell and spatial transcriptomics datasets: 10941"
- [readme] TensorFlow dependency for the neural network backend: [evidence span withheld — anchor status: external_doc]
