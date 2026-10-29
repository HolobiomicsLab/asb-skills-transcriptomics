---
name: differential-activity-rank-comparison
description: Use when you need to validate a clustering-independent DAF/DEG method
  (singleCellHaystack) against cluster-based baselines (Wilcoxon rank-sum, t-test)
  by comparing rankings of the same features on shared normalized data.
when_to_use_negative:
- Your analysis goal is a cluster-oriented scRNA-seq workflow — the article notes
  singleCellHaystack results are not straightforward to incorporate into cluster-oriented
  analyses
- You need formal multi-condition comparisons (e.g. wild-type vs knockout) — not yet
  supported by singleCellHaystack
- Spatial transcriptomics where runtime is the priority — SPARK-X is faster, so prefer
  it when sensitivity trade-offs allow
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_3170
tools:
- name: singleCellHaystack
  role: Cluster-independent DAF prediction via Kullback-Leibler divergence on continuous
    protein expression with protein-derived PCs as cell coordinates; also provides
    plot_compare_ranks, plot_pval_rank, plot_pval_hist for benchmarking
  repo: https://github.com/alexisvdb/singleCellHaystack
- name: singleCellHaystack-py
  role: Python implementation (pip installable from PyPI, version 0.0.5) used with
    scanpy to run hs.haystack and res.top_features on the h5ad atlas
  repo: https://github.com/ddiez/singleCellHaystack-py
- name: scanpy
  role: Loads the h5ad PBMC atlas (read_h5ad) and executes the Wilcoxon rank-sum and
    t-test rank_genes_groups cluster-based baselines on normalized protein data
- name: seurat-wrappers
  role: Seurat ecosystem wrapper collection hosting community method vignettes relevant
    to running singleCellHaystack-style analyses from Seurat
  repo: https://github.com/satijalab/seurat-wrappers
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41598-023-38965-2
    title: A universal tool for predicting differentially active features in single-cell
      and spatial genomics data
  source_content_hash: sha256:7560ebc485053edbea54e4724b0c79333cb0614fbec81a69b0aa23f340107feb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/alexisvdb/singleCellHaystack
  tools:
  - Python
  - scanpy
  - seurat-wrappers
  - singleCellHaystack
  - singleCellHaystack-py
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# differential-activity-rank-comparison

## Summary

Benchmark singleCellHaystack's cluster-independent differentially active feature (DAF) prediction against cluster-based differential expression baselines by comparing feature rankings and p-value distributions, e.g. for protein markers on a CITE-seq PBMC atlas.

## When to use

Use when you need to validate a clustering-independent DAF/DEG method (singleCellHaystack) against cluster-based baselines (Wilcoxon rank-sum, t-test) by comparing rankings of the same features on shared normalized data.

## When NOT to use

- Your analysis goal is a cluster-oriented scRNA-seq workflow — the article notes singleCellHaystack results are not straightforward to incorporate into cluster-oriented analyses
- You need formal multi-condition comparisons (e.g. wild-type vs knockout) — not yet supported by singleCellHaystack
- Spatial transcriptomics where runtime is the priority — SPARK-X is faster, so prefer it when sensitivity trade-offs allow

## Inputs

- Multimodal CITE-seq PBMC h5ad atlas (>200k cells, 228 proteins) with normalized protein expression and 50 protein-based principal components (readable via scanpy read_h5ad)
- Normalized protein expression matrix used identically for haystack and cluster-based baselines

## Outputs

- Top-10 differentially active proteins from singleCellHaystack (res.top_features(n=10))
- Full ranking of all 228 proteins with haystack scores and p-values saved as top_features.csv
- Baseline cluster-based rankings from scanpy rank_genes_groups (Wilcoxon rank-sum and t-test)
- Rank-comparison and p-value diagnostic plots (plot_compare_ranks, plot_pval_rank, plot_pval_hist) reproducing Supplementary Fig. S12

## How to apply

