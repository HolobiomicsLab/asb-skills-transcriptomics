---
name: adjusted-rand-index-clustering-agreement
description: Use when comparing multiple clustering methods' assignments against ground-truth
  cell-type labels on single-cell RNA-seq data, and per-dataset ARI values must be
  assembled into a benchmark table (e.g. CASCC vs. baselines, SIMLR excluded).
when_to_use_negative:
- When only a single method's clustering is available with no reference labels — ARI
  cannot be computed without a ground-truth labeling to compare against
- When the dataset is small enough that all six methods (including SIMLR) run without
  memory pressure — the SIMLR exclusion rationale (out-of-memory errors on 5,000–30,000-cell
  Tabula Sapiens datasets) does not apply
- When the goal is biological interpretation (e.g. attractor co-expression signatures
  or DEGs) rather than method benchmarking
edam_operation: http://edamontology.org/operation_3744
edam_topics:
- http://edamontology.org/topic_3473
tools:
- name: aricode
  role: R package used to compute the adjusted Rand index between predicted cluster
    assignments and reference cell-type labels
- name: CASCC
  role: Clustering method under evaluation; its res$mainType.output$clusteringResults
    assignments are scored by ARI
  repo: https://github.com/LingyiC/CASCC
- name: Seurat 4.3.0
  role: Pinned dependency of the CASCC environment used for the benchmark runs
  repo: https://github.com/LingyiC/CASCC
- name: cafr
  role: Dependency providing the adaptive attractor algorithm underlying CASCC feature
    selection prior to clustering
  repo: https://github.com/weiyi-bitw/cafr
- name: CASCC_benchmark
  role: Repository supplying the 19 Tabula Sapiens organ datasets with ground-truth
    labels and benchmark code
  repo: https://github.com/LingyiC/CASCC_benchmark
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btae283
    title: 'CASCC: a co-expression-assisted single-cell RNA-seq data clustering method'
  source_content_hash: sha256:d82b086ba5d8560281e58b4c6713eac0b04eca461e8d1b6b01a5d01595e6a824
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LingyiC/CASCC
  tools:
  - BBKNN
  - CASCC
  - CASCC_benchmark
  - Harmony
  - NbClust
  - R
  - Scanorama
  - Seurat
  - Seurat 4.3.0
  - aricode
  - cafr
  - scANVI
  - scMerge2
  - scVI
  edam_operation: http://edamontology.org/operation_3744
  edam_topics:
  - http://edamontology.org/topic_3473
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# adjusted-rand-index-clustering-agreement

## Summary

Compute the adjusted Rand index (ARI) between predicted cluster assignments and reference cell-type labels to benchmark clustering methods on scRNA-seq datasets, as done in the CASCC paper to compare CASCC against five baselines across 19 Tabula Sapiens organ datasets (5,000–30,000 cells each).

## When to use

Use when comparing multiple clustering methods' assignments against ground-truth cell-type labels on single-cell RNA-seq data, and per-dataset ARI values must be assembled into a benchmark table (e.g. CASCC vs. baselines, SIMLR excluded).

## When NOT to use

- When only a single method's clustering is available with no reference labels — ARI cannot be computed without a ground-truth labeling to compare against
- When the dataset is small enough that all six methods (including SIMLR) run without memory pressure — the SIMLR exclusion rationale (out-of-memory errors on 5,000–30,000-cell Tabula Sapiens datasets) does not apply
- When the goal is biological interpretation (e.g. attractor co-expression signatures or DEGs) rather than method benchmarking

## Inputs

- Normalized, log-transformed scRNA-seq expression matrices for 19 Tabula Sapiens organ datasets (5,000–30,000 cells) from the LingyiC/CASCC_benchmark repository
- Ground-truth cell-type label vectors for each dataset
- Predicted cluster assignment vectors from CASCC (res$mainType.output$clusteringResults) and from five baseline clustering methods run under the same feature inputs

## Outputs

- Per-dataset adjusted Rand index values for CASCC and each baseline method
- A combined benchmark table saved as tabula_sapiens_ari_table.csv

## How to apply

Run each clustering method (CASCC via run.CASCC with CASCC.featureSelection on 'well-normalized' log-transformed data, predictClusterK with ward.D2 linkage and silhouette index, then kmeansCenterModify; plus the five baseline methods under the same feature inputs) on each dataset, extracting res$mainType.output$clusteringResults for CASCC. Convert predicted cluster assignments and reference labels to integer vectors and compute the adjusted Rand index between them with the aricode package. Repeat per dataset (19 Tabula Sapiens organ datasets, 5,000–30,000 cells) and per method, excluding SIMLR because it was documented to fail with out-of-memory errors on these large datasets. Assemble the per-dataset ARI values for CASCC and all baselines into a single table (tabula_sapiens_ari_table.csv) and compare against the paper's Table S1/Fig. 1c relative method ranking. Use the documented session (R 4.0.5, Seurat 4.3.0) for reproducibility.

## Related tools

- **aricode** (R package used to compute the adjusted Rand index between predicted cluster assignments and reference cell-type labels)
- **CASCC** (Clustering method under evaluation; its res$mainType.output$clusteringResults assignments are scored by ARI) — https://github.com/LingyiC/CASCC
- **Seurat 4.3.0** (Pinned dependency of the CASCC environment used for the benchmark runs) — https://github.com/LingyiC/CASCC
- **cafr** (Dependency providing the adaptive attractor algorithm underlying CASCC feature selection prior to clustering) — https://github.com/weiyi-bitw/cafr
- **CASCC_benchmark** (Repository supplying the 19 Tabula Sapiens organ datasets with ground-truth labels and benchmark code) — https://github.com/LingyiC/CASCC_benchmark

## Examples

```
ari <- aricode::ARI(res$mainType.output$clusteringResults, true_labels)
```

## Evaluation signals

- The assembled tabula_sapiens_ari_table.csv reproduces the relative per-dataset method comparison reported in the paper's Table S1/Fig. 1c benchmark
- CASCC achieves the best ARI among the remaining five methods on the 19 Tabula Sapiens datasets (SIMLR excluded)
- ARI values fall in the expected range (ARI of 0 for chance-level agreement, 1.0 for perfect agreement with ground truth)
- Session reproducibility is confirmed with the documented environment (R 4.0.5, Seurat 4.3.0)
- Each dataset's ARI is computed on the same feature inputs across all methods (SIMLR excluded only on documented out-of-memory grounds)

## Limitations

- SIMLR must be excluded from the large-scale comparison because it fails with out-of-memory errors on the 5,000–30,000-cell Tabula Sapiens datasets
- ARI benchmarking requires ground-truth labels; it measures agreement with reference cell types, not biological discovery
- Results depend on the pinned environment (R 4.0.5, Seurat 4.3.0); version drift may change feature selection and clustering outcomes

## Evidence

- [methods] ARI computed with aricode on CASCC vs baselines: [evidence span withheld — anchor status: not_found]
- [methods] CASCC pipeline components: "res <- CASCC::run.CASCC(Data_PDAC_peng_2k)"
- [methods] predictClusterK parameters: "res.predictK <- predictClusterK(data, fs.res, method = "ward.D2", index = "silhouette", min.nc_fix = FALSE)"
- [readme] Benchmark repo reference in README: "https://github.com/LingyiC/CASCC_benchmark"
- [methods] Feature selection input type: "fs.res <- CASCC.featureSelection(Data_PDAC_peng_2k, inputDataType = "well-normalized""
