---
name: gradient-interpretation-class-vs-loss-methods
description: Use when a trained SpatialDDLS deconvolution model must be made interpretable
  at the gene level — e.g., to recover which input genes (such as Cd8/Cd8b1 or Foxp3)
  drive the predicted cell type proportions in spatial transcriptomics spots.
when_to_use_negative:
- The deconvolution model has not yet been trained — gradient interpretation requires
  a trained keras-backed fully-connected network inside the SpatialDDLS object
- The goal is to validate predicted cell type proportions against marker expression
  in space (e.g., checking colocalization of markers with predicted proportions) rather
  than to extract gene-level model interpretability
tools:
- name: SpatialDDLS
  role: Provides interGradientsDL, topGradientsCellType, and plotHeatmapGradsAgg for
    gradient-based neural network interpretation of the trained deconvolution model
  repo: https://github.com/diegommcc/SpatialDDLS
- name: keras
  role: TensorFlow/Keras backend on which the trained fully-connected network runs;
    gradients of class scores/loss with respect to input genes are computed through
    it
  repo: https://cran.r-project.org/package=keras
- name: ggplot2
  role: Graphics backend used to render the aggregated gradient heatmaps
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae072
    title: 'SpatialDDLS: an R package to deconvolute spatial transcriptomics data
      using neural networks'
  source_content_hash: sha256:20aff4814ce2d0e8b79e0e61406484de8ec9e1a7a3dbf55e057e5485df260037
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6875
n_claims: 16
---

# gradient-interpretation-class-vs-loss-methods

## Summary

Extracts per-cell-type ranked gene lists from a trained SpatialDDLS neural network using gradient-based interpretation (Vanilla Gradient, Simonyan et al. 2014), computed with respect to either predicted class scores ('class') or the loss ('loss'), to identify which genes drive predicted cell type proportions.

## When to use

Use when a trained SpatialDDLS deconvolution model must be made interpretable at the gene level — e.g., to recover which input genes (such as Cd8/Cd8b1 or Foxp3) drive the predicted cell type proportions in spatial transcriptomics spots.

## When NOT to use

- The deconvolution model has not yet been trained — gradient interpretation requires a trained keras-backed fully-connected network inside the SpatialDDLS object
- The goal is to validate predicted cell type proportions against marker expression in space (e.g., checking colocalization of markers with predicted proportions) rather than to extract gene-level model interpretability

## Inputs

- Trained SpatialDDLS object containing the fully-connected neural network deconvolution model and simulated mixed transcriptional profiles (e.g., from trainDeconvModel on simulated profiles from MouseDLN.SCE scRNA-seq data)
- Choice of interpretation method ('class' or 'loss') and scaling ('standardize')

## Outputs

- Per-cell-type ranked gradient gene table (e.g., top_gradients_per_cell_type.tsv) with 'Positive' gradient entries for top.n.genes per cell type
- Heatmap of aggregated gradients per cell type (e.g., top_grad_heatmap.png), 'Absolute' variant from plotHeatmapGradsAgg

## How to apply

Load the trained SpatialDDLS model with its simulated mixed transcriptional profiles and compute gradients of the predicted class scores with respect to input genes using interGradientsDL(object, method = "class", scaling = "standardize"); the Vanilla Gradient method aggregates gradients from the sample/gene level up to the cell type level. Choose method = "class" when you want genes explaining the predicted class scores, or method = "loss" when gradients with respect to the training loss are preferred for interpretation. Extract top-ranked genes per cell type with topGradientsCellType(object, method = "class", top.n.genes = 5), keeping the 'Positive' gradient entries for genes that push predictions toward each cell type, and export the table (e.g., top_gradients_per_cell_type.tsv). Visualize the aggregated gradients with plotHeatmapGradsAgg(object, top.n.genes = 4, method = "class"), saving the 'Absolute' heatmap (e.g., top_grad_heatmap.png). Validate by confirming that canonical markers reported in the vignette — Cd8/Cd8b1 for CD8 T cells and Foxp3 for Tregs — appear among the top-5 positive-gradient genes of their respective cell types, as in Supplementary Fig. S10a–d.

## Related tools

- **SpatialDDLS** (Provides interGradientsDL, topGradientsCellType, and plotHeatmapGradsAgg for gradient-based neural network interpretation of the trained deconvolution model) — https://github.com/diegommcc/SpatialDDLS
- **keras** (TensorFlow/Keras backend on which the trained fully-connected network runs; gradients of class scores/loss with respect to input genes are computed through it) — https://cran.r-project.org/package=keras
- **ggplot2** (Graphics backend used to render the aggregated gradient heatmaps)

## Examples

```
interGradientsDL(mouseDLN.SDDLS, method = "class", scaling = "standardize"); topGradientsCellType(mouseDLN.SDDLS, method = "class", top.n.genes = 5); plotHeatmapGradsAgg(mouseDLN.SDDLS, top.n.genes = 4, method = "class")
```

## Evaluation signals

- Canonical markers reported in the vignette (Cd8/Cd8b1 for CD8 T cells, Foxp3 for Tregs) appear among the top-5 'Positive' gradient genes of their respective cell types, as shown in the reported output table
- topGradientsCellType returns both 'Positive' and 'Negative' gradient columns, and requested top.n.genes entries are populated per cell type
- The heatmap produced by plotHeatmapGradsAgg shows the requested top.n.genes (e.g., 4) mean aggregated gradients per cell type and matches the interpretation shown in Supplementary Fig. S10a–d
- Gradients are aggregated from sample/gene level to cell type level, so the output table has one ranked gene list per cell type in the trained model

## Limitations

- Interpretation is a post-hoc Vanilla Gradient method (Simonyan et al. 2014); it gives insights into the model's decision-making but is an optional module, not a guarantee of biological marker recovery
- Requires a working Python interpreter with TensorFlow/Keras installed (e.g., via installTFpython creating the spatialddls-env conda environment); interpretation fails without this environment
- Gradient signal depends on the genes retained during upstream filtering (e.g., the 1041 genes after logFC filtering and 10941 shared genes between single-cell and spatial datasets in the vignette); markers removed upstream cannot be recovered by gradients

## Evidence

- [supplementary] Gradient module exists and supports downstream analyses: [evidence span withheld — anchor status: not_found]
- [methods] Predicted proportions align with known markers: "markers for each cell type colocalize with the cell proportions predicted by `SpatialDDLS`"
- [intro] Trained networks predict cell type composition of spatial spots: "simulate mixed transcriptional profiles with known cell composition and train fully-connected neural networks to predict"
- [readme] Keras/TensorFlow dependency needed to run the trained model and gradients: [evidence span withheld — anchor status: not_found]
