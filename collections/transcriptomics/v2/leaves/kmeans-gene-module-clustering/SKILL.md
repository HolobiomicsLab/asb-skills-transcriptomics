---
name: kmeans-gene-module-clustering
description: Use when you have a ranked list of trajectory DEGs (e.g. top 1000 from
  singleCellHaystack on 1D pseudotime coordinates) and need to partition them into
  expression-pattern modules along pseudotime for visualization and interpretation.
when_to_use_negative:
- When the goal is cluster-oriented differential expression between predefined cell
  groups rather than unbiased module discovery along a trajectory — the paper notes
  singleCellHaystack results are 'not as straightforward to incorporate ... into a
  cluster-oriented scRNA-seq analysis'
- When a hierarchical clustering variant of cluster_genes is specifically required
  and only the Python package is available — the ddiez/singleCellHaystack-py README
  lists 'Hierarchical clustering method for cluster_genes' as a missing feature
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: singleCellHaystack
  role: 'Upstream predictor: identifies the differentially active genes along the
    1D pseudotime whose top-scoring members are clustered into modules'
  repo: https://github.com/alexisvdb/singleCellHaystack
- name: Seurat
  role: Preprocessing of the scRNA-seq count matrix (normalization, HVG selection,
    scaling, PCA) producing the scaled expression profiles used for k-means
- name: monocle3
  role: Supplies the 1D pseudotime coordinates (learn_graph / order_cells) used as
    the input space for DEG prediction
  repo: https://cole-trapnell-lab.github.io/monocle3
- name: R
  role: Runtime for the workflow including base kmeans() clustering of the top DEG
    profiles with a fixed seed
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41598-023-38965-2
    title: A universal tool for predicting differentially active features in single-cell
      and spatial genomics data
  source_content_hash: sha256:63b0a30d565673481065cda065b858a83157e6310c5680564e7019232b93104f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/alexisvdb/singleCellHaystack
  tools:
  - R
  - Seurat
  - monocle3
  - singleCellHaystack
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# kmeans-gene-module-clustering

## Summary

Clusters top-scoring trajectory DEGs (predicted with singleCellHaystack along a monocle3 pseudotime) into k-means modules on their scaled expression profiles, producing a gene-to-module table whose mean-expression-along-pseudotime plots reveal distinct gene expression change patterns (e.g. k=6 modules matching Fig 3B-C).

## When to use

Use when you have a ranked list of trajectory DEGs (e.g. top 1000 from singleCellHaystack on 1D pseudotime coordinates) and need to partition them into expression-pattern modules along pseudotime for visualization and interpretation.

## When NOT to use

- When the goal is cluster-oriented differential expression between predefined cell groups rather than unbiased module discovery along a trajectory — the paper notes singleCellHaystack results are 'not as straightforward to incorporate ... into a cluster-oriented scRNA-seq analysis'
- When a hierarchical clustering variant of cluster_genes is specifically required and only the Python package is available — the ddiez/singleCellHaystack-py README lists 'Hierarchical clustering method for cluster_genes' as a missing feature

## Inputs

- Haystack result object from singleCellHaystack run on a 1D pseudotime coordinate vector
- Top-N DEG list (e.g. top 1000 via top_features) with scaled expression profiles across cells ordered by pseudotime
- Processed expression matrix from Seurat-normalized scRNA-seq data (e.g. Tabula Muris thymus DN -> SP trajectory)
- k value for k-means (k=6 in the thymus reference workflow)

## Outputs

- Gene-to-module assignment table as CSV (thymus_deg_modules.csv)
- Per-module mean expression vs pseudotime line plots (one per module, k=6 total)

## How to apply

After running singleCellHaystack v1.0.0+ with 1D pseudotime coordinates as input, extract top DEGs via top_features (e.g. n=1000). Scale each gene's expression profile across cells ordered by pseudotime, then run k-means with a fixed seed and a predefined k (the Tabula Muris thymus task used k=6) — either via the cluster_genes function of singleCellHaystack or a direct R kmeans() call. Write the gene-to-module assignment table to CSV (e.g. thymus_deg_modules.csv). Validate by plotting each module's mean expression along pseudotime and confirming the module shapes qualitatively match the expected reference figure (Fig 3B-C). Note: the Python implementation (ddiez/singleCellHaystack-py) lacks the hierarchical clustering method for cluster_genes, so use the R package or plain kmeans for module clustering.

## Related tools

- **singleCellHaystack** (Upstream predictor: identifies the differentially active genes along the 1D pseudotime whose top-scoring members are clustered into modules) — https://github.com/alexisvdb/singleCellHaystack
- **Seurat** (Preprocessing of the scRNA-seq count matrix (normalization, HVG selection, scaling, PCA) producing the scaled expression profiles used for k-means)
- **monocle3** (Supplies the 1D pseudotime coordinates (learn_graph / order_cells) used as the input space for DEG prediction) — https://cole-trapnell-lab.github.io/monocle3
- **R** (Runtime for the workflow including base kmeans() clustering of the top DEG profiles with a fixed seed)

## Examples

```
set.seed(123); km <- kmeans(scaled_profiles[top1000, ], centers = 6); write.csv(data.frame(gene = rownames(scaled_profiles)[top1000], module = km$cluster), "thymus_deg_modules.csv")
```

## Evaluation signals

- The gene-to-module table contains exactly one module assignment per top DEG (1000 rows for top_features(n=1000)) with no missing genes
- The module assignment table resolves to exactly k modules (k=6 for the thymus workflow)
- Per-module mean-expression-along-pseudotime plots show distinct, non-degenerate profiles and qualitatively match the reference patterns in Fig 3B-C
- Re-running k-means with the same fixed seed reproduces identical gene-to-module assignments
- Input DEG ranking matches top_features output from the singleCellHaystack result on the monocle3 pseudotime coordinates

## Limitations

- k-means requires choosing k a priori (k=6 was set to match the reference figure); no data-driven k selection is described in the workflow
- Multi-condition comparisons (e.g. wild-type vs knockout) are not supported by singleCellHaystack upstream, so modules are trajectory-specific only
- The Python singleCellHaystack package is in beta and lacks the hierarchical clustering method for cluster_genes, restricting some clustering variants to the R implementation

## Evidence

- [abstract] it can handle 1D trajectories, 2-3D spatial coordinates, as well as higher-dimensional latent spaces as input coordinates: "it can handle 1D trajectories, 2-3D spatial coordinates, as well as higher-dimensional latent spaces as input coordinates"
- [methods] Hierarchical clustering method for `cluster_genes`.: "Hierarchical clustering method for `cluster_genes`."
- [readme] `singleCellHaystack` uses Kullback-Leibler Divergence to find features that have patterns of activity in subsets of samples that are non-randomly positioned inside any input space.: [evidence span withheld — anchor status: external_doc]
