---
name: protein-marker-concordance-verification
description: Use when you have a multimodal CITE-seq h5ad with normalized protein
  counts and protein-derived PCs, and need to validate clustering-free protein marker
  predictions against Seurat-style Wilcoxon/t-test baselines.
when_to_use_negative:
- 'The goal is a multi-condition comparison (e.g. wild-type vs knockout): the article
  states such comparisons ''cannot be easily conducted'' with singleCellHaystack'
- 'Results must feed directly into a cluster-oriented analysis: the discussion notes
  singleCellHaystack results are ''not as straightforward to incorporate'' into cluster-oriented
  scRNA-seq analyses'
- The feature space is spatial transcriptomics where SPARK-X's faster runtime is preferred
  and sensitivity trade-offs are acceptable
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_2229
tools:
- name: singleCellHaystack
  role: Clustering-independent DAF prediction run on normalized protein expression
    with protein-derived PCs as the coordinate space (basis="pca")
  repo: https://github.com/alexisvdb/singleCellHaystack
- name: singleCellHaystack (Python)
  role: Python implementation (hs.haystack, res.top_features) used to run the protein
    analysis on the atlas-level PBMC CITE-seq data via scanpy integration
  repo: https://github.com/ddiez/singleCellHaystack-py
- name: scanpy
  role: Reads the .h5ad CITE-seq atlas (read_h5ad) and provides rank_genes_groups
    (Wilcoxon rank-sum and t-test) as the cluster-based baseline
- name: R
  role: Runtime for the original singleCellHaystack R package (CRAN v1.0.0+), an alternative
    to the Python implementation for the same ranking comparison
  repo: https://github.com/alexisvdb/singleCellHaystack
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-023-38965-2
    title: A universal tool for predicting differentially active features in single-cell
      and spatial genomics data
  source_content_hash: sha256:846f50397ed4c0dafab28815c1cc2942fe164efe11920c2544230441401eed87
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/alexisvdb/singleCellHaystack
  tools:
  - Python
  - R
  - scanpy
  - singleCellHaystack
  - singleCellHaystack (Python)
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# protein-marker-concordance-verification

## Summary

Verifies that singleCellHaystack's clustering-independent prediction of differentially active proteins on a CITE-seq PBMC atlas (normalized 228-protein matrix + 50 protein-based PCs) agrees with cluster-based differential expression baselines (Wilcoxon rank-sum and t-test), and pinpoints proteins whose rankings diverge.

## When to use

Use when you have a multimodal CITE-seq h5ad with normalized protein counts and protein-derived PCs, and need to validate clustering-free protein marker predictions against Seurat-style Wilcoxon/t-test baselines.

## When NOT to use

- The goal is a multi-condition comparison (e.g. wild-type vs knockout): the article states such comparisons 'cannot be easily conducted' with singleCellHaystack
- Results must feed directly into a cluster-oriented analysis: the discussion notes singleCellHaystack results are 'not as straightforward to incorporate' into cluster-oriented scRNA-seq analyses
- The feature space is spatial transcriptomics where SPARK-X's faster runtime is preferred and sensitivity trade-offs are acceptable

## Inputs

- Multimodal PBMC CITE-seq atlas as .h5ad (>200k cells, 228 proteins) containing normalized protein expression and 50 protein-based principal components
- Normalized protein expression matrix (cells × 228 proteins)
- Cluster labels for the cluster-based differential expression baselines

## Outputs

- singleCellHaystack result object with ranked proteins (haystack scores and p-values) for all 228 proteins
- top_features.csv: full ranking of all 228 proteins with haystack scores and p-values (top 10 via res.top_features(n=10))
- Comparison plots (plot_compare_ranks, plot_pval_rank, plot_pval_hist) contrasting haystack vs Wilcoxon/t-test rankings
- Concordance assessment vs Supplementary Fig. S12, listing divergent proteins (CD2, CD3-1, CD133-1/CD133-2, CD303/CD45-2)

## How to apply

