---
name: deep-embedded-clustering-training
description: Use when clustering combined scRNA-seq data from multiple library-preparation
  protocols (e.g., Fluidigm C1, SMART-seq2, CEL-seq, CEL-seq2) where batch effects
  confound cell-type identification and ARI against reference labels is the target
  metric.
when_to_use_negative:
- When technical differences across batches are larger than true biological variations
  — DESC's batch-effect removal relies on this assumption
- When hard clustering without confidence estimates suffices and a simpler method
  (e.g., Louvain alone) would meet the accuracy requirement
- When batch information must be explicitly modeled and corrected pairwise (e.g.,
  MNN/Scanorama-style workflows are required by the analysis design)
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: DESC
  role: Unsupervised deep embedding algorithm trained to cluster cells and remove
    batch effects via iterative self-learning with KL divergence loss
  repo: https://github.com/eleozzr/desc
- name: Scanpy
  role: 'Preprocessing: cell-level normalization (log1p), highly variable gene selection
    via filter_genes_dispersion, and gene-level scaling'
  repo: https://github.com/theislab/scanpy
- name: Keras
  role: Backend used to train the DESC stacked autoencoder and iterative clustering
    model
- name: TensorFlow
  role: Deep learning backend for DESC; use tensorflow 1.x with desc 2.0.3 to reproduce
    paper results
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-15851-3
    title: Deep learning enables accurate clustering with batch effect removal in
      single-cell RNA-seq analysis
  source_content_hash: sha256:25f97fe2c380e5d9b60f170bc13ec72057d4a1334b832dab2851272933e63aa2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eleozzr/desc
  tools:
  - DESC
  - Keras
  - Scanpy
  - TensorFlow
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# deep-embedded-clustering-training

## Summary

Train DESC, an unsupervised deep embedding algorithm that clusters scRNA-seq data by iteratively optimizing a clustering objective, jointly removing batch effects and producing soft cluster assignment probabilities for high-confidence cell-type classification.

## When to use

Use when clustering combined scRNA-seq data from multiple library-preparation protocols (e.g., Fluidigm C1, SMART-seq2, CEL-seq, CEL-seq2) where batch effects confound cell-type identification and ARI against reference labels is the target metric.

## When NOT to use

- When technical differences across batches are larger than true biological variations — DESC's batch-effect removal relies on this assumption
- When hard clustering without confidence estimates suffices and a simpler method (e.g., Louvain alone) would meet the accuracy requirement
- When batch information must be explicitly modeled and corrected pairwise (e.g., MNN/Scanorama-style workflows are required by the analysis design)

## Inputs

- Combined scRNA-seq UMI count matrices from multiple library-preparation protocols (e.g., Fluidigm C1, SMART-seq2, CEL-seq, CEL-seq2)
- Reference cell-type labels for the same cells

## Outputs

- Cluster assignment for each cell
- Soft cluster assignment probability matrix (per-cell probabilities over clusters)
- Adjusted Rand Index (ARI) against reference labels
- Per-cell-type classification accuracies for high-confidence assignments (e.g., alpha 96.5%, beta 98.3%)
- Metrics table such as islet_desc_clustering_metrics.csv

## How to apply

Preprocess count matrices with Scanpy: divide UMI counts per gene by total UMIs per cell, multiply by 10,000, apply sc.pp.log1p, select 1000 highly variable genes with sc.pp.filter_genes_dispersion, then standardize with sc.pp.scale(max_value=6). Train DESC (desc 2.0.3 on TensorFlow 1.x/Keras) starting with stacked autoencoder pretraining to a low-dimensional embedding, initialize cluster centers using Louvain clustering in the bottleneck-layer feature space, then iteratively minimize KL divergence between soft Student's t-distribution assignments q_i and the auxiliary target distribution p_i, stopping when the fraction of cells changing cluster falls below tol = 0.005. Evaluate clustering accuracy with the Adjusted Rand Index against reference cell-type labels, and use the maximum soft assignment probability to retain high-confidence cells for per-type classification accuracy (e.g., alpha and beta islet cells).

## Related tools

- **DESC** (Unsupervised deep embedding algorithm trained to cluster cells and remove batch effects via iterative self-learning with KL divergence loss) — https://github.com/eleozzr/desc
- **Scanpy** (Preprocessing: cell-level normalization (log1p), highly variable gene selection via filter_genes_dispersion, and gene-level scaling) — https://github.com/theislab/scanpy
- **Keras** (Backend used to train the DESC stacked autoencoder and iterative clustering model)
- **TensorFlow** (Deep learning backend for DESC; use tensorflow 1.x with desc 2.0.3 to reproduce paper results)

## Examples

```
sc.pp.filter_genes_dispersion(adata, n_top_genes=1000); sc.pp.scale(adata, max_value=6)
```

## Evaluation signals

- Computed ARI on the combined four-protocol pancreatic islet data reproduces the reported 0.945 (vs MNN 0.629, scVI 0.696, BERMUDA 0.484, scanorama 0.537)
- Alpha and beta cell classification accuracies from high-confidence assignments reproduce 96.5% and 98.3%
- Training converges: proportion of cells changing cluster assignment between consecutive steps drops below tol = 0.005
- Preprocessing output matches spec: log1p-normalized values, n_top_genes=1000 highly variable genes, scaled with max_value=6
- All values recorded in islet_desc_clustering_metrics.csv

## Limitations

- DESC gradually removes batch effects only as long as technical differences across batches are smaller than true biological variations
- To reproduce the paper's results, TensorFlow must be below 2.0 with desc 2.0.3; Python 3.5.x or 3.6.x is required per the README
- The Student's t distribution kernel was found more robust than the Gaussian kernel, but supplementary details (Supplementary Notes 2-3, Tables 2-3 hyperparameters) are not provided in the main text
- Order-independent batch correction is implicit; no explicit batch labels are passed, so cell-type-specific batch effects may not be fully removed

## Evidence

- [abstract] We present DESC, an unsupervised deep embedding algorithm that clusters scRNA-seq data by iteratively optimizing a clustering objective function: "We present DESC, an unsupervised deep embedding algorithm that clusters scRNA-seq data by iteratively optimizing a clustering objective function"
- [abstract] Through iterative self-learning, DESC gradually removes batch effects, as long as technical differences across batches are smaller than true biological variations: "Through iterative self-learning, DESC gradually removes batch effects, as long as technical differences across batches are smaller than true biological variations"
- [methods] sc.pp.filter_genes_dispersion(adata,n_top_genes=1000): "sc.pp.filter_genes_dispersion(adata,n_top_genes=1000)"
- [methods] sc.pp.scale(adata,max_value=6): "sc.pp.scale(adata,max_value=6)"
- [discussion] This iterative procedure stops when the proportion of cells that changes cluster assignment between two consecutive steps is less than tol.: [evidence span withheld — anchor status: not_found]
- [readme] Because desc depend on `tensorflow`, you should make sure the version of `tensorflow` is lower than `2.0` if you want to get the same results as the results in our paper.: "Because desc depend on `tensorflow`, you should make sure the version of `tensorflow` is lower than `2.0` if you want to get the same results as the results in our paper."
