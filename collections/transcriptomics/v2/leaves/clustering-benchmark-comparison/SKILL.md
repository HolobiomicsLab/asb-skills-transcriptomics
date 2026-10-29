---
name: clustering-benchmark-comparison
description: Use when a single-cell RNA-seq clustering method must be statistically
  compared against established baselines (e.g., Seurat, SC3, SIMLR) over a multi-dataset
  benchmark using ARI significance testing and embedding-quality metrics such as ASW.
when_to_use_negative:
- The input is a single exploratory dataset with no baseline ARI scores — there is
  nothing to pair for the Wilcoxon signed-rank test.
- The analysis goal is running CASCC clustering itself (run.CASCC, predictClusterK)
  rather than comparing method performance against baselines.
- The data are not single-cell RNA-seq expression matrices; the benchmark datasets
  and ASW-on-UMAP evaluation assume scRNA-seq inputs.
edam_operation: http://edamontology.org/operation_2944
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CASCC
  role: Clustering method under evaluation; provides run.CASCC and predictClusterK
    (method = 'ward.D2', index = 'silhouette') and the res$res.predictK$adata UMAP
    embeddings used for ASW computation
  repo: https://github.com/LingyiC/CASCC
- name: Seurat 4.3.0
  role: Baseline clustering method (Seurat V4) and source of DimPlot visualization
    and FindAllMarkers for the benchmarked 2D embeddings
- name: cafr
  role: Dependency of CASCC providing the adaptive attractor co-expression algorithm
    underlying the co-expression features being benchmarked
  repo: https://github.com/weiyi-bitw/cafr
- name: CASCC_benchmark
  role: Repository holding the 15-dataset benchmark code and per-dataset ARI score
    matrices used as input to the significance testing
  repo: https://github.com/LingyiC/CASCC_benchmark
provenance:
  source_task_ids:
  - task_002
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btae283
    title: 'CASCC: a co-expression-assisted single-cell RNA-seq data clustering method'
  source_content_hash: sha256:e3f98b1d0782eeebfa395cd4e91c5547d2c042cfde3136abc2dea401713a1747
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LingyiC/CASCC
  tools:
  - CASCC
  - CASCC_benchmark
  - NbClust
  - R
  - Seurat 4.3.0
  - cafr
  - ggplot2
  edam_operation: http://edamontology.org/operation_2944
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 16
---

# clustering-benchmark-comparison

## Summary

Benchmarking skill for the CASCC co-expression-assisted scRNA-seq clustering pipeline: compare CASCC's ARI against six baseline methods (CIDR, RaceID3, SC3, Seurat V4, SIMLR, TSCAN) across the 15-dataset Table S1 benchmark using paired Wilcoxon signed-rank tests, and evaluate 2D embedding quality via average silhouette width (ASW).

## When to use

Use when a single-cell RNA-seq clustering method must be statistically compared against established baselines (e.g., Seurat, SC3, SIMLR) over a multi-dataset benchmark using ARI significance testing and embedding-quality metrics such as ASW.

## When NOT to use

- The input is a single exploratory dataset with no baseline ARI scores — there is nothing to pair for the Wilcoxon signed-rank test.
- The analysis goal is running CASCC clustering itself (run.CASCC, predictClusterK) rather than comparing method performance against baselines.
- The data are not single-cell RNA-seq expression matrices; the benchmark datasets and ASW-on-UMAP evaluation assume scRNA-seq inputs.

## Inputs

