---
name: hierarchical-pathway-clustering
description: Use when pathway module scores (e.g., Seurat AddModuleScore over msigdbr
  BioCarta gene sets with >=10 genes) have already been scored per spot and ranked
  by singleCellHaystack KLD-based FDR in 2D spatial data, and the top pathways (e.g.,
  top 25) need to be grouped by spatial-pattern similarity.
when_to_use_negative:
- 'You are using the Python singleCellHaystack package (v0.0.5 beta): the README lists
  ''Hierarchical clustering method for cluster_genes'' among its missing features,
  so use the R package instead.'
- You need to compare pathway activity across multiple conditions (e.g., wild-type
  vs knockout); the article states such comparisons cannot easily be conducted.
- The feature matrix is still raw or binary detection data rather than the rescaled
  continuous module scores the updated method expects.
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: singleCellHaystack
  role: Provides the KLD-based ranking of differentially active pathway module scores
    and the hclust_haystack hierarchical clustering method applied to the top pathways
  repo: https://github.com/alexisvdb/singleCellHaystack
- name: Seurat
  role: AddModuleScore computes the per-spot continuous pathway activity features
    that are rescaled and fed into haystack() before clustering
  repo: https://github.com/satijalab/seurat-wrappers
- name: msigdbr
  role: Supplies the 292 BioCarta gene sets filtered to at least 10 genes whose module
    scores are clustered
- name: singleCellHaystack-py
  role: Python implementation of the same method; noted as lacking cluster_genes hierarchical
    clustering, so not suitable for this skill
  repo: https://github.com/ddiez/singleCellHaystack-py
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41598-023-38965-2
    title: A universal tool for predicting differentially active features in single-cell
      and spatial genomics data
  source_content_hash: sha256:c0a396fa6d0ada37cde30c757c2d1b74513f9847eee499287aea4aff6a875c24
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/alexisvdb/singleCellHaystack
  tools:
  - R
  - Seurat
  - msigdbr
  - singleCellHaystack
  - singleCellHaystack-py
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# hierarchical-pathway-clustering

## Summary

Groups top-ranked differentially active pathways by similarity of their spatial activity patterns using singleCellHaystack's hierarchical clustering method (hclust_haystack / cluster_genes), as in the BioCarta module-score analysis of Visium datasets (Fig. 5).

## When to use

Use when pathway module scores (e.g., Seurat AddModuleScore over msigdbr BioCarta gene sets with >=10 genes) have already been scored per spot and ranked by singleCellHaystack KLD-based FDR in 2D spatial data, and the top pathways (e.g., top 25) need to be grouped by spatial-pattern similarity.

## When NOT to use

- You are using the Python singleCellHaystack package (v0.0.5 beta): the README lists 'Hierarchical clustering method for cluster_genes' among its missing features, so use the R package instead.
- You need to compare pathway activity across multiple conditions (e.g., wild-type vs knockout); the article states such comparisons cannot easily be conducted.
- The feature matrix is still raw or binary detection data rather than the rescaled continuous module scores the updated method expects.

## Inputs

- singleCellHaystack result object from running haystack() with z-normalized pathway module scores as the feature matrix and 2D Visium spot coordinates as input positions
- Per-spot continuous pathway module scores (e.g., Seurat AddModuleScore output for 292 msigdbr BioCarta gene sets filtered to >=10 genes)
- FDR-adjusted KLD p-values per pathway, ranked per dataset

## Outputs

- Hierarchical clustering of the top 25 differentially active pathways (dendrogram / pathway cluster assignments)
- Plots of pathway clusters and spatial distributions of example high-scoring pathways across Visium tissue spots (comparable to Fig. 5A-C)

## How to apply

First rank the KLD-based p-values from singleCellHaystack (run on z-normalized, rescaled module scores with the 2D Visium spot coordinates as input positions) and select the top 25 pathways by FDR-adjusted p-value per dataset. Then apply the hierarchical clustering method exposed by the R package — hclust_haystack — to cluster these top pathways according to their non-random spatial activity patterns, and plot the resulting pathway clusters. The clustering operates on the same continuous feature (module-score) values that were passed to haystack(), not on raw counts. Finally, visualize the spatial distribution of example high-scoring pathways across tissue spots and compare the cluster assignments and spatial maps against the published Fig. 5A-C. Use the R package (CRAN singleCellHaystack >= 1.0.0) for this step, because the Python implementation (singleCellHaystack 0.0.5, beta) explicitly does not yet provide the hierarchical clustering method for cluster_genes.

## Related tools

- **singleCellHaystack** (Provides the KLD-based ranking of differentially active pathway module scores and the hclust_haystack hierarchical clustering method applied to the top pathways) — https://github.com/alexisvdb/singleCellHaystack
- **Seurat** (AddModuleScore computes the per-spot continuous pathway activity features that are rescaled and fed into haystack() before clustering) — https://github.com/satijalab/seurat-wrappers
- **msigdbr** (Supplies the 292 BioCarta gene sets filtered to at least 10 genes whose module scores are clustered)
- **singleCellHaystack-py** (Python implementation of the same method; noted as lacking cluster_genes hierarchical clustering, so not suitable for this skill) — https://github.com/ddiez/singleCellHaystack-py

## Examples

```
res <- haystack(x = rescaled_module_scores, coordinates = spot_coords_2d); hc <- hclust_haystack(res, features = top25_pathways)
```

## Evaluation signals

- The FDR-ranked top 25 pathway set per dataset matches the published Fig. 5 ranking before clustering.
- The hclust_haystack dendrogram reproduces the pathway cluster groups shown in Fig. 5A-C (same pathways co-clustered per dataset).
- Spatial maps of example high-scoring pathways show coherent, tissue-structure-aligned patterns across Visium spots rather than random scatter.
- Input feature matrix consists of z-normalized (rescaled) continuous module scores, not raw counts or binary detections.
- Clustering is run on >= the 25 top-FDR pathways per dataset with singleCellHaystack R >= 1.0.0.

## Limitations

- Comparisons between multiple conditions (e.g., wild-type and knockout) cannot easily be conducted with singleCellHaystack.
- singleCellHaystack has relatively longer runtimes than SPARK-X for spatial transcriptomics, though its sensitivity is better.
- Results are not straightforward to incorporate into cluster-oriented scRNA-seq analyses.
- The Python implementation is beta and lacks the hierarchical clustering method for cluster_genes (among other features).

## Evidence

- [methods] Hierarchical clustering method for `cluster_genes`.: "Hierarchical clustering method for `cluster_genes`."
- [intro] our approach can also be used on sets of genes (e.g., genes sharing a common annotation) for predicting differential activities of biological pathways: "our approach can also be used on sets of genes (e.g., genes sharing a common annotation) for predicting differential activities of biological pathways in single-cell and spatial transcriptomics data,"
- [readme] singleCellHaystack uses Kullback-Leibler Divergence to find features that have patterns of activity in subsets of samples that are non-randomly positioned: [evidence span withheld — anchor status: external_doc]
- [readme] This package is currently in beta. The most important functionality in the R package works, but some features are not yet available.: "This package is currently in beta. The most important functionality in the R package works, but some features are not yet available."
