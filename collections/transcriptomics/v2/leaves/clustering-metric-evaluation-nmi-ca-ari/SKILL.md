---
name: clustering-metric-evaluation-nmi-ca-ari
description: Use when comparing clustering results (e.g., scDCC vs. SC3, CIDR, PhenoGraph,
  k-means) on labeled scRNA-seq data, or when verifying that added pairwise constraints
  actually improve NMI/CA/ARI.
when_to_use_negative:
- No ground-truth cell labels exist for the dataset — NMI, CA, and ARI are external
  metrics and cannot be computed
- You need to estimate the number of clusters itself rather than score a fixed-K clustering
  (scDCC provides a separate scDCC_estimating_number_of_clusters module)
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: scDCC
  role: Clustering method whose labels are scored with NMI/CA/ARI against ground truth;
    also provides scDCC_estimating_number_of_clusters
  repo: https://github.com/ttgump/scDCC
- name: Python
  role: Environment (3.6.8) in which metrics are computed and tables assembled
provenance:
  source_task_ids:
  - task_005
  - task_002
  - task_001
  - task_006
  source_papers:
  - doi: 10.1038/s41467-021-22008-3
    title: Model-based deep embedding for constrained clustering analysis of single
      cell RNA-seq data
  source_content_hash: sha256:ad64715809559ac56df6d37a9c4c468afed7b0f7440b2761c98b1a8661d34738
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ttgump/scDCC
  tools:
  - CIDR
  - DCA
  - DEC
  - Nvidia Tesla P100
  - Python
  - Scanpy
  - scDCC
  - scVI
  - scvis
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# clustering-metric-evaluation-nmi-ca-ari

## Summary

Evaluates scRNA-seq clustering results with three external metrics — Normalized Mutual Information (NMI), Clustering Accuracy (CA), and Adjusted Rand Index (ARI) — computed against ground-truth cell labels, as used throughout the scDCC benchmarking study to compare methods and constraint levels.

## When to use

Use when comparing clustering results (e.g., scDCC vs. SC3, CIDR, PhenoGraph, k-means) on labeled scRNA-seq data, or when verifying that added pairwise constraints actually improve NMI/CA/ARI.

## When NOT to use

- No ground-truth cell labels exist for the dataset — NMI, CA, and ARI are external metrics and cannot be computed
- You need to estimate the number of clusters itself rather than score a fixed-K clustering (scDCC provides a separate scDCC_estimating_number_of_clusters module)

## Inputs

- Predicted cluster label vector for each cell (from scDCC or a baseline method such as SC3, CIDR, DCA+k-means, DEC, MPSSC, PCA+k-means, SCVI/scVIS+k-means, SIMLR, Seurat, PhenoGraph, k-means)
- Ground-truth cell label vector (known cell types) for the same cells
- Optional: multiple repeats/splits to average metrics over

## Outputs

- Per-method NMI, CA, and ARI values (means over repeats)
- Aggregated comparison table (e.g., benchmark_comparison_table.csv, cite_pbmc_metrics_table.csv, scdcc_human_liver_metrics.csv)

## How to apply

For each clustering solution, compute NMI, CA (clustering accuracy), and ARI against the ground-truth cell labels on the same set of cells; in the scDCC workflows evaluation is done on the remaining 90% of cells when 10% are held out as the constraint source. Assemble per-method and per-dataset metric values into a single comparison table (e.g., benchmark_comparison_table.csv or cite_pbmc_metrics_table.csv) ordered by method or constraint count, and repeat settings over multiple random splits (ten repeats in the Fig. 2 experiments) recording mean scores. Use the three metrics jointly to judge improvement: in the scDCC paper, most NMI/CA/ARI values exceed 0.8 at 6000 constraints, and constrained scDCC must attain higher NMI/CA/ARI than all competing baselines. For the CITE-seq PBMC analysis, additionally cross-tabulate cluster labels against CD4/CD8 protein-level phenotypes to check biological concordance, since constraint-free scDCC mislabeled CD4+/CD8- cells.

## Related tools

- **scDCC** (Clustering method whose labels are scored with NMI/CA/ARI against ground truth; also provides scDCC_estimating_number_of_clusters) — https://github.com/ttgump/scDCC
- **Python** (Environment (3.6.8) in which metrics are computed and tables assembled)

## Examples

```
python scDCC.py --n_pairwise 6000 --gamma 1 --ml_weight 1 --cl_weight 1  # then compute NMI, CA, ARI vs. true labels and write benchmark_comparison_table.csv
```

## Evaluation signals

- Metrics computed against ground-truth labels on the exact same cell set used for clustering (90% held-out evaluation set when 10% is used for constraints)
- Validation criterion from the paper: scDCC with pairwise constraints attains higher NMI, CA, and ARI than all competing methods on the four small datasets
- Reproduced Figure 2 trend: metrics improve monotonically with constraint count and most NMI/CA/RI values exceed 0.8 at 6000 constraints
- Human liver reference values: scDCC NMI 0.905, CA 0.928, ARI 0.952 vs. k-means-on-ZIFA NMI 0.833, CA 0.928, ARI 0.924
- Ten random splits/repeats per setting with mean scores recorded, so single-split outliers should not drive conclusions

## Limitations

- Metrics depend on the availability and quality of ground-truth labels; they only score label agreement, not biological interpretability of clusters
- Performance varies with the number of pairwise constraints; low constraint counts (near 0) yield lower scores, so evaluation at one constraint level can mislead
- Clustering accuracy (CA) requires an optimal one-to-one mapping between cluster and true labels, which must be handled consistently across methods

## Evidence

- [methods] Metrics against ground truth: [evidence span withheld — anchor status: not_found]
- [results] Benchmark validation criterion: [evidence span withheld — anchor status: not_found]
- [results] Figure 2 expected trend: [evidence span withheld — anchor status: not_found]
- [results] Human liver reference values: [evidence span withheld — anchor status: not_found]
- [results] CITE-seq concordance check: [evidence span withheld — anchor status: not_found]
- [abstract] Paper abstract on clustering performance: "Experiments on various scRNA-seq datasets from thousands to tens of thousands of cells show that scDCC can significantly improve clustering performance"
