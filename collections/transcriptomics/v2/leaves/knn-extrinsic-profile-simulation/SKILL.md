---
name: knn-extrinsic-profile-simulation
description: Use when deconvolving spatial transcriptomics data (e.g. 10x Visium spots)
  with a trained SpatialDDLS model and you want neighbor-aware, spatially regularized
  cell type proportions for each spot.
when_to_use_negative:
- Datasets where the spatial continuity assumption does not hold (e.g. disorganized
  or dissociated spatial architecture) — spatial regularization is optional and may
  mislead
- Deconvolution tasks on bulk RNA-seq or non-spatial single-cell data with no spot
  coordinates, since no k-nearest spots exist
- When only unregularized intrinsic per-spot predictions are desired, extrinsic simulation
  adds unnecessary computation
tools:
- name: SpatialDDLS
  role: Provides deconvSpatialDDLS to simulate extrinsic profiles from k.spots neighboring
    spots, predict intrinsic/extrinsic proportions with the trained neural network,
    and apply spatial regularization
  repo: https://CRAN.R-project.org/package=SpatialDDLS
- name: SpatialExperiment
  role: Holds the 10x Visium spatial transcriptomics data (spots plus coordinates)
    that serve as input to the deconvolution
- name: R
  role: Runtime environment in which the SpatialDDLS package and neural network (Tensorflow/Keras
    backend) are executed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1093/bioinformatics/btae072
    title: 'SpatialDDLS: an R package to deconvolute spatial transcriptomics data
      using neural networks'
  source_content_hash: sha256:1e11229b20129a3f97875d4dacd746983bceab2d86805eaea123207abf8a374e
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
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# knn-extrinsic-profile-simulation

## Summary

Simulates extrinsic transcriptional profiles for each spatial transcriptomics spot from its k-nearest neighbouring spots (k.spots = 6) and uses a trained SpatialDDLS neural network to predict extrinsic cell type proportions, which then spatially regularize the intrinsic per-spot predictions.

## When to use

Use when deconvolving spatial transcriptomics data (e.g. 10x Visium spots) with a trained SpatialDDLS model and you want neighbor-aware, spatially regularized cell type proportions for each spot.

## When NOT to use

- Datasets where the spatial continuity assumption does not hold (e.g. disorganized or dissociated spatial architecture) — spatial regularization is optional and may mislead
- Deconvolution tasks on bulk RNA-seq or non-spatial single-cell data with no spot coordinates, since no k-nearest spots exist
- When only unregularized intrinsic per-spot predictions are desired, extrinsic simulation adds unnecessary computation

## Inputs

- SpatialDDLS model object with a trained fully-connected neural network
- SpatialExperiment spatial transcriptomics object (e.g. MouseDLN.ST from GSE173778), normalized to logCPM
- Spot spatial coordinates for k-nearest-neighbor computation
- k.spots (default k = 6), fast.pca = TRUE, number of PCA dimensions (50)

## Outputs

- Spot-by-cell-type matrix of 'Intrinsic-regularized' cell type proportions
- Unregularized intrinsic proportions per spot
- Extrinsic proportions per spot
- deconvoluted_proportions.csv export

## How to apply

After normalizing the spatial data to logCPM via deconvSpatialDDLS (normalize = TRUE), simulate an extrinsic transcriptional profile for every spot from its k.spots = 6 surrounding spots, optionally projected into a reduced PCA space (fast.pca = TRUE, 50 PCs) for speed, and run these profiles through the trained neural network to obtain extrinsic cell type proportions. Compute distances in PCA space between each spot's intrinsic profile (its actual transcriptional profile) and its extrinsic profile; spots whose distance falls below the mean distance are regularized toward their neighbor-based predictions, yielding 'Intrinsic-regularized' proportions. The rationale is that the extrinsic predictions act as a smoothed version of the final ones, borrowing local spatial context while weighting how similar each spot is to its surroundings. Note this step is optional because the spatial continuity assumption may not hold for all datasets.

## Related tools

- **SpatialDDLS** (Provides deconvSpatialDDLS to simulate extrinsic profiles from k.spots neighboring spots, predict intrinsic/extrinsic proportions with the trained neural network, and apply spatial regularization) — https://CRAN.R-project.org/package=SpatialDDLS
- **SpatialExperiment** (Holds the 10x Visium spatial transcriptomics data (spots plus coordinates) that serve as input to the deconvolution)
- **R** (Runtime environment in which the SpatialDDLS package and neural network (Tensorflow/Keras backend) are executed)

## Examples

```
mouseDLN.SDDLS <- deconvSpatialDDLS(mouseDLN.SDDLS, normalize = TRUE, k.spots = 6, fast.pca = TRUE)
```

## Evaluation signals

- Output contains proportions for all spots and all 12 cell types
- Extrinsic and regularized predictions behave as a smoothed version of the intrinsic ones
- Spots with intrinsic-to-extrinsic PCA distance below the mean distance show regularization toward their neighbors' predictions
- deconvoluted_proportions.csv contains a spot-by-cell-type matrix with Intrinsic-regularized, intrinsic, and extrinsic columns
- Predicted cell type proportions colocalize with expression of known cell type markers

## Limitations

- Spatial regularization is optional and should be skipped when the continuity assumption does not hold for the dataset
- Requires a working Python interpreter with Tensorflow/Keras installed (e.g. via installTFpython with the spatialddls-env conda environment)
- Neighborhood choice (k.spots) and PCA projection (fast.pca, 50 PCs) affect the smoothing behavior of extrinsic predictions

## Evidence

- [methods] extrinsic smoothing claim: "the extrinsic predictions are a smoothed version of the final ones"
- [methods] spatial regularization naming: "We refer to this process as spatial regularization."
- [methods] spatial regularization in deconvolution workflow: [evidence span withheld — anchor status: not_found]
- [readme] trained models predict per-spot proportions: "The resulting trained models can be applied to new spatial transcriptomics data to predict cell type proportions"
- [abstract] NN-based deconvolution framing: "we introduce SpatialDDLS, a fast neural network-based algorithm for cell type deconvolution of spatial transcriptomics data"