Load the multimodal atlas (e.g. >200k-cell CITE-seq PBMC h5ad with 228 normalized proteins and 50 protein-based PCs) via scanpy's read_h5ad. Run hs.haystack on the normalized protein expression matrix with basis="pca" so the 50 protein-derived PCs serve as the cell coordinate space, deliberately without clustering. Extract the top features (res.top_features(n=10)) and save the full 228-protein ranking with haystack scores and p-values (top_features.csv). Run scanpy's rank_genes_groups with Wilcoxon rank-sum and t-test (Seurat-style) baselines on the same normalized protein data. Compare rankings using singleCellHaystack's plot_compare_ranks, plot_pval_rank, and plot_pval_hist to quantify which proteins (e.g. CD2, CD3-1 high in haystack but not baselines) and which biases (CD133/CD303 for Wilcoxon/t-test) differ, reproducing the Supplementary Fig. S12 agreement pattern.

## Related tools

- **singleCellHaystack** (Cluster-independent DAF prediction via Kullback-Leibler divergence on continuous protein expression with protein-derived PCs as cell coordinates; also provides plot_compare_ranks, plot_pval_rank, plot_pval_hist for benchmarking) — https://github.com/alexisvdb/singleCellHaystack
- **singleCellHaystack-py** (Python implementation (pip installable from PyPI, version 0.0.5) used with scanpy to run hs.haystack and res.top_features on the h5ad atlas) — https://github.com/ddiez/singleCellHaystack-py
- **scanpy** (Loads the h5ad PBMC atlas (read_h5ad) and executes the Wilcoxon rank-sum and t-test rank_genes_groups cluster-based baselines on normalized protein data)
- **seurat-wrappers** (Seurat ecosystem wrapper collection hosting community method vignettes relevant to running singleCellHaystack-style analyses from Seurat) — https://github.com/satijalab/seurat-wrappers

## Examples

```
import scanpy as sc, singleCellHaystack as hs; adata = sc.read_h5ad("data.h5ad"); res = hs.haystack(adata, basis="pca"); res.top_features(n=10)
```

## Evaluation signals

- Full output table contains exactly all 228 proteins each with a haystack score and p-value
- Reproduced Supplementary Fig. S12 pattern: concordant calls for CD64 and CLEC12A across methods
- Ranking differences observed for CD2 and CD3-1 (high scores by singleCellHaystack, not by cluster-based methods)
- Wilcoxon/t-test baselines show bias toward CD133-1/CD133-2 and CD303/CD45-2 that singleCellHaystack does not
- Rank-comparison plots (plot_compare_ranks, plot_pval_rank, plot_pval_hist) generated for all three rankings

## Limitations

- Comparisons between multiple conditions (e.g. wild-type and knockout) cannot be easily conducted
- Results are not straightforward to incorporate into cluster-oriented scRNA-seq analyses
- The Python implementation is in beta: weights.advanced.Q, the seeding grid-point method, and hierarchical clustering for cluster_genes are not yet available
- Relatively longer runtimes than SPARK-X for spatial transcriptomics data

## Evidence

- [methods] Run hs.haystack on the normalized 228-protein expression matrix using basis="pca": [evidence span withheld — anchor status: not_found]
- [readme] Python example invocation from README: "res = hs.haystack(adata, basis="pca")
res.top_features(n=10)"
- [readme] Method is clustering-independent: [evidence span withheld — anchor status: external_doc]
- [discussion] Multi-condition limitation: "One is that comparisons between multiple conditions (e.g., wild-type and knockout) cannot be easily conducted."
- [discussion] Cluster-integration limitation: "Compared to cluster-based DEG prediction approaches, it is not as straightforward to incorporate the results of singleCellHaystack into a cluster-oriented scRNA-seq analysis."
- [readme] Betas missing features: "* `weights.advanced.Q` (formerly known as `use.advanced.sampling`).
* `seeding` method for calculating grid points."
