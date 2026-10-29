---
name: single-cell-annotation-stabilization
description: Use when a Seurat scRNA-seq object has root-node cell-type predictions
  from a Census pretrained model and noisy per-cell labels need to be stabilized using
  UMAP clusters and prediction contours before continuing hierarchical annotation.
when_to_use_negative:
- The Seurat object lacks a UMAP dimensionality reduction — the algorithm operates
  on clusters and prediction contours in UMAP space, so it cannot be applied
- The R environment runs Seurat v5; Census requires Seurat version 4 to run, so stabilization
  logic tied to it will not work as documented
- The tissue is outside the 24 supported organs of the main Census model (e.g. no
  trained cancer model for that organ) unless a custom model has been trained
tools:
- name: Census
  role: Provides the pretrained gradient-boosted decision tree models and the custom-developed
    label-stabilizing algorithm that propagates labels and resolves contour overlaps
  repo: https://github.com/sjdlabgroup/Census
- name: Seurat 4
  role: Builds the Seurat object, SNN graph, and UMAP embedding on which cluster-based
    label propagation operates; must be loaded (v4) before the Census library
- name: MASS
  role: Computes the prediction contours in UMAP space for each candidate label used
    in overlap detection and dispute adjudication
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad714
    title: Hierarchical and automated cell-type annotation and inference of cancer
      cell of origin with Census
  source_content_hash: sha256:6c80ae68b8db8899ee3c47d447b164d46263b9920b09bdbfb7e4a9aac0c95f61
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sjdlabgroup/Census
  tools:
  - Census
  - MASS
  - Seurat 4
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# single-cell-annotation-stabilization

## Summary

Census's label-stabilizing algorithm smooths initial gradient-boosted-tree cell-type predictions by propagating average labels across UMAP SNN clusters and adjudicating overlapping MASS prediction contours in UMAP space, iterating until convergence. It counteracts per-cell prediction noise before deeper nodes of the cell-type hierarchy are classified.

## When to use

Use when a Seurat scRNA-seq object has root-node cell-type predictions from a Census pretrained model and noisy per-cell labels need to be stabilized using UMAP clusters and prediction contours before continuing hierarchical annotation.

## When NOT to use

- The Seurat object lacks a UMAP dimensionality reduction — the algorithm operates on clusters and prediction contours in UMAP space, so it cannot be applied
- The R environment runs Seurat v5; Census requires Seurat version 4 to run, so stabilization logic tied to it will not work as documented
- The tissue is outside the 24 supported organs of the main Census model (e.g. no trained cancer model for that organ) unless a custom model has been trained

## Inputs

- Seurat v4 object containing a UMAP dimensionality reduction
- Root-node cell-type predictions from the pretrained Census model (Tabula Sapiens + CCLE, 175 cell-types from 24 organs)
- Organ specification (e.g. 'Lung', 'Pancreas') matching one of the 24 supported organs

## Outputs

- Stabilized per-cell cell-type annotations (res$pred$celltype from census_main)
- For cancer tissues with predict_cancer=T: malignant/normal calls plus predicted cell of origin (cancer_origin)

## How to apply

Load Seurat v4 before the Census library (Census requires Seurat version 4), and ensure the Seurat object contains a UMAP dimensionality reduction produced by the standard Seurat pipeline (NormalizeData, ScaleData, FindVariableFeatures, RunPCA, RunUMAP). Construct the shared nearest-neighbor graph and UMAP embedding, then propagate the most common/average label within each UMAP SNN cluster to spread higher-confidence binary prediction outcomes from the pretrained Tabula Sapiens model (175 cell-types, 24 organs). Compute MASS-based prediction contours in UMAP space for each candidate label, detect overlapping contours, and resolve disputed cells by adjudicating between the overlapping label contours. Re-run average-label propagation on the resolved assignments and iterate the propagate-contour-dispute loop until convergence — no overlapping contours or no further label changes between iterations — then write the stabilized per-cell annotations.

## Related tools

- **Census** (Provides the pretrained gradient-boosted decision tree models and the custom-developed label-stabilizing algorithm that propagates labels and resolves contour overlaps) — https://github.com/sjdlabgroup/Census
- **Seurat 4** (Builds the Seurat object, SNN graph, and UMAP embedding on which cluster-based label propagation operates; must be loaded (v4) before the Census library)
- **MASS** (Computes the prediction contours in UMAP space for each candidate label used in overlap detection and dispute adjudication)

## Examples

```
obj = obj %>% NormalizeData() %>% ScaleData() %>% FindVariableFeatures() %>% RunPCA() %>% RunUMAP(dims=1:50); res = census_main(obj, organ = 'Lung')
```

## Evaluation signals

- Iterative loop converges: no overlapping prediction contours remain, or no further label changes occur between successive propagate-dispute iterations
- Stabilized labels align with UMAP SNN cluster structure — cells within a cluster carry a consistent (most-common) label
- Census predictions visually agree with original author annotations when plotted in UMAP space, as shown for the Travaglini et al. 2020 lung cell atlas (Fig. 3)
- Result object res$pred contains the expected columns (umap1, umap2, celltype; cancer_origin when predict_cancer=T) for downstream ggplot plotting

## Limitations

- Seurat version 4 is required; loading order matters (Seurat v4 prior to the Census library in R)
- If more than one organ is specified in the organ parameter, only the first one is used for the cancer model; available cancer models are limited to Kidney, Large_Intestine, Liver, Lung, Mammary, Pancreas
- Stabilization occurs at each node before proceeding deeper in the hierarchy; it smooths binary prediction outcomes but cannot recover from errors in the pretrained hierarchy itself

## Evidence

- [intro] Label-stabilizing algorithm rationale: "New datasets are annotated using the pretrained models followed by a custom-developed label-stabilizing algorithm (Fig. 1b). This involves using clusters and prediction contours in UMAP space"
- [intro] Propagation purpose: "using clusters and prediction contours in UMAP space to propagate higher confidence binary prediction outcomes"
- [intro] Hierarchical context of stabilization: "The Census model starts at the root node of the cell-type hierarchy and successively classifies cells according to their node identities until terminal classifications are reached"
- [readme] Seurat version dependency: "Census requires Seurat version 4 to run. Please load Seurat v4 prior to loading the Census library in R."
- [readme] Input data requirement: "The input data is a Seurat object that contains a UMAP dimensionality reduction. Users should also specify the tissue organ(s)"
