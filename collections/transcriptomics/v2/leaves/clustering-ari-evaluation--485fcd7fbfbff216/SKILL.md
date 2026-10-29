---
name: clustering-ari-evaluation
description: Use when a clustering result on single-cell RNA-seq data must be quantitatively
  validated against known cell-type labels, e.g., comparing DESC vs MNN/scVI/BERMUDA/Scanorama
  on the combined Fluidigm C1, SMART-seq2, CEL-seq, and CEL-seq2 islet data.
when_to_use_negative:
- No reference cell-type labels are available for the dataset, so ARI cannot be computed
  against ground truth
- The goal is batch-effect removal assessment via regional KL divergence of batch
  mixing rather than clustering accuracy
- Input is raw scRNA-seq count data that has not yet been clustered — this skill evaluates
  clustering, it does not produce clusters
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: DESC
  role: Produces soft cluster assignment probabilities that are converted into labels
    and high-confidence assignments for ARI and classification-accuracy evaluation
  repo: https://github.com/eleozzr/desc
- name: Scanpy
  role: Preprocessing (log1p, filter_genes_dispersion with n_top_genes=1000, scale
    max_value=6) of the count matrices before clustering and evaluation
  repo: https://github.com/scverse/scanpy
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-15851-3
    title: Deep learning enables accurate clustering with batch effect removal in
      single-cell RNA-seq analysis
  source_content_hash: sha256:d292745346b5fd3d9da01a9efee87fb42c70379df126b8e0f6bacdd42b17a08e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eleozzr/desc
  tools:
  - DESC
  - Keras
  - Scanpy
  - TensorFlow
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# clustering-ari-evaluation

## Summary

Evaluates scRNA-seq clustering performance by computing the Adjusted Rand Index (ARI) against reference cell-type labels, and, when soft cluster probabilities are available, high-confidence cell-type classification accuracy (e.g., alpha and beta cells in the four-protocol pancreatic islet data).

## When to use

Use when a clustering result on single-cell RNA-seq data must be quantitatively validated against known cell-type labels, e.g., comparing DESC vs MNN/scVI/BERMUDA/Scanorama on the combined Fluidigm C1, SMART-seq2, CEL-seq, and CEL-seq2 islet data.

## When NOT to use

- No reference cell-type labels are available for the dataset, so ARI cannot be computed against ground truth
- The goal is batch-effect removal assessment via regional KL divergence of batch mixing rather than clustering accuracy
- Input is raw scRNA-seq count data that has not yet been clustered — this skill evaluates clustering, it does not produce clusters

## Inputs

- Cluster assignment labels per cell from the clustering method (e.g., DESC soft assignment probabilities)
- Reference cell-type labels for the same cells (e.g., alpha, beta cells in human pancreatic islet scRNA-seq data)

## Outputs

- Adjusted Rand Index (ARI) comparing clusters to reference labels
- High-confidence cell-type classification accuracies (e.g., alpha 96.5%, beta 98.3%)
- Metrics table, e.g., islet_desc_clustering_metrics.csv

## How to apply

After preprocessing with Scanpy (UMI counts divided by total UMIs per cell, multiplied by 10,000, log1p transformed; 1000 highly variable genes via sc.pp.filter_genes_dispersion; gene standardization with sc.pp.scale, max_value=6), run the clustering method (e.g., DESC with Louvain-initialized cluster centers and iterative KL-divergence self-learning, stopping when the fraction of cells changing cluster falls below tol = 0.005). Compare the resulting cluster assignments to reference cell-type labels using ARI — larger values indicate higher clustering accuracy; DESC reached 0.945 on the combined four-protocol islet data versus MNN (0.629), scVI (0.696), BERMUDA (0.484), and Scanorama (0.537). For soft clustering (DESC), use the maximum assignment probability to designate high-confidence cells and compute classification accuracies against known labels, e.g., alpha 96.5% and beta 98.3%. Record all metrics (ARI, per-cell-type accuracies) in a results table such as islet_desc_clustering_metrics.csv.

## Related tools

- **DESC** (Produces soft cluster assignment probabilities that are converted into labels and high-confidence assignments for ARI and classification-accuracy evaluation) — https://github.com/eleozzr/desc
- **Scanpy** (Preprocessing (log1p, filter_genes_dispersion with n_top_genes=1000, scale max_value=6) of the count matrices before clustering and evaluation) — https://github.com/scverse/scanpy

## Examples

```
sc.pp.log1p(adata); sc.pp.filter_genes_dispersion(adata,n_top_genes=1000); sc.pp.scale(adata,max_value=6)
```

## Evaluation signals

- Computed ARI on the combined four-protocol pancreatic islet data reproduces the reported 0.945 for DESC (vs MNN 0.629, scVI 0.696, BERMUDA 0.484, Scanorama 0.537)
- Alpha and beta cell classification accuracies reproduce the reported 96.5% and 98.3%
- DESC iterative self-learning has converged: fraction of cells changing cluster assignment between consecutive steps is below tol = 0.005
- All computed values are recorded in islet_desc_clustering_metrics.csv
- ARI is strictly a larger-is-better metric — verify the implementation matches the reference-label granularity used in the paper

## Limitations

- DESC stops by tol = 0.005; results can vary with initialization and TensorFlow version (README notes TensorFlow < 2.0 is required to reproduce paper results)
- ARI requires reference labels, which may not reflect all biologically meaningful populations
- High-confidence classification accuracy depends on the maximum soft assignment probability threshold and only covers cells assigned with high confidence

## Evidence

- [discussion] we use ARI to compare the performance of different clustering algorithms. Larger values of ARI indicate higher accuracy in clustering.: "we use ARI to compare the performance of different clustering algorithms. Larger values of ARI indicate higher accuracy in clustering."
- [results] On the combined four-protocol pancreatic islet data, DESC yields the highest ARI of 0.945, much higher than MNN (0.629), scVI (0.696), BERMUDA (0.484), and scanorama (0.537), with alpha and beta cell classification accuracies of 96.5% and 98.3%.: [evidence span withheld — anchor status: not_found]
- [discussion] This iterative procedure stops when the proportion of cells that changes cluster assignment between two consecutive steps is less than tol.: [evidence span withheld — anchor status: not_found]
- [readme] DESC also enables soft clustering by assigning cluster-specific probabilities to each cell, which facilitates the identification of cells clustered with high-confidence and interpretation of results.: "DESC also enables soft clustering by assigning cluster-specific probabilities to each cell, which facilitates the identification of cells clustered with high-confidence"
- [methods] sc.pp.filter_genes_dispersion(adata,n_top_genes=1000): "sc.pp.filter_genes_dispersion(adata,n_top_genes=1000)"
