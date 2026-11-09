---
name: spatial-regularization-alpha-blending
description: Use when you already have a trained SpatialDDLS model and spot-level
  intrinsic predictions and want to spatially regularize proportions using neighborhood
  information (10x Visium spots with spatial coordinates).
when_to_use_negative:
- Datasets without reliable spatial neighborhood structure, since the continuity assumption
  behind extrinsic profiles may not hold — the article marks this step as optional
- Before a deconvolution model has been trained (use genMixedCellProp/simMixedProfiles/trainDeconvModel
  first)
- When you only need raw per-spot proportions and want to avoid the smoothing induced
  by extrinsic predictions
tools:
- name: SpatialDDLS
  role: Provides deconvSpatialDDLS to predict intrinsic and extrinsic proportions
    (k.spots = 6, fast.pca = TRUE, 50 PCs) and compute the alpha-factor-based spatial
    regularization
  repo: https://CRAN.R-project.org/package=SpatialDDLS
- name: R
  role: Runtime environment for running the SpatialDDLS pipeline and exporting proportions
    to CSV
- name: SpatialExperiment
  role: Container class for the spatial transcriptomics data (e.g. MouseDLN.ST) loaded
    into createSpatialDDLSobject
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btae072
    title: 'SpatialDDLS: an R package to deconvolute spatial transcriptomics data
      using neural networks'
  source_content_hash: sha256:90a00d05db3f152224c4da869c1a096fa1fec8bede1ddb9852672b511255630e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - SpatialDDLS
  - SpatialExperiment
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# spatial-regularization-alpha-blending

## Summary

Optional post-deconvolution step in SpatialDDLS that blends predicted intrinsic cell-type proportions with extrinsic predictions simulated from the k nearest spots (k.spots = 6), weighting each spot by an alpha factor derived from the PCA-space distance between its intrinsic and extrinsic profiles.

## When to use

Use when you already have a trained SpatialDDLS model and spot-level intrinsic predictions and want to spatially regularize proportions using neighborhood information (10x Visium spots with spatial coordinates).

## When NOT to use

- Datasets without reliable spatial neighborhood structure, since the continuity assumption behind extrinsic profiles may not hold — the article marks this step as optional
- Before a deconvolution model has been trained (use genMixedCellProp/simMixedProfiles/trainDeconvModel first)
- When you only need raw per-spot proportions and want to avoid the smoothing induced by extrinsic predictions

## Inputs

- Trained SpatialDDLS model object (mouseDLN.SDDLS) with a fitted neural network
- SpatialExperiment spatial transcriptomics data (e.g. MouseDLN.ST from GSE173778, 10x Visium)
- Spatial spot coordinates for k-nearest-neighbor identification
- logCPM-normalized shared gene expression matrix

## Outputs

- Spot-by-cell-type matrix of 'Intrinsic-regularized' proportions
- Unregularized intrinsic and extrinsic proportion matrices
- Exported deconvoluted_proportions.csv (proportions for all spots and 12 cell types)

## How to apply

Normalize the spatial data to logCPM (as deconvSpatialDDLS does with normalize = TRUE), then run deconvSpatialDDLS on the trained SpatialDDLS object: the neural network predicts proportions for intrinsic profiles (each spot's own transcriptional profile) and for extrinsic profiles simulated from the k.spots = 6 surrounding spots (with fast.pca = TRUE, using 50 PCs in PCA space). Distances are computed in PCA space between each spot's intrinsic and extrinsic profiles and converted into alpha factors; spots whose distance is below the mean distance are regularized toward their nearest-neighbor-based (extrinsic) predictions. Export the spot-by-cell-type matrix with the 'Intrinsic-regularized' proportions alongside the unregularized intrinsic and extrinsic proportions. Treat the step as optional because the spatial-continuity assumption may not hold for all datasets.

## Related tools

- **SpatialDDLS** (Provides deconvSpatialDDLS to predict intrinsic and extrinsic proportions (k.spots = 6, fast.pca = TRUE, 50 PCs) and compute the alpha-factor-based spatial regularization) — https://CRAN.R-project.org/package=SpatialDDLS
- **R** (Runtime environment for running the SpatialDDLS pipeline and exporting proportions to CSV)
- **SpatialExperiment** (Container class for the spatial transcriptomics data (e.g. MouseDLN.ST) loaded into createSpatialDDLSobject)

## Examples

```
mouseDLN.SDDLS <- deconvSpatialDDLS(mouseDLN.SDDLS, k.spots = 6, fast.pca = TRUE)
```

## Evaluation signals

- The output CSV contains proportions for all spots and 12 cell types
- Extrinsic/regularized predictions behave as a smoothed version of the intrinsic ones
- Spots with intrinsic–extrinsic distance below the mean distance show regularization toward their neighbors' predictions
- The exported matrix includes both 'Intrinsic-regularized' and unregularized intrinsic (and extrinsic) columns
- Predicted proportions colocalize with expression of known cell type markers

## Limitations

- The spatial-continuity assumption underlying extrinsic profiles may not hold for all datasets, so regularization is optional
- Results depend on the k-nearest-neighbor choice (k.spots = 6) and on PCA-space distances (fast.pca = TRUE, 50 PCs)
- Requires a working Python/Tensorflow installation for the keras-backed neural network predictions

## Evidence

- [methods] Extrinsic predictions are smoothed via neighbor-based regularization: "the extrinsic predictions are a smoothed version of the final ones"
- [methods] Spatial regularization is named by the article itself: "We refer to this process as spatial regularization."
- [readme] Trained models are applied to new spatial transcriptomics data to predict proportions: "The resulting trained models can be applied to new spatial transcriptomics data to predict cell type proportions, allowing for a more accurate cell type identification"
- [methods] The dataset used is 10x Visium murine lymph nodes: "a spatial transcriptomics dataset (10x Visium) comprising three slides from murine lymph nodes"
