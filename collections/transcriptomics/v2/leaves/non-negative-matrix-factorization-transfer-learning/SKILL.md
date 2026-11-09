---
name: non-negative-matrix-factorization-transfer-learning
description: Use when clustering a small target scRNA-Seq dataset where a large well-annotated
  source dataset (shared gene space, non-negative counts) is available and rare cell
  types are hard to resolve with SC3 alone.
when_to_use_negative:
- Source and target share only a small set of transcripts — vital information is lost
  since only genes present in both datasets can be used
- No labeled/annotated source dataset with overlapping cell types is available (e.g.
  a no-overlap setting), where transfer can degrade clustering relative to TargetCluster
- The expression matrix is not non-negative count-level scRNA-Seq data (NMF requires
  non-negativity), or the goal is batch correction/imputation rather than clustering
  improvement
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scRNA (cmd_generate_data.py)
  role: 'Python framework implementing the NMF transfer: data generation, source NMF
    with labels (scRNA-source.sh), and target transfer clustering (scRNA-target.sh)'
  repo: https://github.com/nicococo/scRNA
- name: SC3
  role: Downstream consensus clustering algorithm applied to the modified target dataset
    X_trg_new
- name: scikit-learn NMF
  role: Elastic-net-regularized non-negative matrix factorization used to learn H_src
    and W_src initialized from one-hot-encoded source labels
provenance:
  source_task_ids:
  - task_001
  - task_003
  source_papers:
  - doi: 10.1038/s41598-019-56911-z
    title: Using transfer learning from prior reference knowledge to improve the clustering
      of single-cell RNA-Seq data
  source_content_hash: sha256:a5370e33a97d66556bc0e2f5c37b50fde9830d51928dc9c15d9a97a1c9b71936
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nicococo/scRNA
  tools:
  - SC3
  - scRNA (cmd_generate_data.py)
  - scikit-learn NMF
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# non-negative-matrix-factorization-transfer-learning

## Summary

An NMF-based transfer learning method that adjusts a small target scRNA-Seq dataset using a dictionary learned from a large, annotated source dataset, then feeds the modified target (X_trg_new) to SC3 for improved clustering without concatenating source and target cells.

## When to use

Use when clustering a small target scRNA-Seq dataset where a large well-annotated source dataset (shared gene space, non-negative counts) is available and rare cell types are hard to resolve with SC3 alone.

## When NOT to use

- Source and target share only a small set of transcripts — vital information is lost since only genes present in both datasets can be used
- No labeled/annotated source dataset with overlapping cell types is available (e.g. a no-overlap setting), where transfer can degrade clustering relative to TargetCluster
- The expression matrix is not non-negative count-level scRNA-Seq data (NMF requires non-negativity), or the goal is batch correction/imputation rather than clustering improvement

## Inputs

- Pre-processed source expression matrix X_src (g genes x n_src cells, log-transformed, non-negative) with ground-truth cluster labels y_src
- Pre-processed target expression matrix X_trg (same gene space as X_src, g x n_trg)
- Number of clusters k (or a cluster range, e.g. default 6,7,8 in the scRNA CLI)
- Range of candidate mixture parameter theta values

## Outputs

- Learned source dictionary H_src (g x k) and source data matrix W_src (k x n_src)
- Binary target clustering matrix W'_trg in {0,1}^(k x n_trg)
- Modified target dataset X_trg_new (g x n_trg, non-negative), e.g. saved to 'X_trg_new.npz'
- Predicted target cluster labels after transfer learning (.tsv with cell ids) and Adjusted Rand Index vs ground truth

## How to apply

Preprocess source and target separately (cell filter, ubiquitous/rare gene filter, log-transformation after adding a pseudo-count of 1). Run elastic-net-regularized NMF on the source matrix X_src (g genes x n_src cells), initializing W_src with a one-hot encoding of the source cluster labels, to learn the dictionary H_src (g x k). Fixing H_src, learn the target coefficient matrix W_trg by minimizing ||X_trg - H_src W||_Fro^2, then simplify W_trg to the binary matrix W'_trg by setting ones at column-wise argmax positions. Construct X_trg_new = theta * H_src * W'_trg + (1 - theta) * X_trg, choosing theta automatically via Kernel Target Alignment (KTA) scores over a pre-specified range, and cluster X_trg_new with SC3. The rationale is that injecting the source-derived reconstruction into the target sharpens cluster structure (helping detection of rare cell types) while guaranteeing the target clustering is independent of the source cell types.

## Related tools

- **scRNA (cmd_generate_data.py)** (Python framework implementing the NMF transfer: data generation, source NMF with labels (scRNA-source.sh), and target transfer clustering (scRNA-target.sh)) — https://github.com/nicococo/scRNA
- **SC3** (Downstream consensus clustering algorithm applied to the modified target dataset X_trg_new)
- **scikit-learn NMF** (Elastic-net-regularized non-negative matrix factorization used to learn H_src and W_src initialized from one-hot-encoded source labels)

## Examples

```
pip install git+https://github.com/nicococo/scRNA.git && scRNA-source.sh --no-tsne --cluster-range 6,7,8 && scRNA-target.sh --src-model src_c8.npz --no-tsne
```

## Evaluation signals

- X_trg_new has shape (g x n_trg), is non-negative, and equals the original X_trg when theta = 0
- Each column of W'_trg contains exactly one 1, located at the argmax of the corresponding column of W_trg
- On simulated data (100 repetitions per setting x target size), TransferCluster mean ARI exceeds both TargetCluster and ConcatenateCluster in complete- and incomplete-overlap settings, with non-overlapping 95% confidence intervals
- In the no-overlap setting, TransferCluster CIs overlap TargetCluster (no significant degradation)
- Unsupervised KTA scores used for theta selection correlate with improved clustering accuracy on the target

## Limitations

- Only genes present in both source and target can be used; if they share few transcripts, loss of vital information is inevitable
- The method was evaluated with SC3 as the clustering method; performance with other downstream methods is not guaranteed
- Performance depends on theta selection via KTA; taking the simplified max-based W'_trg performed best and most consistently across investigated scenarios, but other reductions were not always superior
- Batch effect removal (Seurat) and imputation (MAGIC) are optional pre-processing steps, not integrated in the core method

## Evidence

- [methods] NMF of source data regularized with elastic net initialized from source labels: [evidence span withheld — anchor status: not_found]
- [methods] Automatic theta selection via Kernel Target Alignment: "θ is automatically chosen via an unsupervised assessment of the clustering quality through Kernel Target Alignment (KTA) scores76"
- [methods] Simplification of Wtrg to binary membership matrix: [evidence span withheld — anchor status: not_found]
- [readme] Method overview from package README: "Our approach modifies the dataset of interest while incorporating key information from the larger reference dataset via Non-negative Matrix Factorization (NMF)"
- [discussion] Improvement over baselines: "transferring knowledge from a large well-annotated source dataset to a smaller target dataset was not only more successful than applying SC3 to the original target set alone"