- 15 benchmark single-cell RNA-seq datasets from the Table S1 benchmark (available via LingyiC/CASCC_benchmark)
- Per-dataset ARI score matrix containing CASCC and the six baseline methods (CIDR, RaceID3, SC3, Seurat V4, SIMLR, TSCAN)
- 2D UMAP cell embeddings per method per dataset (e.g., CASCC's res$res.predictK$adata feature-selected UMAP)

## Outputs

- supp_table_S4_wilcoxon_significance.csv — one row per CASCC-vs-baseline comparison with Wilcoxon test statistic and p-value
- asw_benchmark_results.csv — method-by-dataset table of average silhouette widths
- supplementary_fig_S2.png — ggplot2 figure comparing per-method ASW distributions across the 15 datasets

## How to apply

Load the per-dataset ARI score matrix for CASCC and the six baselines from the LingyiC/CASCC_benchmark results over the Table S1 benchmark datasets, then for each baseline run a paired Wilcoxon signed-rank test in R with stats::wilcox.test(paired = TRUE) comparing the CASCC ARI vector against the baseline's ARI vector across the same datasets; collect statistic and p-value per comparison into supp_table_S4_wilcoxon_significance.csv. For embedding quality, run the CASCC pipeline (run.CASCC) and comparison methods on the 15 benchmark scRNA-seq datasets in an R 4.0.5 session with Seurat 4.3.0 and cafr installed, extract 2D UMAP cell embeddings (exposed as res$res.predictK$adata, the feature-selected UMAP visualized with Seurat DimPlot), and compute per-cluster silhouette widths averaged into an ASW per method per dataset — consistent with the silhouette index used by predictClusterK (method = 'ward.D2', index = 'silhouette'). Aggregate ASW values into a method-by-dataset table (asw_benchmark_results.csv) and plot per-method ASW distributions across the 15 datasets with ggplot2 (supplementary_fig_S2.png). The rationale is that ARI deltas alone do not demonstrate statistical significance, and ASW quantifies cluster separation and cohesion of the 2D t-SNE/UMAP embeddings.

## Related tools

- **CASCC** (Clustering method under evaluation; provides run.CASCC and predictClusterK (method = 'ward.D2', index = 'silhouette') and the res$res.predictK$adata UMAP embeddings used for ASW computation) — https://github.com/LingyiC/CASCC
- **Seurat 4.3.0** (Baseline clustering method (Seurat V4) and source of DimPlot visualization and FindAllMarkers for the benchmarked 2D embeddings)
- **cafr** (Dependency of CASCC providing the adaptive attractor co-expression algorithm underlying the co-expression features being benchmarked) — https://github.com/weiyi-bitw/cafr
- **CASCC_benchmark** (Repository holding the 15-dataset benchmark code and per-dataset ARI score matrices used as input to the significance testing) — https://github.com/LingyiC/CASCC_benchmark

## Examples

```
res <- CASCC::run.CASCC(Data_PDAC_peng_2k); adata <- res$res.predictK$adata
```

## Evaluation signals

- supp_table_S4_wilcoxon_significance.csv contains a test statistic and p-value for all six CASCC-vs-baseline comparisons; any row missing a statistic or p-value fails
- The Wilcoxon results reproduce the significance reported in the paper's Supplementary Table S4 (CASCC ARI differences versus all six methods statistically significant)
- The ASW table contains values for every method-dataset combination across the 15 Table S1 benchmark datasets
- The generated ASW figure matches the pattern in Supplementary Fig. S2, where CASCC and Seurat achieve much higher ASW values than the other clustering methods on the 2D embeddings
- R environment matches the benchmark session (R 4.0.5 with Seurat 4.3.0) when regenerating embeddings

## Limitations

- The benchmark covers only the 15 Table S1 evaluation datasets; significance conclusions are specific to this benchmark and the six named baselines.
- The CASCC pipeline is version-sensitive: the README instructs pinning Seurat to 4.3.0 via remotes::install_version and installing cafr from GitHub, so deviating from R 4.0.5 / Seurat 4.3.0 may not reproduce the deposited benchmark results.
- ASW evaluation depends on CASCC-internal outputs (res$res.predictK$adata feature-selected UMAP); baselines without comparable 2D embeddings cannot be scored the same way.

## Evidence

- [intro] CASCC, a clustering method designed to improve biological accuracy using gene co-expression features identified using an unsupervised adaptive attractor algorithm.: "CASCC, a clustering method designed to improve biological accuracy using gene co-expression features identified using an unsupervised adaptive attractor algorithm."
- [readme] remotes::install_version("Seurat", "4.3.0") # dependency: "remotes::install_version("Seurat", "4.3.0") # dependency"
- [methods] res.predictK <- predictClusterK(data, fs.res, method = "ward.D2", index = "silhouette": "res.predictK <- predictClusterK(data, fs.res, method = "ward.D2", index = "silhouette""
- [methods] R version 4.0.5 (2021-03-31): "R version 4.0.5 (2021-03-31)"
- [methods] The analysis identifies the presence of eight distinct clusters, each corresponding to a specific attractor: "The analysis identifies the presence of eight distinct clusters, each corresponding to a specific attractor"
