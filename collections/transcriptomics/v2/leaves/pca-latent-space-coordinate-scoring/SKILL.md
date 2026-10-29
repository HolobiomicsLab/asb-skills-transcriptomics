---
name: pca-latent-space-coordinate-scoring
description: Use when you have normalized single-cell feature expression (protein
  counts, RNA, accessibility) plus a reduced-dimensional latent space (e.g., 50 protein-based
  PCs stored in an AnnData object) and need clustering-independent prediction of differentially
  active features.
when_to_use_negative:
- Comparing features across multiple conditions (e.g., wild-type vs knockout) — the
  authors note such comparisons cannot easily be conducted in this version
- Analyses that require DEG results to map directly onto a cluster-oriented scRNA-seq
  workflow — results are not straightforward to incorporate into cluster-based pipelines
- Spatial transcriptomics when runtime is the overriding constraint — SPARK-X is faster,
  though less sensitive
edam_operation: http://edamontology.org/operation_3222
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: singleCellHaystack
  role: Performs the KLD-based, clustering-independent scoring of features against
    the PCA latent coordinate space (basis="pca")
  repo: https://github.com/alexisvdb/singleCellHaystack
- name: singleCellHaystack (Python package, version 0.0.5)
  role: Python implementation used on atlas-level AnnData objects; provides hs.haystack,
    res.top_features and plotting/compare-ranks helpers
  repo: https://github.com/ddiez/singleCellHaystack-py
- name: scanpy
  role: Loads the .h5ad CITE-seq atlas (read_h5ad) and runs the Wilcoxon rank-sum
    / t-test rank_genes_groups baselines for comparison
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-023-38965-2
    title: A universal tool for predicting differentially active features in single-cell
      and spatial genomics data
  source_content_hash: sha256:1211abd2e303f5eafcda7537504f177c04b523a9985fe6d27ab89984ec241528
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/alexisvdb/singleCellHaystack
  tools:
  - Python
  - scanpy
  - singleCellHaystack
  - singleCellHaystack (Python package, version 0.0.5)
  edam_operation: http://edamontology.org/operation_3222
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# pca-latent-space-coordinate-scoring

## Summary

Scores continuous features (e.g., 228 CITE-seq antibody-derived protein tags) for differential activity against a PCA-derived latent cell coordinate space using singleCellHaystack's Kullback-Leibler divergence test, without clustering cells. The Python implementation (singleCellHaystack-py) accepts an AnnData object and a declared PC basis, returning per-feature scores and p-values.

## When to use

Use when you have normalized single-cell feature expression (protein counts, RNA, accessibility) plus a reduced-dimensional latent space (e.g., 50 protein-based PCs stored in an AnnData object) and need clustering-independent prediction of differentially active features.

## When NOT to use

- Comparing features across multiple conditions (e.g., wild-type vs knockout) — the authors note such comparisons cannot easily be conducted in this version
- Analyses that require DEG results to map directly onto a cluster-oriented scRNA-seq workflow — results are not straightforward to incorporate into cluster-based pipelines
- Spatial transcriptomics when runtime is the overriding constraint — SPARK-X is faster, though less sensitive

## Inputs

- AnnData .h5ad object (e.g., PBMC CITE-seq atlas, >200k cells, 228 antibodies) with normalized protein expression
- Precomputed PCA embedding in the AnnData object (e.g., 50 protein-based principal components) referenced via basis="pca"
- Optionally, cluster labels for running cluster-based baseline comparisons

## Outputs

- Haystack result object with per-feature KLD-based scores and p-values for all 228 proteins
- Top-ranked features (e.g., res.top_features(n=10))
- Full feature ranking exported as top_features.csv (scores and p-values)
- Comparison plots (plot_compare_ranks, plot_pval_rank, plot_pval_hist) against Wilcoxon/t-test baselines

## How to apply

Load the multimodal atlas with scanpy's sc.read_h5ad into an AnnData object containing normalized protein expression and precomputed protein-based PCs. Call hs.haystack(adata, basis="pca") so that the KLD-based scoring operates in the stored PC coordinate space rather than on clusters; singleCellHaystack uses Kullback-Leibler Divergence to find features whose activity is non-randomly positioned in the input space, using continuous activity levels rather than a hard binary detection threshold. Extract the top hits with res.top_features(n=10) and save the full ranking (scores and p-values for all features, e.g., 228 proteins) to CSV. Optionally cross-check against cluster-based baselines (Wilcoxon rank sum, t-test via scanpy rank_genes_groups) using plot_compare_ranks, plot_pval_rank, and plot_pval_hist to reveal rank differences. The Python implementation is preferred for atlas-scale data (e.g., >200k cells) because it scales to millions of cells.

## Related tools

- **singleCellHaystack** (Performs the KLD-based, clustering-independent scoring of features against the PCA latent coordinate space (basis="pca")) — https://github.com/alexisvdb/singleCellHaystack
- **singleCellHaystack (Python package, version 0.0.5)** (Python implementation used on atlas-level AnnData objects; provides hs.haystack, res.top_features and plotting/compare-ranks helpers) — https://github.com/ddiez/singleCellHaystack-py
- **scanpy** (Loads the .h5ad CITE-seq atlas (read_h5ad) and runs the Wilcoxon rank-sum / t-test rank_genes_groups baselines for comparison)

## Examples

```
import scanpy as sc, singleCellHaystack as hs; adata = sc.read_h5ad("data.h5ad"); res = hs.haystack(adata, basis="pca"); res.top_features(n=10)
```

## Evaluation signals

- The full output CSV contains exactly one ranked row per input feature (228 proteins for the PBMC CITE-seq case), each with a haystack score and p-value
- Agreement pattern with cluster-based baselines matches Supplementary Fig. S12: concordant calls for CD64 and CLEC12A; high haystack scores for CD2 and CD3-1 that cluster-based methods miss; Wilcoxon/t-test bias toward CD133-1/CD133-2 and CD303/CD45-2
- basis="pca" resolves without error, confirming the AnnData object contains the 50 protein-based PCs used as the coordinate space
- p-value histogram (plot_pval_hist) shows the expected distribution for non-random feature activity rather than uniform noise
- Runtime stays tractable on atlas-scale input (>200k cells) due to the sparse-matrix, scalable Python implementation

## Limitations

- Comparisons between multiple conditions (e.g., wild-type and knockout) cannot be easily conducted in this version
- Results are not straightforward to incorporate into cluster-oriented scRNA-seq analyses
- The Python package is in beta: weights.advanced.Q, the seeding grid method, and hierarchical clustering for cluster_genes are not yet available
- Longer runtimes than SPARK-X for spatial transcriptomics, though with better sensitivity

## Evidence

- [methods] res = hs.haystack(adata, basis="pca"): "res = hs.haystack(adata, basis="pca")"
- [readme] singleCellHaystack uses KLD to find non-random activity patterns: [evidence span withheld — anchor status: external_doc]
- [intro] Python implementation scales to millions of cells: "a Python implementation has been developed which enables the efficient application of singleCellHaystack to atlas level datasets with millions of cells"
- [abstract] Handles higher-dimensional latent spaces as input coordinates: "it can handle 1D trajectories, 2-3D spatial coordinates, as well as higher-dimensional latent spaces as input coordinates"
- [discussion] Multi-condition comparisons limitation: "One is that comparisons between multiple conditions (e.g., wild-type and knockout) cannot be easily conducted."
