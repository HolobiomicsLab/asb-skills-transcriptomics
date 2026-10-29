---
name: repeated-clustering-run-aggregation
description: Use when a stochastic deep-clustering pipeline (e.g., scDCC with random
  initialization) yields run-to-run variability and you must quantify whether a modification
  (e.g., adding pairwise constraints) consistently improves a downstream metric such
  as cluster-specificity scores of marker genes.
when_to_use_negative:
- When the clustering algorithm is deterministic or you only have a single clustering
  run — the paired across-run aggregation and signed-rank test require multiple independent
  repeats.
- When you need to compare more than two conditions (with vs. without constraints)
  — the one-sided Wilcoxon signed-rank test as applied here is strictly paired and
  two-condition.
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scDCC
  role: Deep constrained clustering algorithm run ten times with and without soft
    pairwise constraints to produce the repeated cluster assignments being aggregated
  repo: https://github.com/ttgump/scDCC
- name: Python 3.6.8
  role: Runtime environment for scDCC and the specificity-score/Wilcoxon aggregation
    script
- name: PyTorch 1.5.1+cu101
  role: Deep-learning framework in which scDCC.py is executed
- name: Scanpy 1.0.4
  role: Loads the human liver scRNA-seq count matrix used as input to the repeated
    clustering runs
- name: Nvidia Tesla P100
  role: GPU used for the scDCC training runs
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-021-22008-3
    title: Model-based deep embedding for constrained clustering analysis of single
      cell RNA-seq data
  source_content_hash: sha256:b0ef8ad8b0835d98a02f6dbb95d51da662aa1207e13eb5789d0a281b568a5404
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/ttgump/scDCC
  tools:
  - Nvidia Tesla P100
  - PyTorch 1.5.1+cu101
  - Python
  - Python 3.6.8
  - Scanpy
  - Scanpy 1.0.4
  - scDCC
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# repeated-clustering-run-aggregation

## Summary

Aggregate stochastic clustering results by running a clustering algorithm (scDCC) repeatedly with and without pairwise constraints, computing per-gene cluster-specificity scores (DESeq2 Wald statistics) per repeat, averaging across repeats, and testing paired differences with a one-sided Wilcoxon signed-rank test to determine whether constraints systematically improve marker-gene specificity.

## When to use

Use when a stochastic deep-clustering pipeline (e.g., scDCC with random initialization) yields run-to-run variability and you must quantify whether a modification (e.g., adding pairwise constraints) consistently improves a downstream metric such as cluster-specificity scores of marker genes.

## When NOT to use

- When the clustering algorithm is deterministic or you only have a single clustering run — the paired across-run aggregation and signed-rank test require multiple independent repeats.
- When you need to compare more than two conditions (with vs. without constraints) — the one-sided Wilcoxon signed-rank test as applied here is strictly paired and two-condition.

## Inputs

- scRNA-seq count matrix from the scDCC_data figshare deposit (https://figshare.com/articles/dataset/scDCC_data/21563517), loadable with Scanpy 1.0.4
- Ten unconstrained scDCC cluster assignments
- Ten constrained scDCC cluster assignments (constraints generated via --n_pairwise)
- List of 55 uniquely assigned marker genes for the human liver dataset

## Outputs

- Per-gene table of averaged DESeq2 Wald-statistic-based cluster-specificity scores with and without constraints (55 rows, two score columns)
- One-sided Wilcoxon signed-rank test p-value on the paired averaged scores (expected 0.00024)
- CSV file specificity_wilcoxon_results.csv combining the table and p-value

## How to apply

Run the clustering algorithm (scDCC, PyTorch 1.5.1+cu101 on an Nvidia Tesla P100) in unconstrained mode ten times to obtain ten independent cluster assignments on the human liver scRNA-seq count matrix (loaded with Scanpy 1.0.4 in Python 3.6.8). Then run scDCC ten more times with pairwise constraints generated via the --n_pairwise parameter to obtain ten constrained assignments. For each repeat, compute per-gene DESeq2 Wald-statistic-based cluster-specificity scores for the uniquely assigned marker genes (55 in the Fig. 6b analysis) under each clustering, and average the scores across the ten repeats for the with- and without-constraints conditions. Apply a one-sided Wilcoxon signed-rank test to the paired averaged scores to test whether constraints increase marker-gene specificity; in the scDCC human liver analysis this yielded p = 0.00024. Writing the per-gene table (with and without constraints) plus the p-value to specificity_wilcoxon_results.csv provides the reproducible output.

## Related tools

- **scDCC** (Deep constrained clustering algorithm run ten times with and without soft pairwise constraints to produce the repeated cluster assignments being aggregated) — https://github.com/ttgump/scDCC
- **Python 3.6.8** (Runtime environment for scDCC and the specificity-score/Wilcoxon aggregation script)
- **PyTorch 1.5.1+cu101** (Deep-learning framework in which scDCC.py is executed)
- **Scanpy 1.0.4** (Loads the human liver scRNA-seq count matrix used as input to the repeated clustering runs)
- **Nvidia Tesla P100** (GPU used for the scDCC training runs)

## Examples

```
python scDCC_pairwise_Human_liver.py --n_pairwise <N>
```

## Evaluation signals

- One-sided Wilcoxon signed-rank p-value on the paired averaged specificity scores must reproduce the reported value of 0.00024
- Output table must contain exactly 55 rows, one per uniquely assigned marker gene
- Output table must contain both specificity-score columns (with constraints and without constraints)
- Each condition must be based on ten independent clustering repeats, averaged per gene

## Limitations

- The specificity scores depend on DESeq2 Wald statistics, so results inherit assumptions of that differential-expression framework; run-to-run stochasticity means exact per-gene scores may vary across hardware/seeds even if the test conclusion holds.
- The validation threshold (p = 0.00024) is specific to the human liver dataset and the 55 uniquely assigned marker genes; other datasets will produce different p-values.

## Evidence

- [methods] Run scDCC_pairwise_Human_liver.py: "python scDCC_pairwise_Human_liver.py"
- [methods] Pairwise constraint generation parameter: "--n_pairwise: number of pairwise constraints want to generate"
- [methods] Dataset availability for the repeated clustering runs: "Datasets used in the study is available in: https://figshare.com/articles/dataset/scDCC_data/21563517"
- [abstract] Constrained clustering improves performance claim: "Experiments on various scRNA-seq datasets from thousands to tens of thousands of cells show that scDCC can significantly improve clustering performance"
- [methods] Wrapper for human liver dataset analysis: "the wrapper to run scDCC on the human liver dataset (Figure 6)"
- [readme] Clustering as the analysis context: "Clustering is a critical step in single cell-based studies. Most existing methods support unsupervised clustering without the a priori exploitation of any domain knowledge."
