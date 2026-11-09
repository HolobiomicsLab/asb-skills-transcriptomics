---
name: scrna-seq-label-one-hot-initialization
description: Use when a pre-processed source scRNA-Seq matrix X_src comes with ground-truth
  cluster labels y_src and those annotations should steer NMF dictionary learning
  before transferring knowledge to an unlabeled target dataset.
when_to_use_negative:
- The source dataset has no ground-truth cell-type annotations — the scRNA framework
  then generates labels via NMF clustering instead of one-hot initialization
- You need to cluster the target dataset itself rather than learn a source dictionary
  — use the target-side knowledge-transfer step (learning W_trg given fixed H_src)
  instead
- The source and target gene spaces barely overlap — the paper notes information loss
  is inevitable since only genes present in both datasets can be used
edam_operation: http://edamontology.org/operation_3649
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scRNA (cmd_generate_data.py / scRNA-source.sh)
  role: Python framework whose source-NMF step accepts provided source labels as the
    starting point for NMF (scRNA-source.sh with --labels); falls back to NMF clustering
    when labels are absent
  repo: https://github.com/nicococo/scRNA
- name: scikit-learn NMF (elastic-net regularized)
  role: Performs the regularized non-negative matrix factorization of X_src initialized
    with the one-hot label encoding to learn H_src and W_src
- name: SC3
  role: Downstream consumer of the transfer-learned target dataset; not part of the
    initialization itself but the clustering method the source model ultimately serves
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-019-56911-z
    title: Using transfer learning from prior reference knowledge to improve the clustering
      of single-cell RNA-Seq data
  source_content_hash: sha256:b7f262407cd45592827bc3d8e29893c16ef6cdac638717801db5880077af181d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nicococo/scRNA
  tools:
  - SC3
  - scRNA (cmd_generate_data.py / scRNA-source.sh)
  - scRNA (cmd_generate_data.py)
  - scikit-learn NMF (elastic-net regularized)
  edam_operation: http://edamontology.org/operation_3649
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# scrna-seq-label-one-hot-initialization

## Summary

Initializes the source-data matrix W_src of the elastic-net-regularized NMF in the scRNA transfer-learning pipeline with a one-hot encoding of already-known source cell-type cluster labels, so the learned dictionary H_src (g genes x k clusters) is anchored to the annotated reference cell types rather than discovered from scratch.

## When to use

Use when a pre-processed source scRNA-Seq matrix X_src comes with ground-truth cluster labels y_src and those annotations should steer NMF dictionary learning before transferring knowledge to an unlabeled target dataset.

## When NOT to use

- The source dataset has no ground-truth cell-type annotations — the scRNA framework then generates labels via NMF clustering instead of one-hot initialization
- You need to cluster the target dataset itself rather than learn a source dictionary — use the target-side knowledge-transfer step (learning W_trg given fixed H_src) instead
- The source and target gene spaces barely overlap — the paper notes information loss is inevitable since only genes present in both datasets can be used

## Inputs

- Pre-processed source expression matrix X_src (g genes x n_src cells, cell-filtered, gene-filtered, log-transformed with pseudo-count 1)
- Ground-truth source cluster labels y_src (e.g. .tsv with cluster label and cell id, or labels from the original publication)
- Number of clusters k (e.g. from --cluster-range, default 6,7,8)

## Outputs

- Source NMF model saved as .npz (e.g. src_c8.npz) containing dictionary H_src (g x k) and one-hot-initialized data matrix W_src (k x n_src)
- Model cluster labels after NMF and true/provided labels in .tsv format with corresponding cell ids
- t-SNE plots (.png) per cluster count and a summarizing .png figure

## How to apply

Encode the provided source cluster labels y_src as a one-hot matrix in {0,1}^(k x n_src) with one 1 per cell at its annotated cluster position, and use this matrix as the initial W_src for non-negative matrix factorization of X_src with elastic net regularization (L1/L2 mixing controlled by lambda and penalty multiplier alpha, as in scikit-learn's regularized NMF). Alternating the factorization learns the dictionary H_src (g x k) while the initialization keeps clusters aligned with the reference annotations, which is what the paper cites as the mechanism for 'including the source cell type annotations into the analysis'. If no labels are supplied, the scRNA package falls back to generating labels via NMF clustering, so this skill applies only when annotations exist. Save the resulting source model (H_src together with the label-derived W_src) as a .npz file per cluster count (e.g. src_c8.npz) for the downstream target step.

## Related tools

- **scRNA (cmd_generate_data.py / scRNA-source.sh)** (Python framework whose source-NMF step accepts provided source labels as the starting point for NMF (scRNA-source.sh with --labels); falls back to NMF clustering when labels are absent) — https://github.com/nicococo/scRNA
- **scikit-learn NMF (elastic-net regularized)** (Performs the regularized non-negative matrix factorization of X_src initialized with the one-hot label encoding to learn H_src and W_src)
- **SC3** (Downstream consumer of the transfer-learned target dataset; not part of the initialization itself but the clustering method the source model ultimately serves)

## Examples

```
scRNA-source.sh --input /foo --labels src_labels.tsv --cluster-range 6,7,8 --no-tsne
```

## Evaluation signals

- Model cluster labels after NMF closely match the provided ground-truth source labels (compare .tsv outputs per cluster count)
- W_src is a valid one-hot matrix at initialization: k x n_src with exactly one 1 per column at the annotated cluster position
- H_src has shape (g x k) and both factors are non-negative after the elastic-net-regularized NMF converges
- The saved .npz source model (src_c*.npz) loads and contains H_src usable to reconstruct X_trg via ||X_trg - H_src W||_Fro^2 in the target step
- Per-cluster-count outputs (t-SNE plots, label .tsv files) are generated for every value in --cluster-range (default 6,7,8)

## Limitations

- Requires trustworthy source annotations; the README notes labels 'will be generated via NMF Clustering' if not provided, which changes the semantics of the learned dictionary
- The NMF step can be slow depending on data and cluster range; t-SNE plotting can be disabled with --no-tsne to speed it up
- Only genes shared between source and target can be used downstream, so a small overlap causes loss of vital information (article discussion)

## Evidence

- [methods] One-hot initialization of W_src: [evidence span withheld — anchor status: not_found]
- [readme] Labels as NMF starting point: "You can provide source labels to be used as a starting point for NMF. If not those labels will be generated via NMF Clustering."
- [methods] Role of source annotations in transfer: [evidence span withheld — anchor status: not_found]
- [readme] Source model output artifacts: "Output source model in .npz format for every number of cluster as specified in the --cluster-range argument (default 6,7,8)"
