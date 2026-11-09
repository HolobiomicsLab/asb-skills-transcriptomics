---
name: canonical-lymphocyte-marker-validation
description: 'Use when a trained SpatialDDLS neural network deconvolution model must
  be interpreted or sanity-checked: after trainDeconvModel, to confirm predicted cell
  type proportions are driven by canonical markers via gradient-based ranked gene
  lists per cell type.'
when_to_use_negative:
- The deconvolution model has not yet been trained — gradients require the trained
  network; run trainDeconvModel first
- The goal is marker discovery from raw scRNA-seq without a trained deconvolution
  model — use differential expression/logFC-based marker selection (e.g. the sc.filt.genes.cluster
  pipeline) instead
- The task is deconvolution itself (predicting cell type proportions from spatial
  data via deconvSpatialDDLS) rather than interpreting which genes drive an already-trained
  model
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3518
tools:
- name: SpatialDDLS
  role: Provides the gradient-based neural network interpretation module (interGradientsDL,
    topGradientsCellType, plotHeatmapGradsAgg) applied to the trained deconvolution
    model
  repo: https://CRAN.R-project.org/package=SpatialDDLS
- name: keras
  role: Backend dependency providing the trained TensorFlow/Keras fully-connected
    network whose class-score gradients are computed
  repo: https://cran.r-project.org/package=keras
- name: ggplot2
  role: Plotting library underlying the heatmap visualization of aggregated gradients
    produced by plotHeatmapGradsAgg
- name: SingleCellExperiment
  role: Container format for the scRNA-seq reference (MouseDLN.SCE) whose cell type
    labels define the cell types interpreted
- name: SpatialDDLSdata
  role: Data package supplying the murine lymph node example (MouseDLN.SCE / MouseDLN.ST)
    used to reproduce the vignette marker table
  repo: https://github.com/diegommcc/SpatialDDLSdata
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae072
    title: 'SpatialDDLS: an R package to deconvolute spatial transcriptomics data
      using neural networks'
  source_content_hash: sha256:f709fc76192dbeae413c50ee066bb507918d856ed0b2c815d43381527ee07328
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - SingleCellExperiment
  - SpatialDDLS
  - SpatialDDLSdata
  - ggplot2
  - keras
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5384615384615384
n_claims: 13
---

# Canonical Lymphocyte Marker Validation

## Summary

Validates that a trained SpatialDDLS deconvolution model has learned biologically meaningful cell-type signatures by extracting per-cell-type gradient-based gene rankings from the neural network and confirming recovery of canonical lymphocyte markers such as Cd8/Cd8b1 for CD8 T cells and Foxp3 for regulatory T cells.

## When to use

Use when a trained SpatialDDLS neural network deconvolution model must be interpreted or sanity-checked: after trainDeconvModel, to confirm predicted cell type proportions are driven by canonical markers via gradient-based ranked gene lists per cell type.

## When NOT to use

- The deconvolution model has not yet been trained — gradients require the trained network; run trainDeconvModel first
- The goal is marker discovery from raw scRNA-seq without a trained deconvolution model — use differential expression/logFC-based marker selection (e.g. the sc.filt.genes.cluster pipeline) instead
- The task is deconvolution itself (predicting cell type proportions from spatial data via deconvSpatialDDLS) rather than interpreting which genes drive an already-trained model

## Inputs

- Trained SpatialDDLS object containing the trained fully-connected neural network deconvolution model (e.g. after trainDeconvModel on simulated mixed transcriptional profiles)
- Simulated mixed transcriptional profiles stored in the SpatialDDLS object (from genMixedCellProp and simMixedProfiles)
- Cell type labels from the scRNA-seq reference (e.g. MouseDLN.SCE SingleCellExperiment with lymph node cell types including CD8 T cells and Tregs)

## Outputs

- top_gradients_per_cell_type.tsv — per-cell-type table of top-5 positive-gradient genes
- top_grad_heatmap.png — heatmap of top 4 mean aggregated gradients per cell type ('Absolute' method)
- Per-cell-type ranked gene lists (Positive vs Absolute gradient entries) supporting downstream analyses

## How to apply

