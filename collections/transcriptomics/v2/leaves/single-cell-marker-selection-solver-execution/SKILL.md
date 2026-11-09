---
name: single-cell-marker-selection-solver-execution
description: Use when given a simulated or real scRNA-seq expression matrix with cell-type
  labels and a target marker panel size (e.g., 50 markers), and you must run scGeneFit
  to extract selected gene indices and measure solver runtime.
when_to_use_negative:
- The input is already a small feature/marker table or the labels are unknown — scGeneFit
  requires given cellular labels to discriminate
- You need nonlinear dimension reduction or manifold/trajectory placement of unlabeled
  cells — the article states nonlinear methods were not implemented and categorical
  (not manifold-constrained) labeling is required
- Cell-type classes are not linearly separable in the reduced space — the article
  notes linear separability is a key assumption, reasonable in high dimensions but
  possibly violated in lower dimensions
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scGeneFit
  role: 'Primary solver: jointly selects the marker set via label-aware compressive
    classification on the expression matrix and label vector; used to time the fit
    and extract selected gene indices'
  repo: github.com/solevillar/scGeneFit
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-21453-4
    title: Optimal marker gene selection for cell type discrimination in single cell
      analyses
  source_content_hash: sha256:031711dcca78a29fced647eaa2153eeab96ba7965eba6a537258235c4f4eb0fb
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - scGeneFit
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# single-cell-marker-selection-solver-execution

## Summary

Executes the scGeneFit solver to jointly select an optimal marker-gene panel from a labeled scRNA-seq expression matrix using label-aware compressive classification, supporting scalability runs (e.g., 50 markers on 10,000 genes x 10,000 cells) and panel-size sweeps.

## When to use

Use when given a simulated or real scRNA-seq expression matrix with cell-type labels and a target marker panel size (e.g., 50 markers), and you must run scGeneFit to extract selected gene indices and measure solver runtime.

## When NOT to use

- The input is already a small feature/marker table or the labels are unknown — scGeneFit requires given cellular labels to discriminate
- You need nonlinear dimension reduction or manifold/trajectory placement of unlabeled cells — the article states nonlinear methods were not implemented and categorical (not manifold-constrained) labeling is required
- Cell-type classes are not linearly separable in the reduced space — the article notes linear separability is a key assumption, reasonable in high dimensions but possibly violated in lower dimensions

## Inputs

- scRNA-seq expression matrix (cells x genes, e.g., 10,000 cells x 10,000 genes)
- Cell-type label vector / annotation (e.g., 40 cell classes from 15 functional groups)
- Target marker set size k (e.g., 50 markers)
- Optional hierarchical partition of cell-type labels

## Outputs

- Indices of the selected marker genes from the fitted solver (panel of exactly k genes)
- JSON record of measured wall-clock runtime in seconds and selected marker count (e.g., 50)
- Accuracy of label recovery as a function of number of allowed markers (under multivariate normal or Poisson-link generalized multivariate normal simulations)

## How to apply

Load the simulated or real scRNA-seq expression matrix (e.g., 10,000 cells x 10,000 genes) together with the cell-type label vector (e.g., 40 classes from 15 functional groups per the simulation spec shipped with the scGeneFit release). Instantiate the scGeneFit solver on the expression matrix and label vector with the target marker set size set explicitly (e.g., 50 markers), optionally supplying a hierarchical partition of labels. Execute the fit, timing it with a wall-clock timer, since the article reports runs on the order of ~15 minutes on laptop-class hardware at this scale. Extract the indices of the selected markers from the fitted solver and confirm the panel size matches the target. The rationale is that scGeneFit jointly recovers the optimal gene set for robust label partitioning rather than testing each gene separately, yielding less redundant panels than one-vs-all methods.

## Related tools

- **scGeneFit** (Primary solver: jointly selects the marker set via label-aware compressive classification on the expression matrix and label vector; used to time the fit and extract selected gene indices) — github.com/solevillar/scGeneFit

## Evaluation signals

- The fitted solver returns exactly the target number of markers (e.g., exactly 50 selected indices for k=50)
- Measured wall-clock runtime is on the order of ~15 minutes on laptop-class hardware for a 10,000 cells x 10,000 genes, 40-class run with 50 markers
- Classification accuracy on held-out labels improves monotonically (or near-monotonically) as the number of allowed markers increases, under both multivariate normal and Poisson-link generalized multivariate normal simulations
- The serialized JSON record contains the runtime in seconds and the marker count, and the selected panel is non-empty with valid gene indices
- Selected panels are less redundant than one-vs-all marker sets, consistent with the article's claim of a more robust, less redundant marker set

## Limitations

- Key assumption that cell type classes are linearly separable, which is reasonable in high dimensions but may fail in lower dimensions
- No nonlinear dimension reduction methods implemented; relaxing the linear separability assumption is planned future work
- Marker selection limited to categorical labels; no support for placing unlabeled cells on trajectories or spatial assays (manifold constraint envisioned as future work)
- Optimal marker selection given a hierarchical label partition is combinatorially difficult, so runtime grows with data scale (e.g., ~15 minutes at 10,000 genes x 10,000 cells)

## Evidence

- [abstract] Given single cell RNA-seq data and a set of cellular labels to discriminate, scGeneFit selects gene markers that jointly optimize cell label recovery: "Given single cell RNA-seq data and a set of cellular labels to discriminate, scGeneFit selects gene markers that jointly optimize cell label recovery"
- [intro] we developed scGeneFit, a rigorous and efficient approach for marker selection in the context of scRNA-seq data with a given hierarchical partition of labels: "we developed scGeneFit, a rigorous and efficient approach for marker selection in the context of scRNA-seq data with a given hierarchical partition of labels."
- [results] scGeneFit selected as markers the two features representing the circle coordinates whenever the size of the target marker set was ≥2: "scGeneFit selected as markers the two features representing the circle coordinates whenever the size of the target marker set was ≥2"
- [discussion] cell type classes are linearly separable, which is a reasonable assumption in high dimensions, but may not: "A key underlying assumption common to both scGeneFit and existing one-vs-all methods is that cell type classes are linearly separable, which is a reasonable assumption in high dimensions, but may not"
