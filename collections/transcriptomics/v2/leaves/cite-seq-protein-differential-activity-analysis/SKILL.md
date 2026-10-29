---
name: cite-seq-protein-differential-activity-analysis
description: Use when analyzing a multimodal CITE-seq/PBMC h5ad atlas (>200k cells,
  228 proteins) to rank surface proteins by differential activity in PCA space without
  clustering, and to compare that ranking against cluster-based DE baselines.
when_to_use_negative:
- Comparisons between multiple conditions (e.g., wild-type vs knockout) are needed
  — the article notes these 'cannot be easily conducted' with singleCellHaystack
- Results must be integrated into a cluster-oriented scRNA-seq analysis — the article
  notes incorporating haystack results into cluster-based analyses is 'not as straightforward'
- Spatial transcriptomics where raw runtime is the binding constraint — the article
  notes singleCellHaystack has 'relatively longer runtimes' than SPARK-X
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: singleCellHaystack
  role: Python package (pip install singleCellHaystack) used to run hs.haystack with
    basis="pca" on the protein matrix and extract top_features rankings
  repo: https://github.com/ddiez/singleCellHaystack-py
- name: scanpy
  role: Used to read the multimodal PBMC h5ad atlas (sc.read_h5ad) and to run cluster-based
    differential expression baselines (rank_genes_groups, Wilcoxon/t-test)
- name: singleCellHaystack (R package)
  role: R/CRAN implementation (>= 1.0.0) of the same DAF-prediction method for users
    working in R; provides plot_compare_ranks, plot_pval_rank, plot_pval_hist
  repo: https://github.com/alexisvdb/singleCellHaystack
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-023-38965-2
    title: A universal tool for predicting differentially active features in single-cell
      and spatial genomics data
  source_content_hash: sha256:6ba580e289035e42be84600d48cd98931ea7ef767262ef41a183e6047d792e01
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ddiez/singleCellHaystack-py
  tools:
  - Python
  - scanpy
  - singleCellHaystack
  - singleCellHaystack (R package)
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# cite-seq-protein-differential-activity-analysis

## Summary

Cluster-independent prediction of differentially active surface proteins from CITE-seq data using singleCellHaystack, run on protein-derived PCs rather than cell clusters, and benchmarked against Seurat-style Wilcoxon rank-sum and t-test baselines via scanpy rank_genes_groups.

## When to use

Use when analyzing a multimodal CITE-seq/PBMC h5ad atlas (>200k cells, 228 proteins) to rank surface proteins by differential activity in PCA space without clustering, and to compare that ranking against cluster-based DE baselines.

## When NOT to use

- Comparisons between multiple conditions (e.g., wild-type vs knockout) are needed — the article notes these 'cannot be easily conducted' with singleCellHaystack
- Results must be integrated into a cluster-oriented scRNA-seq analysis — the article notes incorporating haystack results into cluster-based analyses is 'not as straightforward'
- Spatial transcriptomics where raw runtime is the binding constraint — the article notes singleCellHaystack has 'relatively longer runtimes' than SPARK-X

## Inputs

- Multimodal PBMC CITE-seq atlas as .h5ad (>200k cells) with normalized protein expression (228 proteins) read via scanpy read_h5ad
- 50 protein-based principal components stored in the AnnData object (used as basis="pca" cell coordinates)

## Outputs

- Full ranking of all 228 proteins with singleCellHaystack scores and p-values (top_features.csv)
- Top 10 features from res.top_features(n=10)
- Cluster-based baseline rankings from scanpy rank_genes_groups (Wilcoxon rank sum and t-test)
- Comparison plots: plot_compare_ranks, plot_pval_rank, plot_pval_hist

## How to apply

Load the multimodal PBMC CITE-seq atlas h5ad with scanpy's read_h5ad; it must contain normalized protein expression for 228 proteins and the 50 protein-based principal components. Run hs.haystack on the normalized protein matrix with basis="pca", using the 50 protein-derived PCs as the cell coordinate space so no clustering is performed. Extract top features with res.top_features(n=10) and save the full 228-protein ranking (haystack scores and p-values) as top_features.csv. Run cluster-based baselines (Wilcoxon rank sum and t-test, as in Seurat, via scanpy rank_genes_groups) on the same normalized protein data, then compare rankings using plot_compare_ranks, plot_pval_rank, and plot_pval_hist. The method uses continuous activity levels and Kullback-Leibler divergence to find features non-randomly positioned in the input space, without assuming a particular statistical distribution.

## Related tools

- **singleCellHaystack** (Python package (pip install singleCellHaystack) used to run hs.haystack with basis="pca" on the protein matrix and extract top_features rankings) — https://github.com/ddiez/singleCellHaystack-py
- **scanpy** (Used to read the multimodal PBMC h5ad atlas (sc.read_h5ad) and to run cluster-based differential expression baselines (rank_genes_groups, Wilcoxon/t-test))
- **singleCellHaystack (R package)** (R/CRAN implementation (>= 1.0.0) of the same DAF-prediction method for users working in R; provides plot_compare_ranks, plot_pval_rank, plot_pval_hist) — https://github.com/alexisvdb/singleCellHaystack

## Examples

```
import scanpy as sc, singleCellHaystack as hs; adata = sc.read_h5ad("data.h5ad"); res = hs.haystack(adata, basis="pca"); res.top_features(n=10)
```

## Evaluation signals

- Full output table contains exactly 228 protein rows with haystack scores and p-values, ordered into a ranking
- Supplementary Fig. S12 pattern reproduced: concordant calls for CD64 and CLEC12A between singleCellHaystack and cluster-based methods
- Ranking differences match the expected bias pattern: high singleCellHaystack scores for CD2 and CD3-1 that cluster-based methods miss, while Wilcoxon/t-test favor CD133-1/CD133-2 and CD303/CD45-2
- plot_pval_rank and plot_pval_hist show a sensible p-value distribution without mass inflation of near-zero p-values
- haystack was run with basis="pca" (the 50 protein PCs), i.e., no clustering was performed at any step

## Limitations

- Comparisons between multiple conditions (e.g., wild-type and knockout) cannot be easily conducted
- Results are not straightforward to incorporate into cluster-oriented scRNA-seq analyses
- Relatively longer runtimes compared to SPARK-X for spatial transcriptomics; multi-condition comparison and cluster-integration features are future work
- The Python implementation is in beta: weights.advanced.Q, the seeding grid method, and cluster_genes hierarchical clustering are missing

## Evidence

- [abstract] cluster-independent DAF prediction: "We previously developed singleCellHaystack, a method for predicting differentially expressed genes from single-cell transcriptome data, without relying on comparisons between clusters of cells."
- [abstract] continuous features incl. protein expression: "singleCellHaystack now accepts continuous features that can be RNA or protein expression, chromatin accessibility or module scores from single-cell, spatial and even bulk genomics data"
- [intro] R and Python availability: "The new singleCellHaystack method is implemented as an R package and is available from CRAN (version 1.0.0 and higher) and GitHub"
- [discussion] no distributional assumptions: "These improvements, together with the fact that it does not make strong assumptions about the statistical distribution of the input data, make singleCellHaystack applicable to a wide range of data"
- [readme] package scope and method description: [evidence span withheld — anchor status: external_doc]
- [readme] KL divergence criterion: [evidence span withheld — anchor status: external_doc]
- [methods] example invocation pattern: "adata = sc.read_h5ad("data.h5ad")"
