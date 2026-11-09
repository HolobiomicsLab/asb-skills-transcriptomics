---
name: gene-scoring-per-cell-type-from-deconvolution-model
description: Use when a trained SpatialDDLS deconvolution model exists and you need
  to know WHICH genes the network uses to predict each cell type's proportions, e.g.
  to recover canonical markers (Cd8/Cd8b1, Foxp3) from predicted spot compositions.
when_to_use_negative:
- You only need predicted cell type proportions per spot — use deconvSpatialDDLS directly
  without the interpretation module.
- No trained SpatialDDLS model exists yet; gradients require a fitted fully-connected
  neural network, so first run genMixedCellProp, simMixedProfiles and trainDeconvModel.
- The goal is marker discovery from raw scRNA-seq (e.g. cluster logFC markers) rather
  than interpreting an existing model's decisions — the gradients reflect what the
  network learned from the simulated mixtures, not independent differential expression.
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0091
tools:
- name: SpatialDDLS
  role: Provides interGradientsDL, topGradientsCellType and plotHeatmapGradsAgg for
    gradient-based neural network interpretation on the trained deconvolution model
  repo: https://CRAN.R-project.org/package=SpatialDDLS
- name: keras
  role: TensorFlow/Keras backend on which the fully-connected neural network whose
    class-score gradients are computed is trained
  repo: https://cran.r-project.org/package=keras
- name: ggplot2
  role: Rendering backend for plotHeatmapGradsAgg heatmap visualization of aggregated
    gradients
- name: SingleCellExperiment
  role: Container for the scRNA-seq reference (e.g. MouseDLN.SCE) loaded into the
    SpatialDDLS object before gradient interpretation
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae072
    title: 'SpatialDDLS: an R package to deconvolute spatial transcriptomics data
      using neural networks'
  source_content_hash: sha256:fc768137f279c518acc9113e3f8821ee47b56467e1e1bde01ea692963e8e6f7d
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
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# gene-scoring-per-cell-type-from-deconvolution-model

## Summary

Ranks genes driving a trained SpatialDDLS neural network's cell type proportion predictions via gradient-based interpretation (Vanilla Gradient, Simonyan et al. 2014), yielding per-cell-type gene importance scores and heatmaps for downstream marker analyses.

## When to use

Use when a trained SpatialDDLS deconvolution model exists and you need to know WHICH genes the network uses to predict each cell type's proportions, e.g. to recover canonical markers (Cd8/Cd8b1, Foxp3) from predicted spot compositions.

## When NOT to use

- You only need predicted cell type proportions per spot — use deconvSpatialDDLS directly without the interpretation module.
- No trained SpatialDDLS model exists yet; gradients require a fitted fully-connected neural network, so first run genMixedCellProp, simMixedProfiles and trainDeconvModel.
- The goal is marker discovery from raw scRNA-seq (e.g. cluster logFC markers) rather than interpreting an existing model's decisions — the gradients reflect what the network learned from the simulated mixtures, not independent differential expression.

## Inputs

- Trained SpatialDDLS object (deconvolution model fitted with trainDeconvModel on simulated mixed transcriptional profiles)
- scRNA-seq reference as SingleCellExperiment (e.g. MouseDLN.SCE from 10x Chromium, GEO GSE173778) used to build the model
- Optional: spatial transcriptomics data as SpatialExperiment (e.g. MouseDLN.ST from 10x Visium) if interpreting a deconvolved dataset

## Outputs

- Per-cell-type top-N gradient gene table (top_gradients_per_cell_type.tsv) with 'Positive' gradient entries
- Heatmap of top 4 mean aggregated gradients per cell type ('Absolute' scaling; top_grad_heatmap.png)
- Gradient values stored in the SpatialDDLS object accessible via interGradientsDL aggregation

## How to apply

