---
name: pseudotime-tree-visualization
description: Use when a multi-sample scRNA-seq pseudotime tree has just been inferred
  (e.g. via infer_tree_structure with origin.marker and number.cluster) and you need
  to inspect or publish the trajectory's branch structure and cell ordering.
when_to_use_negative:
- The trajectory was inferred with Monocle2, Monocle3, or Slingshot — Lamian modules
  1 and 2 do not support these trajectories, so plotmclust cannot visualize them
- The goal is to quantify branch uncertainty or test differential topology rather
  than inspect/depict the inferred tree; use evaluate_uncertainty or branchPropTest
  instead
edam_operation: http://edamontology.org/operation_0567
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2229
tools:
- name: Lamian
  role: Provides plotmclust for rendering the inferred TSCAN cluster-based MST trajectory
    and infer_tree_structure that produces the trajectory object to visualize
  repo: https://github.com/Winnie09/Lamian
- name: TSCAN
  role: Default method Lamian uses to construct the pseudotime tree (cluster-based
    minimum spanning tree) that is then visualized with plotmclust
  repo: https://github.com/zji90/TSCAN
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-42841-y
    title: A statistical framework for differential pseudotime analysis with multiple
      single-cell RNA-seq samples
  source_content_hash: sha256:1042517c41c4c47e9dccc494a36858397347ec714f18dece3a9cf27c42ac9aa6
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Winnie09/Lamian
  tools:
  - Lamian
  - TSCAN
  - branchPropTest
  - infer_tree_structure
  - plotmclust
  edam_operation: http://edamontology.org/operation_0567
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# pseudotime-tree-visualization

## Summary

Visualize a cluster-based minimum spanning tree (TSCAN-style) pseudotime trajectory inferred by Lamian's infer_tree_structure, rendering cells in PCA space with branch structure overlaid using plotmclust. This step documents the inferred tree topology (e.g. the three-branch HCA-BM tree 2 --> 3 --> 1, 2 --> 4, 2 --> 5) prior to uncertainty quantification and differential topology testing.

## When to use

Use when a multi-sample scRNA-seq pseudotime tree has just been inferred (e.g. via infer_tree_structure with origin.marker and number.cluster) and you need to inspect or publish the trajectory's branch structure and cell ordering.

## When NOT to use

- The trajectory was inferred with Monocle2, Monocle3, or Slingshot — Lamian modules 1 and 2 do not support these trajectories, so plotmclust cannot visualize them
- The goal is to quantify branch uncertainty or test differential topology rather than inspect/depict the inferred tree; use evaluate_uncertainty or branchPropTest instead

## Inputs

- Trajectory object returned by Lamian's infer_tree_structure (cluster-based MST with branch definitions)
- PCA embedding of cells (e.g. hca_bm_pca from the Lamian package example data)
- Gene-by-cell expression matrix (e.g. hca_bm_saver)
- Cell-to-sample annotation (e.g. hca_bm_cellanno)

## Outputs

- Tree structure figure (e.g. tree_structure.png) showing cells in PCA space with the inferred branch layout
- Visually verified pseudotemporal tree topology (branch connectivity, e.g. 2 --> 3 --> 1, 2 --> 4, 2 --> 5)

## How to apply

After constructing the trajectory with Lamian's infer_tree_structure on the gene-by-cell expression matrix, PCA embedding, and cell-to-sample annotation (e.g. number.cluster = 5, origin.marker = 'CD34' for the 8-donor HCA bone marrow data), pass the resulting trajectory object to Lamian::plotmclust to draw cells and the inferred branch structure in the low-dimensional space. Set cell_point_size = 0.5 for dense scRNA-seq point clouds and label the axes Principal component 1/2 when plotting PCA embeddings. Inspect the rendered tree to confirm the expected topology (origin branch, lineage branches) matches the cluster MST before running evaluate_uncertainty or branchPropTest, since downstream differential topology tests assume the inferred tree is biologically sensible. Save the figure (e.g. tree_structure.png) as a reusable artifact documenting the trajectory used in the analysis.

## Related tools

- **Lamian** (Provides plotmclust for rendering the inferred TSCAN cluster-based MST trajectory and infer_tree_structure that produces the trajectory object to visualize) — https://github.com/Winnie09/Lamian
- **TSCAN** (Default method Lamian uses to construct the pseudotime tree (cluster-based minimum spanning tree) that is then visualized with plotmclust) — https://github.com/zji90/TSCAN

## Examples

```
res = infer_tree_structure(pca = man_tree_data[['pca']], ...); Lamian::plotmclust(res, cell_point_size = 0.5)
```

## Evaluation signals

- The rendered figure displays cells in Principal component 1/2 space with the expected branch connectivity (e.g. 2 --> 3 --> 1, 2 --> 4, 2 --> 5 for the HCA-BM data with number.cluster = 5)
- The visualization derives from the same trajectory object that is subsequently passed to evaluate_uncertainty, so branch labels in the plot match branches in the detection-rate output (~0.938, 0.953, 0.615 for HCA-BM)
- Plot settings match the documented call (cell_point_size = 0.5) and the figure exports successfully (e.g. tree_structure.png) without Cairo/X11 errors
- Origin orientation is consistent with the origin.marker used during inference (CD34+ origin at the tree root)

## Limitations

- Lamian modules 1 and 2 do not support Monocle2, Monocle3, or Slingshot trajectories, so this visualization is limited to TSCAN-style cluster-based MST trees
- The Cairo package (and X11/XQuartz on mac) is required for visualization; if ComplexHeatmap installation fails it is removed from Imports and heatmap plotting does not work
- Systematic comparison of harmonization methods used to build the low-dimensional embedding is beyond the scope of the study

## Evidence

- [other] Visualize the inferred tree with plotmclust (cell_point_size = 0.5, axes labeled Principal component 1/2) and save as tree_structure.png.: [evidence span withheld — anchor status: not_found]
- [discussion] In Lamian, TSCAN is used as the default method to construct pseudotime due to its flexibility and scalability.: "In Lamian, TSCAN is used as the default method to construct pseudotime due to its flexibility and scalability."
- [methods] Lamian::plotmclust(res, cell_point_size = 0.5,: "Lamian::plotmclust(res, cell_point_size = 0.5,"
- [readme] For visualization, please download X11 or XQuartz (mac users: https://www.xquartz.org/), and then download the package Cairo: [evidence span withheld — anchor status: external_doc]
