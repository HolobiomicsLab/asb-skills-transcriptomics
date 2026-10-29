---
name: umap-cluster-label-propagation
description: Use when a Seurat object has root-node hierarchical predictions from
  a pretrained Census model (or analogous classifier) and per-cell labels need smoothing/denoising
  before proceeding to the next node in the cell-type hierarchy.
when_to_use_negative:
- Input lacks a UMAP dimensionality reduction — the Seurat object must be processed
  through the standard Seurat pipeline (NormalizeData, ScaleData, FindVariableFeatures,
  RunPCA, RunUMAP) first
- Cell-type labels are already stable terminal classifications with no overlapping
  prediction contours — no further stabilization is needed
- Seurat version is not v4 — Census requires Seurat version 4 to run
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: Census
  role: 'Implements the label-stabilizing algorithm: cluster-based label propagation,
    MASS prediction contours, contour-overlap dispute resolution, and iterative convergence'
  repo: https://github.com/sjdlabgroup/Census
- name: Seurat 4
  role: Constructs the SNN graph and UMAP embedding whose clusters define the propagation
    units; must be loaded (v4) before the Census library
  repo: https://github.com/sjdlabgroup/Census
- name: MASS
  role: Computes prediction contours in UMAP space for each candidate label
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad714
    title: Hierarchical and automated cell-type annotation and inference of cancer
      cell of origin with Census
  source_content_hash: sha256:1b835b45c2256f9df0c19d37551d504742352ea5a831f88494ee95cafdabd1a4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sjdlabgroup/Census
  tools:
  - Census
  - MASS
  - Seurat 4
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# umap-cluster-label-propagation

## Summary

Census's label-stabilizing algorithm that refines per-cell cell-type predictions by propagating average labels within UMAP SNN clusters and resolving overlapping MASS-based prediction contours, iterating until convergence to counteract dataset noise and prediction error.

## When to use

Use when a Seurat object has root-node hierarchical predictions from a pretrained Census model (or analogous classifier) and per-cell labels need smoothing/denoising before proceeding to the next node in the cell-type hierarchy.

## When NOT to use

- Input lacks a UMAP dimensionality reduction — the Seurat object must be processed through the standard Seurat pipeline (NormalizeData, ScaleData, FindVariableFeatures, RunPCA, RunUMAP) first
- Cell-type labels are already stable terminal classifications with no overlapping prediction contours — no further stabilization is needed
- Seurat version is not v4 — Census requires Seurat version 4 to run

## Inputs

- Seurat object with a UMAP dimensionality reduction and root-node Census predictions (cell-type hierarchy)
- Pretrained Census model (e.g., Tabula Sapiens + CCLE model covering 175 cell-types from 24 organs)

## Outputs

- Converged per-cell stabilized cell-type annotations (stabilized_annotations.csv)
- Final UMAP-space prediction contours per candidate label

## How to apply

Load the Seurat object containing root-node predictions, ensuring Seurat v4 is loaded before the Census library. Construct the shared nearest-neighbor graph and UMAP embedding with Seurat, then propagate the most common (average) label within each UMAP SNN cluster to spread higher-confidence binary prediction outcomes. Compute MASS-based prediction contours in UMAP space for each candidate label, detect overlapping contours, and adjudicate disputed cells between the overlapping label contours. Re-run the average-label propagation on the resolved assignments and iterate this propagate-contour-dispute loop until convergence — defined as no overlapping contours or no further label changes between iterations. The rationale is that spatially contiguous cells in UMAP space should share labels, so cluster-level consensus plus contour-based dispute resolution stabilizes noisy individual predictions.

## Related tools

- **Census** (Implements the label-stabilizing algorithm: cluster-based label propagation, MASS prediction contours, contour-overlap dispute resolution, and iterative convergence) — https://github.com/sjdlabgroup/Census
- **Seurat 4** (Constructs the SNN graph and UMAP embedding whose clusters define the propagation units; must be loaded (v4) before the Census library) — https://github.com/sjdlabgroup/Census
- **MASS** (Computes prediction contours in UMAP space for each candidate label)

## Examples

```
res = census_main(obj, organ = 'Lung')
```

## Evaluation signals

- Iteration terminates with no overlapping prediction contours between labels, or no further label changes between successive propagate-contour iterations
- Stabilized labels show spatial coherence: cells within a UMAP SNN cluster carry the propagated majority label
- Disputed cells from overlapping contours are all adjudicated to exactly one of the overlapping label contours
- Output is a complete per-cell annotation table (stabilized_annotations.csv) with one stabilized label per cell matching the input cell count
- Stabilized annotations can be compared against the original per-cell predictions to confirm noisy singleton labels were absorbed into cluster consensus

## Limitations

- Requires Seurat v4 specifically; loading Seurat v5 or later before the Census library will break execution
- Stabilization only runs after the pretrained model produces root-node predictions; it cannot be applied to raw expression data alone
- Convergence is not guaranteed to eliminate all overlaps — the loop also stops when no further label changes occur, so some contour overlap may persist

## Evidence

- [intro] Label-stabilizing step using clusters and prediction contours in UMAP space to propagate higher confidence binary prediction outcomes: "Label-stabilizing step using clusters and prediction contours in UMAP space to propagate higher confidence binary prediction outcomes"
- [readme] New datasets are annotated using the pretrained models followed by a custom-developed label-stabilizing algorithm: "New datasets are annotated using the pretrained models followed by a custom-developed label-stabilizing algorithm (Fig. 1b). This involves using clusters and prediction contours in UMAP space"
- [readme] Census requires Seurat version 4 to run: "Census requires Seurat version 4 to run. Please load Seurat v4 prior to loading the Census library in R."
