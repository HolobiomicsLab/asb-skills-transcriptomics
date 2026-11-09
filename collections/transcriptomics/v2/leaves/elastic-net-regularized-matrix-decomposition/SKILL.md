---
name: elastic-net-regularized-matrix-decomposition
description: Use when you have a large labeled source scRNA-Seq expression matrix
  (genes x cells) with ground-truth cluster labels and need a gene-level dictionary
  H_src to adjust a smaller target dataset for clustering.
when_to_use_negative:
- When source and target datasets share only a small set of transcripts — the article
  notes that 'only the set of genes present in both datasets can be used', causing
  inevitable loss of vital information
- When the goal is simply to cluster the target dataset without any prior reference
  knowledge — plain SC3 on the original target is the correct baseline
- When the source data lack cell type annotations and NMF-derived labels would be
  unreliable for initializing the dictionary
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: scRNA (cmd_generate_data.py)
  role: Python framework providing data generation, elastic-net-regularized NMF of
    the source data, and the transfer workflow
  repo: https://github.com/nicococo/scRNA
- name: scikit-learn NMF
  role: Elastic-net regularized NMF implementation cited in the paper for learning
    H_src and W_src
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-019-56911-z
    title: Using transfer learning from prior reference knowledge to improve the clustering
      of single-cell RNA-Seq data
  source_content_hash: sha256:898da5a073345155851bee1e9df0c4de921e724137abf390307cf7e0ba557260
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nicococo/scRNA
  tools:
  - scRNA (cmd_generate_data.py)
  - scikit-learn NMF
  - scikit-learn NMF (elastic-net regularized, as cited in the paper)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# elastic-net-regularized-matrix-decomposition

## Summary

Non-negative matrix factorization (NMF) of a source scRNA-Seq expression matrix with elastic net (L1/L2) regularization, initialized with one-hot-encoded source cell type labels, yielding the dictionary H_src used to transfer knowledge to a target dataset via the scRNA Python framework.

## When to use

Use when you have a large labeled source scRNA-Seq expression matrix (genes x cells) with ground-truth cluster labels and need a gene-level dictionary H_src to adjust a smaller target dataset for clustering.

## When NOT to use

- When source and target datasets share only a small set of transcripts — the article notes that 'only the set of genes present in both datasets can be used', causing inevitable loss of vital information
- When the goal is simply to cluster the target dataset without any prior reference knowledge — plain SC3 on the original target is the correct baseline
- When the source data lack cell type annotations and NMF-derived labels would be unreliable for initializing the dictionary

## Inputs

- Pre-processed source expression matrix X_src (g genes x n_src cells, non-negative)
- Ground-truth source cluster labels y_src (one-hot encoded for NMF initialization; optional)
- Target expression matrix X_trg in the same gene space
- NMF regularization parameters: lambda (L1/L2 mixing), alpha (penalty multiplier)
- Cluster range (default 6,7,8)

## Outputs

- Source dictionary H_src (g x k)
- Source data matrix W_src (k x n_src)
- Source model per cluster count in .npz format (e.g. src_c8.npz)
- NMF model cluster labels in .tsv format with cell ids
- t-SNE plots (.png) per cluster count and a summarizing .png figure

## How to apply

Load the pre-processed source matrix X_src (g genes x n_src cells), its cluster labels y_src, and the target matrix X_trg in the same gene space (e.g. simulated with scRNA/cmd_generate_data.py). Run elastic-net-regularized NMF (L1/L2 mixing controlled by lambda with penalty multiplier alpha), initializing W_src with a one-hot encoding of the given labels, to learn H_src (g x k) and W_src (k x n_src); if labels are not supplied, the scRNA pipeline generates them via NMF clustering. Fix H_src and learn the target coefficient matrix W_trg by minimizing the squared Frobenius norm ||X_trg - H_src W||_Fro^2. The learned model is saved per cluster count (default --cluster-range 6,7,8) as .npz files (e.g. src_c8.npz) for later use by the target-transfer step. The rationale is that the regularized dictionary captures source cell-type structure so that the target data can be reconstructed and modified (via the theta-mixed X_trg_new) without concatenating the two datasets.

## Related tools

- **scRNA (cmd_generate_data.py)** (Python framework providing data generation, elastic-net-regularized NMF of the source data, and the transfer workflow) — https://github.com/nicococo/scRNA
- **scikit-learn NMF** (Elastic-net regularized NMF implementation cited in the paper for learning H_src and W_src)

## Examples

```
scRNA-source.sh --gene-ids genes.tsv --src-data src.tsv --src-labels src_labels.tsv --cluster-range 6,7,8
```

## Evaluation signals

- Verify H_src has shape (g x k) and W_src has shape (k x n_src) matching the input source matrix dimensions
- When source labels are provided as initialization, check that the NMF decomposition converges from the one-hot-encoded W_src init
- Check all learned matrices are non-negative, as required for subsequent construction of a non-negative X_trg_new
- Downstream check: with theta = 0 the constructed X_trg_new must exactly equal the original X_trg
- Sanity-check that NMF model cluster labels (.tsv) correspond to the chosen cluster count (e.g. 6, 7 or 8 columns/groups)

## Limitations

- The method requires source and target datasets to share a common gene space; if only a small set of transcripts is shared, information loss is inevitable
- NMF decomposition of the source data can be time-consuming depending on the data and cluster range; t-SNE plots can be disabled with --no-tsne to speed it up
- With Anaconda, an 'Intel MKL FATAL ERROR: Cannot load libmkl_avx.so or libmkl_def.so.' may occur; fix with 'conda install mkl'

## Evidence

- [methods] NMF of source data with elastic net regularization to learn dictionary Hsrc and data matrix Wsrc, initialized with one-hot-encoded source labels: [evidence span withheld — anchor status: not_found]
- [methods] Knowledge transfer through the learned dictionary: "we now transfer knowledge from the source to the target dataset through the dictionary by learning a target data matrix"
- [readme] CLI workflow for source NMF in the README: "NMF of source data using the provided gene ids and source data. Ie. we want
 to turn off the cell- and gene-filter as well as the log transformation.
 You can provide source labels to be used as a"
- [readme] Output artifacts of the source step: "Output source model in .npz format for every number of cluster as specified in the --cluster-range argument (default 6,7,8)"
- [abstract] Overall approach statement: "Our approach modifies the dataset of interest while incorporating key information from the larger reference dataset via Non-negative Matrix Factorization (NMF)"
