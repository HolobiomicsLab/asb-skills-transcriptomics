---
name: aggregated-gradient-heatmap-visualization
description: 'Use when a trained SpatialDDLS deconvolution model exists and the goal
  is model interpretation: ranking which genes drive predicted cell type proportions
  and visualizing top per-cell-type gradient markers as a heatmap.'
when_to_use_negative:
- The SpatialDDLS model has not yet been trained (no trainDeconvModel output) — gradients
  require a fitted neural network over simulated mixed profiles
- The task is deconvolution itself (predicting cell type proportions from spatial
  data via deconvSpatialDDLS with spatial regularization) rather than interpreting
  an already-trained model
edam_operation: http://edamontology.org/operation_2422
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3308
tools:
- name: SpatialDDLS
  role: Provides the interpretation functions interGradientsDL, topGradientsCellType,
    and plotHeatmapGradsAgg that compute, rank, and plot per-cell-type gradients from
    the trained model
  repo: https://github.com/diegommcc/SpatialDDLS
- name: keras
  role: TensorFlow/Keras backend dependency on which the neural network gradients
    are computed
  repo: https://cran.r-project.org/package=keras
- name: ggplot2
  role: Plotting library underlying the aggregated gradient heatmap rendering
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae072
    title: 'SpatialDDLS: an R package to deconvolute spatial transcriptomics data
      using neural networks'
  source_content_hash: sha256:1aaea3c439aac09b338d7b91dfa898c2ec653d3fff5e9b03293ba3e9998e1404
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/diegommcc/SpatialDDLS
  tools:
  - R
  - SingleCellExperiment
  - SpatialDDLS
  - SpatialDDLSdata
  - ggplot2
  - keras
  edam_operation: http://edamontology.org/operation_2422
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# aggregated-gradient-heatmap-visualization

## Summary

Interprets a trained SpatialDDLS neural network deconvolution model by computing per-gene gradients of predicted class scores (Vanilla Gradient, Simonyan et al. 2014), aggregating them to per-cell-type ranked gene lists, and visualizing the top aggregated gradients as heatmaps (e.g., Cd8/Cd8b1 for CD8 T cells, Foxp3 for Tregs).

## When to use

Use when a trained SpatialDDLS deconvolution model exists and the goal is model interpretation: ranking which genes drive predicted cell type proportions and visualizing top per-cell-type gradient markers as a heatmap.

## When NOT to use

- The SpatialDDLS model has not yet been trained (no trainDeconvModel output) — gradients require a fitted neural network over simulated mixed profiles
- The task is deconvolution itself (predicting cell type proportions from spatial data via deconvSpatialDDLS with spatial regularization) rather than interpreting an already-trained model

## Inputs

- Trained SpatialDDLS object containing the fully-connected neural network deconvolution model (e.g., mouseDLN.SDDLS from MouseDLN.SCE scRNA-seq data and MouseDLN.ST spatial data)
- Simulated mixed transcriptional profiles with known cell composition generated from scRNA-seq data (SingleCellExperiment input)

## Outputs

- Per-cell-type ranked gradient gene table (top_gradients_per_cell_type.tsv) with 'Positive' gradient entries, top 5 genes per cell type
- Heatmap of top mean aggregated gradients per cell type (top_grad_heatmap.png, 'Absolute' scaling, top 4 genes)

## How to apply

Load the trained SpatialDDLS object (e.g., mouseDLN.SDDLS) together with its simulated mixed transcriptional profiles and compute gradients of predicted class scores with respect to input genes via interGradientsDL(object, method = 'class', scaling = 'standardize'); gradients are first computed at the sample/gene level and then aggregated to the cell type level. Extract per-cell-type ranked genes with topGradientsCellType(object, method = 'class', top.n.genes = 5), keeping the 'Positive' gradient entries (genes whose gradients increase the predicted proportion for that cell type), and export the table, e.g., as top_gradients_per_cell_type.tsv. Render the heatmap of the top aggregated gradients with plotHeatmapGradsAgg(object, top.n.genes = 4, method = 'class'), saving the 'Absolute' variant, e.g., as top_grad_heatmap.png. Validate the interpretation by checking that canonical markers from the vignette (Cd8/Cd8b1 for CD8 T cells, Foxp3 for Tregs) appear among the top-5 positive-gradient genes of the matching cell types.

## Related tools

- **SpatialDDLS** (Provides the interpretation functions interGradientsDL, topGradientsCellType, and plotHeatmapGradsAgg that compute, rank, and plot per-cell-type gradients from the trained model) — https://github.com/diegommcc/SpatialDDLS
- **keras** (TensorFlow/Keras backend dependency on which the neural network gradients are computed) — https://cran.r-project.org/package=keras
- **ggplot2** (Plotting library underlying the aggregated gradient heatmap rendering)

## Examples

```
interGradientsDL(mouseDLN.SDDLS, method = "class", scaling = "standardize"); topGradientsCellType(mouseDLN.SDDLS, method = "class", top.n.genes = 5); plotHeatmapGradsAgg(mouseDLN.SDDLS, top.n.genes = 4, method = "class")
```

## Evaluation signals

- Canonical marker Cd8 (Cd8b1) appears among the top-5 positive-gradient genes for CD8 T cells in the reported output table
- Canonical marker Foxp3 appears among the top-5 positive-gradient genes for Tregs
- topGradientsCellType with top.n.genes = 5 returns only 'Positive' gradient entries per cell type, exported as top_gradients_per_cell_type.tsv
- plotHeatmapGradsAgg(top.n.genes = 4, method = 'class') produces an 'Absolute' aggregated gradient heatmap saved as top_grad_heatmap.png
- Gradients were computed with interGradientsDL using method = 'class' and scaling = 'standardize', aggregated from sample/gene level to cell type level

## Limitations

- The gradient interpretation is an optional module based on the Vanilla Gradient method (Simonyan et al. 2014); it complements but does not replace other model-evaluation metrics such as MAE/MSE from calculateEvalMetrics
- Interpretation reflects gradients over simulated mixed transcriptional profiles with known cell composition, not the raw spatial spots themselves
- Requires a working Python/Tensorflow interpreter (e.g., the spatialddls-env conda environment installed via installTFpython) for the keras-backed model

## Evidence

- [supplementary] gradient module: [evidence span withheld — anchor status: not_found]
- [methods] interGradientsDL call: [evidence span withheld — anchor status: not_found]
- [methods] marker colocalization rationale: "markers for each cell type colocalize with the cell proportions predicted by `SpatialDDLS`"
- [readme] package documentation: "A detailed manual of the main functionalities implemented in the package can be found at https://diegommcc.github.io/SpatialDDLS"
