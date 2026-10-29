---
name: label-aware-compressive-classification-configuration
description: Use when you have a cells x genes scRNA-seq expression matrix plus cell-type
  labels (e.g., 40 classes) and need to select a specific number of markers (e.g.,
  50) that jointly discriminate the labels, including on large synthetic panels (10,000
  genes).
when_to_use_negative:
- The cell-type classes are not linearly separable or require nonlinear manifold structure
  — the method assumes linear separability and implements no nonlinear dimension reduction
  (planned future work)
- Labels are not categorical, e.g., you need markers placing unlabeled cells along
  trajectories or spatial locations — categorical labeling is a current limitation
  (manifold constraint is envisioned, not implemented)
- You need to place unlabeled cells on trajectories or in spatial assays rather than
  discriminate discrete cell labels
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scGeneFit
  role: 'Primary solver: instantiates the label-aware compressive classification optimization
    over the expression matrix and label vector to return the jointly selected marker
    set of the requested size'
  repo: github.com/solevillar/scGeneFit
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-21453-4
    title: Optimal marker gene selection for cell type discrimination in single cell
      analyses
  source_content_hash: sha256:43777117ee3b443d55dc92d5cf8296cd3f3fe898513be10a706d8c9dd97ace1a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - scGeneFit
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# label-aware-compressive-classification-configuration

## Summary

Configure scGeneFit to jointly select a fixed-size marker gene panel from scRNA-seq data using label-aware compressive classification, optimizing recovery of cellular labels (including hierarchical cell-type partitions) rather than testing genes one-vs-all.

## When to use

Use when you have a cells x genes scRNA-seq expression matrix plus cell-type labels (e.g., 40 classes) and need to select a specific number of markers (e.g., 50) that jointly discriminate the labels, including on large synthetic panels (10,000 genes).

## When NOT to use

- The cell-type classes are not linearly separable or require nonlinear manifold structure — the method assumes linear separability and implements no nonlinear dimension reduction (planned future work)
- Labels are not categorical, e.g., you need markers placing unlabeled cells along trajectories or spatial locations — categorical labeling is a current limitation (manifold constraint is envisioned, not implemented)
- You need to place unlabeled cells on trajectories or in spatial assays rather than discriminate discrete cell labels

## Inputs

- scRNA-seq expression matrix (cells x genes), e.g., 10,000 cells x 10,000 genes from a simulated panel with 15 functional groups
- Cell-type label vector (e.g., 40 cell-class annotations per cell)
- Target marker set size k (e.g., 50 markers)
- Optional: hierarchical partition of cell-type labels

## Outputs

- Indices of selected marker genes (exactly k, e.g., 50) from the fitted scGeneFit solver
- JSON or equivalent record of measured wall-clock runtime (seconds) and number of selected markers
- Downstream classification accuracy as a function of marker count (under multivariate normal or Poisson-link generalized MVN evaluation)

## How to apply

Load the scRNA-seq expression matrix (cells x genes) and the corresponding cell-label vector, then instantiate the scGeneFit solver on these two inputs with the target marker set size (e.g., 50 markers). Execute the label-aware compressive classification optimization, which jointly recovers the optimal gene set of the given size for robust partitioning of labels instead of scoring each gene separately; wrap the fit call with a wall-clock timer when reproducing scalability runs. Extract the selected marker gene indices from the fitted solver and confirm the panel size matches the requested number exactly. Rationale: joint selection yields a more robust, less redundant panel than one-vs-all methods, and the method scales to ~15 minutes on laptop-class hardware for a 10,000-cell x 10,000-gene dataset with 40 classes; it can be validated under a multivariate normal model or a generalized multivariate normal model with a Poisson link.

## Related tools

- **scGeneFit** (Primary solver: instantiates the label-aware compressive classification optimization over the expression matrix and label vector to return the jointly selected marker set of the requested size) — github.com/solevillar/scGeneFit

## Evaluation signals

- The fitted solver returns exactly the requested number of markers (e.g., 50 indices), with no duplicates
- Wall-clock fit runtime is on the order of ~15 minutes for the 10,000-cell x 10,000-gene, 40-class configuration on laptop-class hardware
- Classification accuracy improves substantially as the allowed number of markers increases, under both the multivariate normal model and the generalized MVN model with a Poisson link
- The selected panel is less redundant and more robust than one-vs-all marker sets on the same labels (the paper's stated advantage of joint selection)

## Limitations

- A key assumption is that cell type classes are linearly separable — reasonable in high dimensions but may not hold in lower dimensions
- No nonlinear dimension reduction methods are implemented; relaxing the linear separability assumption is planned future work
- Marker selection is limited to categorical labeling; no support for placing unlabeled cells on trajectories or spatial locations (envisioned future manifold constraint)
- Fewer markers than k-1 can be redundant when the latent dimension of cell-type expression profiles is lower than the number of cell labels

## Evidence

- [abstract] Given single cell RNA-seq data and a set of cellular labels to discriminate, scGeneFit selects gene markers that jointly optimize cell label recovery: "Given single cell RNA-seq data and a set of cellular labels to discriminate, scGeneFit selects gene markers that jointly optimize cell label recovery"
- [abstract] This results in a substantially more robust and less redundant set of markers than existing methods: "This results in a substantially more robust and less redundant set of markers than existing methods"
- [intro] we developed scGeneFit, a rigorous and efficient approach for marker selection in the context of scRNA-seq data with a given hierarchical partition of labels: "we developed scGeneFit, a rigorous and efficient approach for marker selection in the context of scRNA-seq data with a given hierarchical partition of labels"
- [results] scGeneFit selected as markers the two features representing the circle coordinates whenever the size of the target marker set was ≥2: "scGeneFit selected as markers the two features representing the circle coordinates whenever the size of the target marker set was ≥2"
- [discussion] A key underlying assumption common to both scGeneFit and existing one-vs-all methods is that cell type classes are linearly separable: "A key underlying assumption common to both scGeneFit and existing one-vs-all methods is that cell type classes are linearly separable"