Load the multimodal PBMC CITE-seq atlas (>200k cells, 228 proteins) with scanpy's read_h5ad, ensuring normalized protein expression and the 50 protein-based principal components are present. Run hs.haystack with basis="pca", i.e. using the 50 protein-derived PCs as the cell coordinate space without clustering, on the normalized 228-protein matrix. Extract top features via res.top_features(n=10) and save the full ranking of all 228 proteins with haystack scores and p-values as top_features.csv. Run cluster-based baselines — Wilcoxon rank-sum and t-test as in Seurat — on the same normalized protein data via scanpy's rank_genes_groups, then compare rankings using plot_compare_ranks, plot_pval_rank and plot_pval_hist. Judge correctness by reproducing the Supplementary Fig. S12 agreement pattern: concordant calls for CD64 and CLEC12A, high singleCellHaystack scores for CD2 and CD3-1 that cluster-based methods miss, and Wilcoxon/t-test bias toward CD133-1/CD133-2 and CD303/CD45-2.

## Related tools

- **singleCellHaystack** (Clustering-independent DAF prediction run on normalized protein expression with protein-derived PCs as the coordinate space (basis="pca")) — https://github.com/alexisvdb/singleCellHaystack
- **singleCellHaystack (Python)** (Python implementation (hs.haystack, res.top_features) used to run the protein analysis on the atlas-level PBMC CITE-seq data via scanpy integration) — https://github.com/ddiez/singleCellHaystack-py
- **scanpy** (Reads the .h5ad CITE-seq atlas (read_h5ad) and provides rank_genes_groups (Wilcoxon rank-sum and t-test) as the cluster-based baseline)
- **R** (Runtime for the original singleCellHaystack R package (CRAN v1.0.0+), an alternative to the Python implementation for the same ranking comparison) — https://github.com/alexisvdb/singleCellHaystack

## Examples

```
import scanpy as sc, singleCellHaystack as hs; adata = sc.read_h5ad("data.h5ad"); res = hs.haystack(adata, basis="pca"); res.top_features(n=10)
```

## Evaluation signals

- Ranking file top_features.csv contains exactly all 228 proteins with both haystack scores and p-values
- Concordance pattern matches Supplementary Fig. S12: CD64 and CLEC12A called concordantly by both approaches
- Divergence pattern reproduced: CD2 and CD3-1 receive high singleCellHaystack scores but are not detected by Wilcoxon/t-test cluster baselines
- Wilcoxon and t-test baselines show bias toward CD133-1/CD133-2 and CD303/CD45-2 that singleCellHaystack does not rank highly
- hs.haystack ran with basis="pca" on the 50 protein-derived PCs (not transcriptome PCs) and without clustering, per the documented workflow

## Limitations

- Comparisons between multiple conditions (e.g. wild-type and knockout) cannot be easily conducted with singleCellHaystack
- Results are not straightforward to incorporate into cluster-oriented scRNA-seq analyses
- The Python implementation is in beta: weights.advanced.Q, the seeding grid method, and hierarchical clustering for cluster_genes are not yet available
- For spatial transcriptomics, singleCellHaystack has relatively longer runtimes than SPARK-X, though with better sensitivity

## Evidence

- [results] singleCellHaystack's protein predictions on the CITE-seq PBMC data were generally consistent with cluster-based prediction approaches, though ranking differences were observed for some proteins (Supplementary Fig. S12).: [evidence span withheld — anchor status: not_found]
- [methods] res = hs.haystack(adata, basis="pca"): "res = hs.haystack(adata, basis="pca")"
- [methods] res.top_features(n=10): "res.top_features(n=10)"
- [abstract] We previously developed singleCellHaystack, a method for predicting differentially expressed genes from single-cell transcriptome data, without relying on comparisons between clusters of cells.: "a method for predicting differentially expressed genes from single-cell transcriptome data, without relying on comparisons between clusters of cells"
- [readme] singleCellHaystack uses Kullback-Leibler Divergence to find features that have patterns of activity in subsets of samples that are non-randomly positioned inside any input space.: [evidence span withheld — anchor status: external_doc]
- [discussion] One is that comparisons between multiple conditions (e.g., wild-type and knockout) cannot be easily conducted.: "comparisons between multiple conditions (e.g., wild-type and knockout) cannot be easily conducted"
