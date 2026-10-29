---
name: cell-type-classification-accuracy-assessment
description: Use when scRNA-seq clusters have been produced and reference cell-type
  labels exist, and you need ARI plus per-cell-type classification accuracy (e.g.,
  alpha/beta cells) to quantify clustering fidelity.
when_to_use_negative:
- No reference cell-type labels are available for the dataset, so neither ARI nor
  classification accuracy can be computed.
- The method produces hard-only cluster assignments without soft probabilities, precluding
  high-confidence cell selection via maximum assignment probability.
- The goal is batch-effect removal or embedding quality assessment (e.g., regional
  KL divergence of batch mixing) rather than cell-type label recovery.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: DESC
  role: Produces the soft cluster assignment probabilities and cluster labels that
    are evaluated against reference cell-type labels via ARI and maximum-probability
    classification accuracy
  repo: https://github.com/eleozzr/desc
- name: Scanpy
  role: Preprocessing of the expression matrix (log1p normalization, filter_genes_dispersion
    highly variable gene selection, scaling) prior to DESC clustering and accuracy
    evaluation
  repo: https://github.com/theislab/scanpy
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-15851-3
    title: Deep learning enables accurate clustering with batch effect removal in
      single-cell RNA-seq analysis
  source_content_hash: sha256:d51078ee4e51762a78ceb3fb714bc1dc2df912f98a83d316944127f3d0a41550
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/eleozzr/desc
  tools:
  - DESC
  - Keras
  - Scanpy
  - TensorFlow
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# cell-type-classification-accuracy-assessment

## Summary

Assesses how accurately a clustering result recovers known cell types by computing the Adjusted Rand Index (ARI) against reference labels and, using DESC's soft cluster assignment probabilities, the classification accuracy of high-confidence cells for specific cell types (e.g., alpha and beta cells in pancreatic islet data).

## When to use

Use when scRNA-seq clusters have been produced and reference cell-type labels exist, and you need ARI plus per-cell-type classification accuracy (e.g., alpha/beta cells) to quantify clustering fidelity.

## When NOT to use

- No reference cell-type labels are available for the dataset, so neither ARI nor classification accuracy can be computed.
- The method produces hard-only cluster assignments without soft probabilities, precluding high-confidence cell selection via maximum assignment probability.
- The goal is batch-effect removal or embedding quality assessment (e.g., regional KL divergence of batch mixing) rather than cell-type label recovery.

## Inputs

- DESC soft cluster assignment probability matrix (cells x clusters)
- Hard cluster labels per cell from DESC
- Reference cell-type labels for each cell (e.g., alpha, beta islet cells)
- Preprocessed expression matrix (UMI counts normalized to 10,000 per cell, log1p transformed, top ~1000 highly variable genes via filter_genes_dispersion, scaled with max_value=6)

## Outputs

- Adjusted Rand Index (ARI) value comparing clusters to reference labels
- Per-cell-type classification accuracies for high-confidence cells (e.g., alpha = 96.5%, beta = 98.3% on the four-protocol islet data)
- Metrics table (e.g., islet_desc_clustering_metrics.csv) with computed ARI and accuracies

## How to apply

After running DESC (stacked autoencoder pretraining, Louvain-based cluster center initialization, then iterative self-learning minimizing KL divergence between soft Student's t-distribution assignments and the auxiliary target distribution until the fraction of cells changing cluster is below tol = 0.005), retrieve the soft cluster assignment probabilities per cell. Compute the Adjusted Rand Index between the predicted cluster assignments and the reference cell-type labels to obtain a single clustering-accuracy score. Then use the maximum assignment probability to designate high-confidence cells and compute per-cell-type classification accuracies against the known labels — e.g., for the combined four-protocol pancreatic islet data (Fluidigm C1, SMART-seq2, CEL-seq, CEL-seq2), DESC achieved ARI 0.945 with alpha and beta cell classification accuracies of 96.5% and 98.3%. Record all metrics (e.g., in a CSV such as islet_desc_clustering_metrics.csv) so they can be checked against reported values.

## Related tools

- **DESC** (Produces the soft cluster assignment probabilities and cluster labels that are evaluated against reference cell-type labels via ARI and maximum-probability classification accuracy) — https://github.com/eleozzr/desc
- **Scanpy** (Preprocessing of the expression matrix (log1p normalization, filter_genes_dispersion highly variable gene selection, scaling) prior to DESC clustering and accuracy evaluation) — https://github.com/theislab/scanpy

## Examples

```
import desc; adata = desc.train(adata, dims=[512,128,32], tol=0.005); y_pred = adata.obs['desc']; y_prob = adata.uns['probabilities']; from sklearn.metrics import adjusted_rand_score; ari = adjusted_rand_score(labels, y_pred)
```

## Evaluation signals

- Computed ARI reproduces the reported value (0.945 for DESC on the combined four-protocol pancreatic islet data, versus MNN 0.629, scVI 0.696, BERMUDA 0.484, scanorama 0.537).
- Alpha cell classification accuracy reproduces the reported 96.5% and beta cell accuracy reproduces 98.3%.
- The DESC iterative self-learning converged, i.e., the proportion of cells changing cluster assignment between two consecutive steps fell below tol = 0.005.
- High-confidence cell sets are derived by maximum soft assignment probability, and their reference labels agree with assigned cluster identities.
- All computed metrics are recorded in the expected output file (islet_desc_clustering_metrics.csv).

## Limitations

- Accuracies depend on DESC's convergence and on technical differences across batches being smaller than true biological variations; otherwise batch effects may not be removed and accuracies degrade.
- Only cells with high maximum assignment probability can be used for classification accuracy, so low-confidence cells are excluded from per-type accuracy estimates.
- Reproducing paper-identical results requires TensorFlow < 2.0 and desc 2.0.3; newer TensorFlow versions require desc 2.1.1, and versions for Keras/Scanpy are not stated in the article.
- ARI and classification accuracy require trustworthy reference labels; cell-type-specific batch effects (from depleting or enriching certain cell types) can confound the comparison.

## Evidence

- [discussion] Clustering accuracy evaluation using ARI against reference cell-type labels: "we use ARI to compare the performance of different clustering algorithms. Larger values of ARI indicate higher accuracy in clustering."
- [discussion] Iterative self-learning stopping criterion: [evidence span withheld — anchor status: not_found]
- [readme] Soft clustering enables high-confidence cell identification: "DESC also enables soft clustering by assigning cluster-specific probabilities to each cell, which facilitates the identification of cells clustered with high-confidence and interpretation of results."
- [discussion] DESC workflow via KL divergence self-training: [evidence span withheld — anchor status: not_found]
- [readme] Reproducibility requires older TensorFlow: "Because desc depend on `tensorflow`, you should make sure the version of `tensorflow` is lower than `2.0` if you want to get the same results as the results in our paper."
