---
name: scrna-seq-cell-label-annotation
description: Use when you have a small selected marker-gene panel (e.g., a 15-marker
  panel) plus per-cell expression data and cell type labels, and you need to verify
  the panel preserves 13-way (or similar) cell type separation in a 2-D t-SNE embedding.
when_to_use_negative:
- The goal is marker selection itself rather than visual validation of an already-selected
  panel — use scGeneFit's optimization directly instead
- The task requires placing unlabeled cells along trajectories or in spatial assays,
  which the method's categorical labeling does not support
- Cell type classes are not linearly separable in the reduced marker space, an assumption
  the method acknowledges may fail in low dimensions
edam_operation: http://edamontology.org/operation_0337
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scGeneFit
  role: Source of the selected marker-gene panel and release data files used to subset
    the expression matrix before t-SNE embedding
  repo: github.com/solevillar/scGeneFit
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-21453-4
    title: Optimal marker gene selection for cell type discrimination in single cell
      analyses
  source_content_hash: sha256:dbe57d95f6aa78a6f1f3f8cd673b0df68fba1b33890b9ddb101e286fd05dd3ce
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - scGeneFit
  edam_operation: http://edamontology.org/operation_0337
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# scrna-seq-cell-label-annotation

## Summary

Visualize single-cell RNA-seq cell type labels in the reduced space of a small selected marker panel (e.g., scGeneFit's 15 markers for the CBMC dataset) by running t-SNE on the marker-gene projection and rendering a label-colored scatter plot. This checks whether a compact marker panel preserves the distinctions between cell type populations.

## When to use

Use when you have a small selected marker-gene panel (e.g., a 15-marker panel) plus per-cell expression data and cell type labels, and you need to verify the panel preserves 13-way (or similar) cell type separation in a 2-D t-SNE embedding.

## When NOT to use

- The goal is marker selection itself rather than visual validation of an already-selected panel — use scGeneFit's optimization directly instead
- The task requires placing unlabeled cells along trajectories or in spatial assays, which the method's categorical labeling does not support
- Cell type classes are not linearly separable in the reduced marker space, an assumption the method acknowledges may fail in low dimensions

## Inputs

- Single-cell RNA-seq expression matrix (cells x genes) for the CBMC cord blood mononuclear cell dataset
- Selected marker-gene panel (15 marker genes from scGeneFit)
- Per-cell cell type label annotations (13 distinct cell populations)

## Outputs

- Two-dimensional t-SNE embedding of cells in the 15-marker projection
- Scatter plot colored by cell type label saved as tsne_marker_projection.png (or .pdf)

## How to apply

Load the single-cell expression dataset and the selected marker panel shipped with the scGeneFit release (github.com/solevillar/scGeneFit). Subset the full expression matrix to the selected marker-gene columns (15 markers for the CBMC cord blood mononuclear cell dataset) to form a low-dimensional projection of each cell, and append the cellular label annotations so each projected cell retains its identity. Run t-SNE on the 15-dimensional marker projection to embed the cells in two dimensions, then render a scatter plot colored by cell type label and save as tsne_marker_projection.png (or .pdf). Rationale: scGeneFit jointly optimizes cell label recovery with label-aware compressive classification, so the reduced marker space should retain cluster structure if the panel is adequate.

## Related tools

- **scGeneFit** (Source of the selected marker-gene panel and release data files used to subset the expression matrix before t-SNE embedding) — github.com/solevillar/scGeneFit

## Examples

```
from sklearn.manifold import TSNE; emb = TSNE(n_components=2).fit_transform(X[:, marker_idx]); plt.scatter(emb[:,0], emb[:,1], c=labels); plt.savefig('tsne_marker_projection.png')
```

## Evaluation signals

- Output figure file tsne_marker_projection.png (or .pdf) exists and is a valid PNG/PDF
- The scatter plot visually separates cells into distinct clusters corresponding to the 13 cell type populations
- The clustering pattern in the t-SNE embedding is consistent with the published t-SNE visualization of the CBMC dataset
- The input matrix used for t-SNE contains exactly the 15 marker-gene columns, confirming correct subsetting of the full expression matrix

## Limitations

- scGeneFit assumes cell type classes are linearly separable, which is reasonable in high dimensions but may not hold in the reduced marker space
- No nonlinear dimension reduction methods are implemented in scGeneFit itself (t-SNE is applied externally); nonlinear methods are planned future work
- Marker selection is limited to categorical labeling and does not place unlabeled cells on trajectories or in spatial assays

## Evidence

- [abstract] Given single cell RNA-seq data and a set of cellular labels to discriminate, scGeneFit selects gene markers that jointly optimize cell label recovery: "Given single cell RNA-seq data and a set of cellular labels to discriminate, scGeneFit selects gene markers that jointly optimize cell label recovery"
- [results] we applied scGeneFit to a cord blood mononuclear cell (CBMC) study containing isolated cells from cord blood: "we applied scGeneFit to a cord blood mononuclear cell (CBMC) study containing isolated cells from cord blood"
- [discussion] A key underlying assumption common to both scGeneFit and existing one-vs-all methods is that cell type classes are linearly separable: "A key underlying assumption common to both scGeneFit and existing one-vs-all methods is that cell type classes are linearly separable, which is a reasonable assumption in high dimensions, but may not"
