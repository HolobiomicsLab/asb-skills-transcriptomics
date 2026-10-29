---
name: visium-spatial-expression-visualization
description: Use when Visium spatial transcriptomics data with spot-level counts and
  2D coordinates is available and you need to find and visualize gene sets/pathways
  whose module-score activity is non-randomly distributed across tissue spots.
when_to_use_negative:
- Comparisons between multiple conditions (e.g., wild-type and knockout) are needed
  — the article notes these cannot easily be conducted with singleCellHaystack
- Cluster-oriented DEG analysis is the goal — results are not straightforward to incorporate
  into cluster-based scRNA-seq analyses
- Speed is the priority for very large spatial datasets — SPARK-X is faster for spatial
  transcriptomics despite lower sensitivity
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3308
tools:
- name: singleCellHaystack
  role: Predicts pathways with non-random spatial distributions from module scores
    using KLD-based testing on 2D spot coordinates; provides hclust_haystack clustering
    of top pathways
  repo: https://github.com/alexisvdb/singleCellHaystack
- name: Seurat
  role: Loads Visium spot-level counts and computes per-spot pathway activity via
    AddModuleScore
- name: msigdbr
  role: Supplies the 292 BioCarta gene sets used for pathway scoring
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-023-38965-2
    title: A universal tool for predicting differentially active features in single-cell
      and spatial genomics data
  source_content_hash: sha256:e7a2c4b87da6047550a2ca0be0f42fef19dccff5f8e39020dfa59ec7e3c573c9
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/alexisvdb/singleCellHaystack
  tools:
  - R
  - Seurat
  - msigdbr
  - singleCellHaystack
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# visium-spatial-expression-visualization

## Summary

Visualizes the spatial distribution of gene-set (pathway) activity across Visium spots by scoring pathways per spot with Seurat AddModuleScore, ranking them with singleCellHaystack KLD-based p-values, and plotting top pathways on 2D tissue coordinates (as in Fig. 5A-C of the singleCellHaystack spatial tutorial).

## When to use

Use when Visium spatial transcriptomics data with spot-level counts and 2D coordinates is available and you need to find and visualize gene sets/pathways whose module-score activity is non-randomly distributed across tissue spots.

## When NOT to use

- Comparisons between multiple conditions (e.g., wild-type and knockout) are needed — the article notes these cannot easily be conducted with singleCellHaystack
- Cluster-oriented DEG analysis is the goal — results are not straightforward to incorporate into cluster-based scRNA-seq analyses
- Speed is the priority for very large spatial datasets — SPARK-X is faster for spatial transcriptomics despite lower sensitivity

## Inputs

- Visium spot-level gene expression count matrix (mouse anterior brain, posterior brain, kidney datasets from the singleCellHaystack spatial tutorial)
- 2D spatial coordinates (x, y) per Visium spot
- msigdbr BioCarta gene sets (292 sets, filtered to >= 10 genes)
- Seurat object with per-spot data

## Outputs

- Per-spot pathway module scores (continuous activity matrix)
- KLD-based p-values and FDR-adjusted p-values per gene set/pathway
- Ranked list of top 25 differentially active pathways per dataset
- hclust_haystack hierarchical clustering of top pathways
- Spatial plots of pathway activity across tissue spots (comparable to Fig. 5A-C)

## How to apply

Load the mouse Visium datasets (anterior brain, posterior brain, kidney) with spot-level counts and 2D spatial coordinates from the singleCellHaystack spatial tutorial data. Score the 292 msigdbr BioCarta gene sets (filtered to those with at least 10 genes) in every spot using Seurat AddModuleScore to get continuous per-spot pathway activity values. Z-normalize (rescale) the module scores, then run singleCellHaystack v1.0.0 with the 2D spot coordinates as input positions and the rescaled scores as the feature matrix, yielding KLD-based p-values per pathway. Rank pathways by FDR-adjusted p-value, select the top 25 differentially active pathways, cluster them with hclust_haystack, and plot the spatial distribution of example high-scoring pathways across the tissue spots, comparing against published Fig. 5A-C. The rationale is that the updated singleCellHaystack accepts continuous features such as module scores and 2-3D spatial coordinates, using Kullback-Leibler Divergence without clustering spots into arbitrary groups.

## Related tools

- **singleCellHaystack** (Predicts pathways with non-random spatial distributions from module scores using KLD-based testing on 2D spot coordinates; provides hclust_haystack clustering of top pathways) — https://github.com/alexisvdb/singleCellHaystack
- **Seurat** (Loads Visium spot-level counts and computes per-spot pathway activity via AddModuleScore)
- **msigdbr** (Supplies the 292 BioCarta gene sets used for pathway scoring)

## Examples

```
res = hs.haystack(adata, basis="pca"); res.top_features(n=25)
```

## Evaluation signals

- Pathway FDR-adjusted p-values are computed from singleCellHaystack KLD scores on z-normalized module scores with 2D spot coordinates as input positions
- Exactly the top 25 differentially active pathways per dataset are selected and cluster with hclust_haystack
- Spatial activity plots of example high-scoring pathways match the patterns in published Fig. 5A-C (anterior brain, posterior brain, kidney)
- All scored BioCarta gene sets contain at least 10 genes after filtering the 292 msigdbr sets
- Results reproduce with singleCellHaystack R version 1.0.0 or higher (CRAN version), not the legacy binary 0.3.4 branch

## Limitations

- Comparisons between multiple conditions (e.g., wild-type and knockout) cannot easily be conducted
- Results are not straightforward to incorporate into cluster-oriented scRNA-seq analyses
- Relatively longer runtimes compared to SPARK-X for spatial transcriptomics, though sensitivity is better
- The Python implementation is in beta: the hclust_haystack hierarchical clustering method for cluster_genes and the seeding grid method are not yet available

## Evidence

- [abstract] singleCellHaystack now accepts continuous features (RNA or protein expression, chromatin accessibility, module scores): "singleCellHaystack now accepts continuous features that can be RNA or protein expression, chromatin accessibility or module scores from single-cell, spatial and even bulk genomics data"
- [abstract] Handles 2-3D spatial coordinates: "it can handle 1D trajectories, 2-3D spatial coordinates, as well as higher-dimensional latent spaces as input coordinates"
- [readme] KLD-based non-random activity detection: [evidence span withheld — anchor status: external_doc]
- [readme] Visium spatial tutorial exists: [evidence span withheld — anchor status: external_doc]
- [discussion] Faster alternative for spatial: "The better sensitivity of singleCellHaystack makes it the best alternative, despite relatively longer runtimes."
