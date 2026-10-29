---
name: single-cell-expression-subsetting
description: Use when a marker-gene panel (e.g. scGeneFit-selected genes) has been
  computed and you need to reduce full scRNA-seq expression data to just those gene
  columns before downstream visualization such as t-SNE.
when_to_use_negative:
- You need marker selection itself, not projection — the panel has not yet been computed
  (run scGeneFit's selection step first)
- Cell type classes may not be linearly separable in the reduced space — the article
  notes the method assumes linear separability, reasonable in high dimensions but
  potentially not in lower dimensions
- You need to place unlabeled cells on trajectories or spatial locations — the method
  supports only categorical labeling, not manifold constraints
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scGeneFit
  role: Provides the selected marker-gene panel (shipped in its release data files)
    that defines which expression-matrix columns to retain during subsetting
  repo: github.com/solevillar/scGeneFit
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-21453-4
    title: Optimal marker gene selection for cell type discrimination in single cell
      analyses
  source_content_hash: sha256:d4ad5d524ee356e641171865d4bf17fc4a54a15a3c53e8e66a651e80a3d489d0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - scGeneFit
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# single-cell-expression-subsetting

## Summary

Subset a single-cell RNA-seq expression matrix to a small selected marker-gene panel (e.g. scGeneFit's 15-marker CBMC panel) to form a low-dimensional projection of each cell that retains cell type discrimination.

## When to use

Use when a marker-gene panel (e.g. scGeneFit-selected genes) has been computed and you need to reduce full scRNA-seq expression data to just those gene columns before downstream visualization such as t-SNE.

## When NOT to use

- You need marker selection itself, not projection — the panel has not yet been computed (run scGeneFit's selection step first)
- Cell type classes may not be linearly separable in the reduced space — the article notes the method assumes linear separability, reasonable in high dimensions but potentially not in lower dimensions
- You need to place unlabeled cells on trajectories or spatial locations — the method supports only categorical labeling, not manifold constraints

## Inputs

- Single-cell RNA-seq expression matrix (cells x genes), e.g. from the CBMC cord blood mononuclear cell study
- Selected marker-gene list (e.g. scGeneFit's 15-marker panel from release data files)
- Cell type label annotations per cell (e.g. 13 CBMC populations)

## Outputs

- Marker-subset expression matrix (cells x 15 marker-gene columns)
- 2-D t-SNE embedding colored by cell type, saved as tsne_marker_projection.png (or .pdf)

## How to apply

Load the single-cell expression dataset and the marker panel shipped with the tool's release data files (github.com/solevillar/scGeneFit), then subset the full expression matrix to only the selected marker-gene columns, forming a low-dimensional projection of each cell. Append the cellular label annotations so each projected cell retains its cell type identity (e.g. the 13 distinct CBMC cell populations). This reduced marker space is the correct input for downstream visualization: run t-SNE on the 15-dimensional marker projection to embed cells in 2-D and render a scatter plot colored by cell type label. Rationale: scGeneFit jointly optimizes cell label recovery with as few markers as possible, so subsetting to the panel preserves the distinctions between cell labels while dropping thousands of non-informative genes.

## Related tools

- **scGeneFit** (Provides the selected marker-gene panel (shipped in its release data files) that defines which expression-matrix columns to retain during subsetting) — github.com/solevillar/scGeneFit

## Examples

```
X_markers = X[:, marker_idx]; Y = TSNE(n_components=2).fit_transform(X_markers); plt.scatter(Y[:,0], Y[:,1], c=labels); plt.savefig('tsne_marker_projection.png')
```

## Evaluation signals

- Output figure file exists and is a valid PNG/PDF
- Scatter plot visually separates the 13 CBMC cell populations into distinct clusters consistent with the published t-SNE visualization
- Subset matrix has exactly the panel's number of columns (e.g. 15) and the same number of rows (cells) as the original matrix
- Every retained column maps to a gene in the selected marker panel, and cell labels remain aligned row-for-row after subsetting

## Limitations

- Assumes linear separability of cell type classes, reasonable in high dimensions but may not hold in lower dimensions
- No nonlinear dimension reduction methods are implemented in the selection; nonlinear visualization (t-SNE) is applied only after subsetting
- Limited to categorical labeling — no support for placing unlabeled cells on trajectories or in spatial assays

## Evidence

- [abstract] scGeneFit selects gene markers that jointly optimize cell label recovery using label-aware compressive classification methods.: "Given single cell RNA-seq data and a set of cellular labels to discriminate, scGeneFit selects gene markers that jointly optimize cell label recovery"
- [results] Applied to the CBMC dataset for marker subsetting and t-SNE projection.: "we applied scGeneFit to a cord blood mononuclear cell (CBMC) study containing isolated cells from cord blood"
- [discussion] Why reduced-dimension projections may not separate classes as well as high dimensions.: "cell type classes are linearly separable, which is a reasonable assumption in high dimensions, but may not"