Start from a fitted SpatialDDLS object trained on simulated mixed transcriptional profiles built from scRNA-seq (e.g. MouseDLN.SCE with the 10x Chromium data). Compute gradients of predicted class scores with respect to input genes using interGradientsDL(object, method = "class", scaling = "standardize") — the Vanilla Gradient method, aggregated from sample/gene level to cell type level. Extract top-ranked genes per cell type with topGradientsCellType(object, method = "class", top.n.genes = 5), keeping the 'Positive' gradient entries, and export as top_gradients_per_cell_type.tsv. Visualize the top 4 mean aggregated gradients per cell type with plotHeatmapGradsAgg(object, top.n.genes = 4, method = "class"), saving the 'Absolute' heatmap as top_grad_heatmap.png (Supplementary Fig. S10a–d style output). The rationale is that gradient magnitude of the class score w.r.t. each gene reflects the gene's contribution to the network's decision for that cell type, so gradient ranking substitutes for a direct marker list.

## Related tools

- **SpatialDDLS** (Provides interGradientsDL, topGradientsCellType and plotHeatmapGradsAgg for gradient-based neural network interpretation on the trained deconvolution model) — https://CRAN.R-project.org/package=SpatialDDLS
- **keras** (TensorFlow/Keras backend on which the fully-connected neural network whose class-score gradients are computed is trained) — https://cran.r-project.org/package=keras
- **ggplot2** (Rendering backend for plotHeatmapGradsAgg heatmap visualization of aggregated gradients)
- **SingleCellExperiment** (Container for the scRNA-seq reference (e.g. MouseDLN.SCE) loaded into the SpatialDDLS object before gradient interpretation)

## Examples

```
interGradientsDL(mouseDLN.SDDLS, method = "class", scaling = "standardize"); topGradientsCellType(mouseDLN.SDDLS, method = "class", top.n.genes = 5)
```

## Evaluation signals

- Canonical markers reported in the vignette — Cd8/Cd8b1 for CD8 T cells and Foxp3 for Tregs — appear among the top-5 positive-gradient genes of their respective cell types.
- top_gradients_per_cell_type.tsv contains one ranked gene list per cell type with only 'Positive' gradient entries retained, matching the vignette's reported output table.
- The heatmap from plotHeatmapGradsAgg(top.n.genes = 4, method = "class") shows distinct top genes per cell type, consistent with Supplementary Fig. S10a–d.
- interGradientsDL was run with method = "class" and scaling = "standardize"; gradients are aggregated from sample/gene level to cell type level before ranking.
- Rankings reproduce across reruns on the same trained model (gradient computation on a fixed network is deterministic given the same inputs).

## Limitations

- Gradient attributions are model-dependent: they reflect what the fully-connected network learned from simulated mixed transcriptional profiles, so they inherit biases from the scRNA-seq reference and simulation parameters (e.g. sc.n.genes.per.cluster, sc.min.mean.counts, min.zero.prop filters).
- The module is optional and adds computation on top of deconvolution; it uses the Vanilla Gradient method (Simonyan et al. 2014), which is a first-order saliency approximation and may attribute to correlated genes ambiguously.
- Interpretation requires a working TensorFlow/Keras Python environment (e.g. via installTFpython / spatialddls-env); missing Python setup will block interGradientsDL.

## Evidence

- [supplementary] gradient module supports downstream marker recovery: [evidence span withheld — anchor status: not_found]
- [abstract] package purpose grounding the trained model used for gradients: "we introduce SpatialDDLS, a fast neural network-based algorithm for cell type deconvolution of spatial transcriptomics data"
- [methods] training context for the model being interpreted: "we can then train a neural network using the training subset and evaluate the model by predicting cell type proportions on the test subset"
- [readme] tool availability and docs: "The **SpatialDDLS** R package provides a neural network-based solution for cell type deconvolution of spatial transcriptomics data. The package takes advantage of single-cell RNA sequencing"
- [methods] marker-validation rationale for gradient rankings: "markers for each cell type colocalize with the cell proportions predicted by `SpatialDDLS`"
