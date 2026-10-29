---
name: contour-overlap-dispute-resolution
description: Use when stabilizing raw hierarchical cell-type predictions on a scRNA-seq
  dataset where overlapping label regions in UMAP space indicate disputed cells, and
  an iterative contour-dispute loop is needed to produce converged annotations.
when_to_use_negative:
- Input cells lack a UMAP dimensionality reduction or SNN clustering — the contours
  and cluster propagation cannot be computed
- Predictions are already stabilized/converged (no overlapping contours or label changes)
  — iterating further is unnecessary
- You need raw model predictions rather than cluster- and contour-smoothed annotations
edam_operation: http://edamontology.org/operation_0222
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: Census
  role: 'Provides the label-stabilizing algorithm: computes prediction contours, detects
    overlaps, adjudicates disputed cells, and iterates with SNN label propagation
    to convergence'
  repo: https://github.com/sjdlabgroup/Census
- name: Seurat 4
  role: Builds the SNN graph and UMAP embedding whose clusters supply the majority-label
    propagation step; must be loaded before the Census library
  repo: https://github.com/sjdlabgroup/Census
- name: MASS
  role: Used to compute prediction contours in UMAP space for each candidate label
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad714
    title: Hierarchical and automated cell-type annotation and inference of cancer
      cell of origin with Census
  source_content_hash: sha256:4be0ffaa698b9f3f6021581d6223f8d4023fc2e80c4a62233fa9d33065f9c998
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sjdlabgroup/Census
  tools:
  - Census
  - MASS
  - Seurat 4
  edam_operation: http://edamontology.org/operation_0222
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# contour-overlap-dispute-resolution

## Summary

Resolves per-cell label conflicts by fitting MASS-based prediction contours for each candidate label in UMAP space, adjudicating cells falling in overlapping contours, and re-propagating the majority label within UMAP SNN clusters until convergence. This is Census's core label-stabilizing step that counteracts dataset noise and prediction error before descending to the next hierarchy node.

## When to use

Use when stabilizing raw hierarchical cell-type predictions on a scRNA-seq dataset where overlapping label regions in UMAP space indicate disputed cells, and an iterative contour-dispute loop is needed to produce converged annotations.

## When NOT to use

- Input cells lack a UMAP dimensionality reduction or SNN clustering — the contours and cluster propagation cannot be computed
- Predictions are already stabilized/converged (no overlapping contours or label changes) — iterating further is unnecessary
- You need raw model predictions rather than cluster- and contour-smoothed annotations

## Inputs

- Seurat object (v4) containing a UMAP dimensionality reduction and per-cell root-node predictions from the pretrained Census model
- UMAP SNN cluster assignments from the Seurat shared nearest-neighbor graph
- Per-cell candidate label predictions at the current hierarchy node

## Outputs

- Converged per-cell stabilized cell-type annotations (stabilized_annotations.csv)
- Resolved label assignments free of overlapping prediction contours, ready for the next node in the hierarchy

## How to apply

After binary predictions are propagated as average labels within UMAP SNN clusters (built with Seurat), compute prediction contours in UMAP space for each candidate label using MASS kernel densities. Detect contours that overlap and adjudicate the disputed cells lying in overlap regions between the competing label contours. After resolving contour disputes, re-propagate the most common label across each UMAP SNN cluster and recompute the prediction contours, iterating this propagate-contour-dispute loop until there are no overlapping contours or no further label changes between iterations. The converged per-cell stabilized annotations are then written to stabilized_annotations.csv and used before proceeding to the next node in the cell-type hierarchy. Note that Census requires Seurat version 4, which must be loaded prior to loading the Census library in R.

## Related tools

- **Census** (Provides the label-stabilizing algorithm: computes prediction contours, detects overlaps, adjudicates disputed cells, and iterates with SNN label propagation to convergence) — https://github.com/sjdlabgroup/Census
- **Seurat 4** (Builds the SNN graph and UMAP embedding whose clusters supply the majority-label propagation step; must be loaded before the Census library) — https://github.com/sjdlabgroup/Census
- **MASS** (Used to compute prediction contours in UMAP space for each candidate label)

## Examples

```
res = census_main(obj, organ = 'Lung')
```

## Evaluation signals

- Convergence check: no overlapping contours remain or no further label changes occur between propagate-contour-dispute iterations
- Each cell carries exactly one stabilized label consistent with its UMAP SNN cluster's propagated average label
- Stabilized annotations are emitted as stabilized_annotations.csv with per-cell stabilized assignments
- No cells remain inside two different label contours after the loop terminates
- Seurat v4 is confirmed loaded before library(Census) to avoid version incompatibility

## Limitations

- Stabilization is performed in UMAP space, so results depend on the Seurat UMAP/SNN construction parameters and pipeline upstream of census_main
- Census requires Seurat version 4 specifically; loading other Seurat versions prior to the Census library will break the run
- The loop is iterative and may require multiple propagate-contour-dispute rounds before convergence

## Evidence

- [intro] New datasets are annotated using the pretrained models followed by a custom-developed label-stabilizing algorithm (Fig. 1b). This involves using clusters and prediction contours in UMAP space: "New datasets are annotated using the pretrained models followed by a custom-developed label-stabilizing algorithm (Fig. 1b). This involves using clusters and prediction contours in UMAP space"
- [intro] Label-stabilizing step propagates higher confidence binary predictions: "using clusters and prediction contours in UMAP space to propagate higher confidence binary prediction outcomes"
- [readme] Census requires Seurat version 4: "Census requires Seurat version 4 to run. Please load Seurat v4 prior to loading the Census library in R."
