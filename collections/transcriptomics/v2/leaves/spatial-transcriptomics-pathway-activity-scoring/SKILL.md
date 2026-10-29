---
name: spatial-transcriptomics-pathway-activity-scoring
description: Use when analyzing Visium spatial transcriptomics data to identify biological
  pathways (gene sets) whose per-spot activity levels are non-randomly distributed
  across tissue coordinates, rather than testing individual genes.
when_to_use_negative:
- Comparing pathway activity between multiple conditions (e.g., wild-type vs knockout)
  — singleCellHaystack currently cannot easily conduct multi-condition comparisons
- When a cluster-oriented differential expression framework is required — singleCellHaystack
  results are not straightforward to incorporate into cluster-based scRNA-seq analyses
- When computational speed on spatial data is the priority and some sensitivity can
  be sacrificed — SPARK-X is faster, though singleCellHaystack has better sensitivity
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3523
- http://edamontology.org/topic_0203
tools:
- name: singleCellHaystack
  role: KLD-based detection of pathways with non-random spatial activity using 2D
    spot coordinates as positions and z-normalized module scores as features (R package
    v1.0.0)
  repo: https://github.com/alexisvdb/singleCellHaystack
- name: Seurat
  role: Loading Visium data and computing per-spot pathway module scores with AddModuleScore
  repo: https://github.com/satijalab/seurat-wrappers
- name: msigdbr
  role: Source of the 292 BioCarta gene sets, filtered to >= 10 genes before scoring
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-023-38965-2
    title: A universal tool for predicting differentially active features in single-cell
      and spatial genomics data
  source_content_hash: sha256:523823c6863cf3b46d6daa8a887dc03d769bdba8b569e58e18d7ddc80a2ea85e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/alexisvdb/singleCellHaystack
  tools:
  - R
  - Seurat
  - msigdbr
  - singleCellHaystack
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3523
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# spatial-transcriptomics-pathway-activity-scoring

## Summary

Scores BioCarta pathway activity per Visium spot using Seurat AddModuleScore on msigdbr gene sets, then applies singleCellHaystack (v1.0.0) with 2D spot coordinates and z-normalized module scores as features to detect pathways with highly non-random spatial distributions via KLD-based p-values.

## When to use

Use when analyzing Visium spatial transcriptomics data to identify biological pathways (gene sets) whose per-spot activity levels are non-randomly distributed across tissue coordinates, rather than testing individual genes.

## When NOT to use

- Comparing pathway activity between multiple conditions (e.g., wild-type vs knockout) — singleCellHaystack currently cannot easily conduct multi-condition comparisons
- When a cluster-oriented differential expression framework is required — singleCellHaystack results are not straightforward to incorporate into cluster-based scRNA-seq analyses
- When computational speed on spatial data is the priority and some sensitivity can be sacrificed — SPARK-X is faster, though singleCellHaystack has better sensitivity

## Inputs

- Visium spatial transcriptomics spot-level count matrix (Seurat object) for e.g. mouse anterior brain, posterior brain, or kidney
- 2D spot spatial coordinates
- msigdbr BioCarta gene sets (292 sets, filtered to >= 10 genes)

## Outputs

- Per-spot continuous module scores for each pathway (AddModuleScore output)
- KLD-based p-values and FDR-adjusted p-values per pathway from singleCellHaystack
- Ranked list of top 25 differentially active pathways per dataset
- Hierarchical clustering of top pathways via hclust_haystack and spatial distribution plots of example high-scoring pathways

## How to apply

Load mouse Visium datasets (e.g., anterior brain, posterior brain, kidney) with spot-level counts and 2D spatial coordinates. Retrieve 292 msigdbr BioCarta gene sets, filter to those with at least 10 genes, and compute per-spot pathway activity with Seurat AddModuleScore to obtain continuous module scores. Z-normalize the module scores and run singleCellHaystack v1.0.0, passing the 2D spot coordinates as input positions and the rescaled scores as the feature matrix; the method uses Kullback-Leibler Divergence to flag features whose activity is non-randomly positioned, without clustering spots. Rank pathways by FDR-adjusted p-value per dataset, select the top 25 differentially active pathways, and cluster them with the hclust_haystack hierarchical clustering method. The rationale is that singleCellHaystack v1.0 accepts continuous features such as module scores, avoiding the hard binary detection threshold of version 0.3.4 that overestimated significance of modest differences.

## Related tools

- **singleCellHaystack** (KLD-based detection of pathways with non-random spatial activity using 2D spot coordinates as positions and z-normalized module scores as features (R package v1.0.0)) — https://github.com/alexisvdb/singleCellHaystack
- **Seurat** (Loading Visium data and computing per-spot pathway module scores with AddModuleScore) — https://github.com/satijalab/seurat-wrappers
- **msigdbr** (Source of the 292 BioCarta gene sets, filtered to >= 10 genes before scoring)

## Examples

```
res <- haystack(x = rescaled_module_scores, position = spot_coords_2d); res <- FDR_haystack(res); pathways_top25 <- res[order(res$p_adj), ][1:25, ]; geneset_clusters <- hclust_haystack(pathways, res)
```

## Evaluation signals

- FDR-adjusted p-values per pathway are produced for each Visium dataset and the top 25 pathways can be ranked per dataset
- Spatial plots of example high-scoring pathways reproduce the tissue-spot patterns comparable to published Fig 5A-C
- hclust_haystack clustering of the top 25 pathways yields coherent pathway clusters matching the published figure
- Each retained gene set has at least 10 genes after msigdbr filtering, and module scores are z-normalized before running haystack
- Results use continuous module scores rather than binary detection, consistent with singleCellHaystack >= 1.0.0 (not the 'binary' branch v0.3.4)

## Limitations

- Comparisons between multiple conditions (e.g., wild-type and knockout) cannot be easily conducted with singleCellHaystack
- Results are not straightforward to incorporate into cluster-oriented scRNA-seq analyses
- Longer runtimes than SPARK-X for spatial transcriptomics, though with better sensitivity
- The Python implementation (singleCellHaystack-py, v0.0.5) lacks hierarchical clustering for cluster_genes, so hclust_haystack requires the R package

## Evidence

- [abstract] singleCellHaystack now accepts continuous features (RNA or protein expression, chromatin accessibility, module scores) from single-cell, spatial, and bulk genomics data: "singleCellHaystack now accepts continuous features that can be RNA or protein expression, chromatin accessibility or module scores from single-cell, spatial and even bulk genomics data"
- [intro] our approach can also be used on sets of genes (e.g., genes sharing a common annotation) for predicting differential activities of biological pathways in single-cell and spatial transcriptomics data,: "our approach can also be used on sets of genes (e.g., genes sharing a common annotation) for predicting differential activities of biological pathways in single-cell and spatial transcriptomics data,"
- [readme] singleCellHaystack uses Kullback-Leibler Divergence to find features that have patterns of activity in subsets of samples that are non-randomly positioned inside any input space.: [evidence span withheld — anchor status: external_doc]
- [readme] The Python implementation lacks the hierarchical clustering method needed to cluster top pathways.: "Hierarchical clustering method for `cluster_genes`."
- [readme] Analysis of gene set activities is provided as a documented example workflow in the package documentation.: "- [Analysis of gene set
  activities](https://alexisvdb.github.io/singleCellHaystack/articles/examples/a07_gene_sets.html)"
