---
name: lmds-dimensionality-reduction-for-trajectory-input
description: Use when preparing a single-cell dataset for tree-shaped trajectory inference
  (e.g., Totem CLARA k-medoids clustering) and a fast low-dimensional embedding is
  needed instead of PCA or full MDS on large datasets.
when_to_use_negative:
- The dataset must be visualized in an embedding with 3 or more dimensions — the article
  notes 3-dimensional embeddings are not supported for plotting at the moment
- The downstream analysis involves trajectories with cycles or disconnected components,
  which Totem (operating on this embedding) cannot generate
- The dataset has extremely rare and extremely prevalent cell types simultaneously
  — k-medoids on this embedding may fail and require manual cluster merging
tools:
- name: dyndimred
  role: Provides the landmark Multi-Dimensional Scaling (LMDS) implementation used
    to generate the 5-dimensional low-dimensional embedding
- name: Totem
  role: Consumes the LMDS embedding for cell-connectivity-guided tree trajectory inference;
    PrepareTotem is applied before dimensionality reduction
  repo: https://github.com/elolab/Totem
- name: SingleCellExperiment
  role: Container object holding the expression data and the computed reduced-dimension
    embedding
- name: R
  role: Runtime environment in which dyndimred, Totem, and SingleCellExperiment are
    used
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad515
    title: Cell-connectivity-guided trajectory inference from single-cell data
  source_content_hash: sha256:5ffd04a31233d8f707c7bea8f9e51cb2da267931d3b08f7e149e03c8810fdccd
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/elolab/Totem
  tools:
  - BiocManager
  - R
  - SingleCellExperiment
  - Slingshot
  - TinGa
  - Totem
  - dyndimred
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# lmds-dimensionality-reduction-for-trajectory-input

## Summary

Generate a low-dimensional embedding of single-cell data with landmark Multi-Dimensional Scaling (LMDS) from the dyndimred package (5 dimensions) so that downstream trajectory inference tools such as Totem, Slingshot, and TinGa all consume a common, fast-to-compute input space.

## When to use

Use when preparing a single-cell dataset for tree-shaped trajectory inference (e.g., Totem CLARA k-medoids clustering) and a fast low-dimensional embedding is needed instead of PCA or full MDS on large datasets.

## When NOT to use

- The dataset must be visualized in an embedding with 3 or more dimensions — the article notes 3-dimensional embeddings are not supported for plotting at the moment
- The downstream analysis involves trajectories with cycles or disconnected components, which Totem (operating on this embedding) cannot generate
- The dataset has extremely rare and extremely prevalent cell types simultaneously — k-medoids on this embedding may fail and require manual cluster merging

## Inputs

- Prepared single-cell expression object (SingleCellExperiment prepared with PrepareTotem)
- Number of target embedding dimensions (5 in the article's workflow; 2 for visualization)

## Outputs

- 5-dimensional LMDS low-dimensional embedding used as common input for trajectory inference
- 2-dimensional MDS embedding for visualization of cells and connectivity

## How to apply

First create a SingleCellExperiment object and prepare it with PrepareTotem; then apply LMDS from the dyndimred package to produce a low-dimensional embedding with 5 dimensions, which serves as the standardized input for all trajectory inference methods. LMDS is chosen because it is faster than full MDS for large datasets, and the article warns that k-medoids (CLARA) can behave badly on high-dimensional PCA input (e.g., 30 principal components), producing 1–2 cell clusters. Generate a separate two-dimensional embedding (MDS) only for visualization, since 3-dimensional embeddings are not supported for plotting at the moment. Keep the 5-dimensional LMDS embedding identical across all methods being compared so scores (geodesic distance correlation, featureimp correlation, F1 branches, HIM) reflect method differences rather than input differences.

## Related tools

- **dyndimred** (Provides the landmark Multi-Dimensional Scaling (LMDS) implementation used to generate the 5-dimensional low-dimensional embedding)
- **Totem** (Consumes the LMDS embedding for cell-connectivity-guided tree trajectory inference; PrepareTotem is applied before dimensionality reduction) — https://github.com/elolab/Totem
- **SingleCellExperiment** (Container object holding the expression data and the computed reduced-dimension embedding)
- **R** (Runtime environment in which dyndimred, Totem, and SingleCellExperiment are used)

## Examples

```
emb <- dyndimred::dimred(sce_counts, method = "lmds", ndim = 5)
```

## Evaluation signals

- The resulting embedding has exactly the requested number of dimensions (5 for trajectory input, 2 for visualization)
- Downstream CLARA k-medoids clustering does not collapse into 1–2 cell clusters (the failure mode observed with high-dimensional PCA input such as 30 principal components)
- The same 5-dimensional embedding is reused for every method compared (Totem, Slingshot, TinGa), so benchmark scores reflect method differences, not embedding differences
- Runtime scales acceptably on large datasets (LMDS is chosen because it is faster than MDS for large datasets)

## Limitations

- k-medoids may not work well with PCA if the number of principal components is high (e.g., 30); LMDS with a modest dimension count is used to avoid this
- The 2D embedding used for visualization is limited to two dimensions — 3-dimensional embeddings are not supported at the moment

## Evidence

- [intro] LMDS embedding specification and tool: "Here, we use the landmark Multi-Dimensional Scaling (LMDS) from `dyndimred` package [4] to generate a low-dimensional embedding with 5 dimensions."
- [intro] Rationale for LMDS over MDS: "LMDS is faster than MDS for large datasets."
- [discussion] Why PCA input is avoided for k-medoids: "the k-medoids algorithm may not work well with PCA if the number principal components is high (e.g. 30)"
- [intro] Visualization embedding constraint: "To visualize the data,
we need a two-dimensional embedding (3-dimensional not supported at the moment)."