Load the trained SpatialDDLS deconvolution model together with the simulated mixed transcriptional profiles, then compute gradients of predicted class scores with respect to input genes using interGradientsDL(object, method = "class", scaling = "standardize"), which implements the Vanilla Gradient method (Simonyan et al. 2014) and aggregates sample/gene-level gradients to the cell-type level. Extract top-ranked genes per cell type with topGradientsCellType(object, method = "class", top.n.genes = 5), retaining the 'Positive' gradient entries (genes whose gradient positively supports the cell type's predicted score), and write the table to top_gradients_per_cell_type.tsv. Visualize the top 4 mean aggregated gradients per cell type using plotHeatmapGradsAgg(object, top.n.genes = 4, method = "class") and save the 'Absolute' heatmap as top_grad_heatmap.png. Validation consists of checking that canonical markers from the vignette output table — Cd8/Cd8b1 for CD8 T cells and Foxp3 for Tregs — appear among the top-5 positive-gradient genes of their respective cell types, mirroring Supplementary Fig. S10a–d. This also cross-checks the independent finding that known markers colocalize with SpatialDDLS-predicted cell proportions.

## Related tools

- **SpatialDDLS** (Provides the gradient-based neural network interpretation module (interGradientsDL, topGradientsCellType, plotHeatmapGradsAgg) applied to the trained deconvolution model) — https://CRAN.R-project.org/package=SpatialDDLS
- **keras** (Backend dependency providing the trained TensorFlow/Keras fully-connected network whose class-score gradients are computed) — https://cran.r-project.org/package=keras
- **ggplot2** (Plotting library underlying the heatmap visualization of aggregated gradients produced by plotHeatmapGradsAgg)
- **SingleCellExperiment** (Container format for the scRNA-seq reference (MouseDLN.SCE) whose cell type labels define the cell types interpreted)
- **SpatialDDLSdata** (Data package supplying the murine lymph node example (MouseDLN.SCE / MouseDLN.ST) used to reproduce the vignette marker table) — https://github.com/diegommcc/SpatialDDLSdata

## Examples

```
grads <- interGradientsDL(mouseDLN.SDDLS, method = "class", scaling = "standardize"); topGradientsCellType(mouseDLN.SDDLS, method = "class", top.n.genes = 5); plotHeatmapGradsAgg(mouseDLN.SDDLS, top.n.genes = 4, method = "class")
```

## Evaluation signals

- Cd8 or Cd8b1 appears among the top-5 positive-gradient genes for the CD8 T cell type, matching the reported vignette output table
- Foxp3 appears among the top-5 positive-gradient genes for the Tregs cell type
- top_gradients_per_cell_type.tsv exists and contains one row block per cell type with 'Positive' gradient entries, each with a gene symbol and gradient value
- The 'Absolute' heatmap (top_grad_heatmap.png) renders top 4 genes per cell type and their relative aggregated gradient magnitudes, consistent with Supplementary Fig. S10a–d
- interGradientsDL was run with method = "class" and scaling = "standardize" so gradients are aggregated from sample/gene level to cell type level consistently across cell types

## Limitations

- The interpretation module is optional and provides insights rather than causal proof — gradient rankings (Simonyan et al. 2014 Vanilla Gradient) are a heuristic read-out of the trained model's decision-making process
- Validation is anchored to the murine lymph node vignette (Cd8, Cd8b1, Foxp3); other tissues require user-defined canonical markers and absence of a marker among top gradients does not necessarily mean the cell type was not learned
- Gradient aggregation from sample/gene level to cell type level means rankings reflect the trained model's simulated profiles, not the spatial data itself; genes filtered out during preprocessing (e.g. the 1041 genes retained after logFC filtering, mitochondrial genes removed) cannot be recovered

## Evidence

- [supplementary] gradient-based NN interpretation: [evidence span withheld — anchor status: not_found]
- [methods] marker colocalization: "markers for each cell type colocalize with the cell proportions predicted by `SpatialDDLS`"
- [intro] training workflow context: "simulate mixed transcriptional profiles with known cell composition and train fully-connected neural networks to predict cell type composition of spatial transcriptomics spots"
- [readme] package purpose: "The **SpatialDDLS** R package provides a neural network-based solution for cell type deconvolution of spatial transcriptomics data."
- [methods] filtering thresholds: "=== Number of genes after filtering based on logFC: 1041"
