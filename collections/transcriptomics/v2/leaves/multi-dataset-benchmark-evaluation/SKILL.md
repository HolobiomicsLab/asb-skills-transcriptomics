---
name: multi-dataset-benchmark-evaluation
description: Use when a supervised ARI comparison of one scRNA-seq clustering method
  against multiple baselines is needed across a panel of labeled datasets (e.g., Tabula
  Sapiens organs) to establish relative performance.
when_to_use_negative:
- Datasets lack ground-truth cell-type labels, so Adjusted Rand Index cannot be computed
  against a reference
- Datasets are small enough that all baselines (including SIMLR) run without out-of-memory
  errors, so exclusion is unnecessary
- The goal is internal CASCC analysis (attractor identification, DEGs, UMAP) rather
  than cross-method benchmarking
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CASCC
  role: Method under benchmark; run via run.CASCC (featureSelection, predictClusterK,
    kmeansCenterModify) to produce clusteringResults
  repo: https://github.com/LingyiC/CASCC
- name: aricode
  role: Computes the adjusted Rand index between predicted cluster assignments and
    reference cell-type labels
- name: Seurat
  role: Pinned dependency (4.3.0) for the benchmark environment and downstream analyses
    such as FindAllMarkers
- name: cafr
  role: Dependency providing the adaptive attractor algorithm used by CASCC feature
    selection
  repo: https://github.com/weiyi-bitw/cafr
- name: NbClust
  role: Baseline clustering method compared in the large-scale ARI benchmark
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btae283
    title: 'CASCC: a co-expression-assisted single-cell RNA-seq data clustering method'
  source_content_hash: sha256:4cce219c7866a8519493ab24b8440554c810ac0534080a7193238c5375f35a87
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LingyiC/CASCC
  tools:
  - CASCC
  - NbClust
  - R
  - Seurat
  - aricode
  - cafr
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# multi-dataset-benchmark-evaluation

## Summary

Benchmark a clustering method against baselines across many single-cell RNA-seq datasets by computing per-dataset Adjusted Rand Index against ground-truth cell-type labels, as done for CASCC on 19 Tabula Sapiens organ datasets (5,000–30,000 cells each).

## When to use

Use when a supervised ARI comparison of one scRNA-seq clustering method against multiple baselines is needed across a panel of labeled datasets (e.g., Tabula Sapiens organs) to establish relative performance.

## When NOT to use

- Datasets lack ground-truth cell-type labels, so Adjusted Rand Index cannot be computed against a reference
- Datasets are small enough that all baselines (including SIMLR) run without out-of-memory errors, so exclusion is unnecessary
- The goal is internal CASCC analysis (attractor identification, DEGs, UMAP) rather than cross-method benchmarking

## Inputs

- 19 Tabula Sapiens organ scRNA-seq datasets (5,000–30,000 cells each) with ground-truth cell-type labels from LingyiC/CASCC_benchmark
- Normalized, log-transformed expression matrices (inputDataType 'well-normalized')
- Baseline clustering method implementations run under the same feature inputs

## Outputs

- Per-dataset ARI values for the method under test and each baseline
- Consolidated comparison table: tabula_sapiens_ari_table.csv

## How to apply

Load each labeled dataset (here 19 Tabula Sapiens organ scRNA-seq datasets with ground-truth cell-type labels from the LingyiC/CASCC_benchmark repository) and preprocess to normalized, log-transformed expression suitable for CASCC with inputDataType 'well-normalized'. Run the method under test via CASCC::run.CASCC — comprising CASCC.featureSelection, predictClusterK with Ward.D2 linkage and silhouette index, and kmeansCenterModify — and extract res$mainType.output$clusteringResults. Run each baseline method on the same feature inputs, excluding any method documented to fail (SIMLR hits out-of-memory errors on these large datasets). Compute adjusted Rand index between predicted clusters and reference labels with the aricode package, assemble per-dataset ARI values for all methods into one table (tabula_sapiens_ari_table.csv), and validate against the paper's Table S1/Fig. 1c relative comparison using documented session info (R 4.0.5, Seurat 4.3.0).

## Related tools

- **CASCC** (Method under benchmark; run via run.CASCC (featureSelection, predictClusterK, kmeansCenterModify) to produce clusteringResults) — https://github.com/LingyiC/CASCC
- **aricode** (Computes the adjusted Rand index between predicted cluster assignments and reference cell-type labels)
- **Seurat** (Pinned dependency (4.3.0) for the benchmark environment and downstream analyses such as FindAllMarkers)
- **cafr** (Dependency providing the adaptive attractor algorithm used by CASCC feature selection) — https://github.com/weiyi-bitw/cafr
- **NbClust** (Baseline clustering method compared in the large-scale ARI benchmark)

## Examples

```
res <- CASCC::run.CASCC(data); ARI <- aricode::ARI(res$mainType.output$clusteringResults, ground_truth_labels)
```

## Evaluation signals

- ARI values in tabula_sapiens_ari_table.csv reproduce the relative per-dataset method ranking of the paper's Table S1 / Fig. 1c benchmark
- CASCC achieves the best ARI among the five remaining methods across the 19 Tabula Sapiens datasets
- SIMLR is excluded from the large-scale comparison, consistent with its documented out-of-memory failures on 5,000–30,000-cell datasets
- All methods receive the same feature inputs per dataset, ensuring a fair per-dataset comparison
- Session info matches the documented environment: R 4.0.5 and Seurat 4.3.0

## Limitations

- SIMLR cannot be included in large-scale (5,000–30,000 cell) comparisons because it fails with out-of-memory errors
- Benchmark validity depends on the documented environment (R 4.0.5, Seurat 4.3.0); other versions may not reproduce results

## Evidence

- [other] SIMLR was excluded from large-scale comparison due to out-of-memory errors, and on the 19 Tabula Sapiens datasets CASCC achieved the best ARI performance: [evidence span withheld — anchor status: not_found]
- [other] Compute the adjusted Rand index between each method's predicted cluster assignments and the reference cell-type labels using the aricode package: [evidence span withheld — anchor status: not_found]
- [methods] CASCC pipeline call extracting mainType clustering results: "res <- CASCC::run.CASCC(Data_PDAC_peng_2k)"
- [readme] Seurat version pinning in benchmark environment: "remotes::install_version("Seurat", "4.3.0") # dependency"
