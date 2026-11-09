---
name: gene-marker-selection
description: Use when you have a scRNA-seq expression matrix with categorical cell
  labels (flat or hierarchical) and must design a minimal gene panel that discriminates
  all labels jointly with high precision.
when_to_use_negative:
- The goal is placing unlabeled cells on trajectories or in spatial assays — scGeneFit
  only supports categorical labeling, not manifold constraints.
- Cell type classes are not linearly separable in the reduced space — both scGeneFit
  and one-vs-all methods assume linear separability, reasonable in high dimensions
  but possibly violated in low dimensions.
- A non-linear dimension reduction or feature table unrelated to label discrimination
  is needed — scGeneFit is built on linear label-aware compressive classification.
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3518
tools:
- name: scGeneFit
  role: 'Primary marker selection engine: performs label-aware joint marker optimization
    given expression data and the label hierarchy to discriminate'
  repo: https://github.com/solevillar/scGeneFit
- name: COMET
  role: Baseline comparison method that exhaustively selects sets of k markers ranked
    in a one-vs-all task
provenance:
  source_task_ids:
  - task_003
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-21453-4
    title: Optimal marker gene selection for cell type discrimination in single cell
      analyses
  source_content_hash: sha256:c9013a41f6b7c81821a0c245159d1f8b7ce0cf1558264febb35129aed201f846
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/solevillar/scGeneFit
  tools:
  - COMET
  - scGeneFit
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# gene-marker-selection

## Summary

Select a small, non-redundant set of gene markers that jointly optimize recovery of cell type labels from scRNA-seq data, using scGeneFit's label-aware compressive classification rather than per-gene one-vs-all testing. Validated on CBMC CITE-seq data (8.05% vs 9.90% curated vs 30% random classification error) and synthetic 40-class simulations.

## When to use

Use when you have a scRNA-seq expression matrix with categorical cell labels (flat or hierarchical) and must design a minimal gene panel that discriminates all labels jointly with high precision.

## When NOT to use

- The goal is placing unlabeled cells on trajectories or in spatial assays — scGeneFit only supports categorical labeling, not manifold constraints.
- Cell type classes are not linearly separable in the reduced space — both scGeneFit and one-vs-all methods assume linear separability, reasonable in high dimensions but possibly violated in low dimensions.
- A non-linear dimension reduction or feature table unrelated to label discrimination is needed — scGeneFit is built on linear label-aware compressive classification.

## Inputs

- scRNA-seq or CITE-seq expression matrix (e.g. CBMC CITE-seq data from GEO accession GSE100866) with a log2(1+X) normalization
- Vector of categorical cell type labels per cell (flat partition or hierarchical partition of labels)
- Target marker panel size (number of markers to select)
- Synthetic multimodal gene expression data (e.g. 1000 cells, 10,000 genes, 15 functional groups, 40 classes; multivariate normal or generalized multivariate normal with Poisson link)

## Outputs

- Jointly optimized marker gene set for the target panel size
- Per-label classification error and f1 scores for each marker set (aggregated into one CSV row per label, e.g. marker_selection_comparison.csv)
- Held-out label classification error per marker set (e.g. 8.05% scGeneFit, 9.90% curated, 30% random on CBMC)

## How to apply

Normalize the expression matrix with a log2(1+X) transform, then run scGeneFit's joint marker selection against the provided label partition (e.g. the 40 synthetic cell labels or CBMC cell types) to recover an optimal set of genes of a given size that robustly partitions labels. Assemble baselines: a one-vs-all selection that scores each gene separately per label (optionally via COMET, which exhaustively selects k-marker sets ranked one-vs-all), a curated panel, and a size-matched random panel. Split cells 70% training / 30% test, train a k-nearest-neighbor classifier on the training split in the reduced marker space for each marker set, and evaluate label classification error on the held-out 30%. The rationale is that joint selection exploits the latent dimension of gene expression profiles, so fewer than k−1 markers are needed when latent dimension is below the number of labels, yielding a more robust and less redundant set than one-vs-all approaches.

## Related tools

- **scGeneFit** (Primary marker selection engine: performs label-aware joint marker optimization given expression data and the label hierarchy to discriminate) — https://github.com/solevillar/scGeneFit
- **COMET** (Baseline comparison method that exhaustively selects sets of k markers ranked in a one-vs-all task)

## Examples

```
from scGeneFit import fit; markers = fit(X=log2(1+expr), labels=cell_types, num_markers=k); print(markers)
```

## Evaluation signals

- On held-out data, the scGeneFit error must be lowest among marker sets (order: scGeneFit < curated < random; 8.05% < 9.90% < 30% on CBMC).
- Per-label CSV must contain one error/f1 row per label, and scGeneFit must achieve lower error (or higher f1) than one-vs-all on the majority of labels (verified across all 40 synthetic labels).
- With a target marker set size of ≥2 on the spectral toy model, the two selected features must be the two encoding circle coordinates, enabling sample-label recovery.
- Selected markers should be non-redundant: fewer markers than labels suffice when latent dimension of cell-type expression profiles is below the number of labels.
- The improvement of scGeneFit over one-vs-all must persist and scale with the number of markers in the larger synthetic simulation (15 functional groups, 40 classes).

## Limitations

- Assumes cell type classes are linearly separable; reasonable in high dimensions but may not hold in lower dimensions.
- No nonlinear dimension reduction methods implemented; relaxing the linear separability assumption is planned future work.
- Restricted to categorical labels — cannot place unlabeled cells on trajectories or spatial locations (manifold-constrained selection is envisioned future work).

## Evidence

- [abstract] joint label-aware marker selection: "Given single cell RNA-seq data and a set of cellular labels to discriminate, scGeneFit selects gene markers that jointly optimize cell label recovery"
- [abstract] robustness over one-vs-all: "This results in a substantially more robust and less redundant set of markers than existing methods, most of which identify markers that separate each cell label from the rest."
- [abstract] hierarchical labels efficiency: "the markers found by our method improves the recovery of the cell type hierarchy with fewer markers than existing methods using a computationally efficient and principled optimization"
- [intro] latent dimension rationale: "these markers will be redundant if the latent dimension of the space spanned by the gene expression profiles of each cell type is lower than the number of cell labels we aim to distinguish"
- [intro] COMET baseline: "One such method, COMET10, exhaustively select sets of k markers that it then ranks in a one-vs-all task."
- [results] toy-model marker recovery: "scGeneFit selected as markers the two features representing the circle coordinates whenever the size of the target marker set was ≥2"
- [results] CBMC benchmark dataset: "we applied scGeneFit to a cord blood mononuclear cell (CBMC) study containing isolated cells from cord blood"
- [discussion] linear separability assumption: "A key underlying assumption common to both scGeneFit and existing one-vs-all methods is that cell type classes are linearly separable, which is a reasonable assumption in high dimensions, but may not"
- [discussion] manifold future work: "we envision relaxing the categorical labeling to a manifold constraint that will allow selection of markers to place unlabeled cells at, for example, time points along cell trajectories or locations"
