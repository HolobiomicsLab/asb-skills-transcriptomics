---
name: cluster-membership-argmax-thresholding
description: Use when a source-regularized NMF has produced soft cluster coefficients
  W_trg for target cells and the transfer-learning construction of X_trg_new requires
  hard one-hot cluster memberships per cell.
when_to_use_negative:
- The target coefficient matrix W_trg has not yet been learned against the source
  dictionary H_src — argmax thresholding presupposes a source-anchored NMF solution,
  not raw expression data
- 'Source and target only share a small set of transcripts: the paper notes ''a loss
  of (probably) vital information is inevitable since only the set of genes present
  in both datasets can be used'''
- You need soft/probabilistic cluster assignments for downstream weighting rather
  than the hard one-hot memberships required by the X_trg_new convex combination
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3170
tools:
- name: scRNA
  role: Python framework implementing the transfer-learning pipeline whose target
    step binarizes W_trg by column-wise maxima and builds X_trg_new before SC3 clustering
  repo: https://github.com/nicococo/scRNA
- name: SC3
  role: Exemplary downstream clustering method applied to X_trg_new built from the
    binarized W'_trg
  repo: https://github.com/nicococo/scRNA
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-019-56911-z
    title: Using transfer learning from prior reference knowledge to improve the clustering
      of single-cell RNA-Seq data
  source_content_hash: sha256:98daa7a7301d007ed4bcead50a6a903c485e2bae53c96bbfd84e8cc620624b83
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nicococo/scRNA
  tools:
  - SC3
  - scRNA
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# cluster-membership-argmax-thresholding

## Summary

Converts the real-valued NMF coefficient matrix W_trg (k x n_trg) learned for the target scRNA-Seq dataset into a binary cluster-membership matrix W'_trg in {0,1}^(k x n_trg) by placing a single 1 at each column's argmax. This hardened matrix is then mixed back into the target data via X_trg_new = theta * H_src * W'_trg + (1 - theta) * X_trg before SC3 clustering.

## When to use

Use when a source-regularized NMF has produced soft cluster coefficients W_trg for target cells and the transfer-learning construction of X_trg_new requires hard one-hot cluster memberships per cell.

## When NOT to use

- The target coefficient matrix W_trg has not yet been learned against the source dictionary H_src — argmax thresholding presupposes a source-anchored NMF solution, not raw expression data
- Source and target only share a small set of transcripts: the paper notes 'a loss of (probably) vital information is inevitable since only the set of genes present in both datasets can be used'
- You need soft/probabilistic cluster assignments for downstream weighting rather than the hard one-hot memberships required by the X_trg_new convex combination

## Inputs

- W_trg: real-valued NMF coefficient/data matrix (k x n_trg) for the target dataset, learned against the fixed source dictionary H_src (g x k)
- Source cluster labels y_src (used one-hot to initialize the source NMF that produced H_src)
- Original pre-processed target expression matrix X_trg (g genes x n_trg cells, shared gene space with source)

## Outputs

- W'_trg: binary cluster-membership matrix in {0,1}^(k x n_trg) with exactly one 1 per column at the argmax cluster
- X_trg_new: modified target dataset theta * H_src * W'_trg + (1 - theta) * X_trg (g x n_trg), saved e.g. as 'X_trg_new.npz' together with H_src and W'_trg

## How to apply

After running elastic-net regularized NMF (L1/L2 mixing controlled by lambda and penalty multiplier alpha) of the source matrix X_src — initialized with a one-hot encoding of the source cluster labels to learn dictionary H_src (g x k) — fix H_src and learn the target coefficient matrix W_trg by minimizing ||X_trg - H_src * W||_Fro^2. Then threshold W_trg column-wise: for each of the n_trg cells, set the entry at the argmax cluster to 1 and all other entries to 0, yielding the binary membership matrix W'_trg in {0,1}^(k x n_trg). The paper deliberately discards the soft coefficient magnitudes because 'taking the simplified version as described above performed best and most consistently for all scenarios under investigation'. W'_trg is then used to reconstruct a denoised target dataset as theta * H_src * W'_trg, mixed with the original X_trg via the convex-combination parameter theta (automatically selected via Kernel Target Alignment scores) and fed to SC3 clustering.

## Related tools

- **scRNA** (Python framework implementing the transfer-learning pipeline whose target step binarizes W_trg by column-wise maxima and builds X_trg_new before SC3 clustering) — https://github.com/nicococo/scRNA
- **SC3** (Exemplary downstream clustering method applied to X_trg_new built from the binarized W'_trg) — https://github.com/nicococo/scRNA

## Examples

```
W_trg_prime = (W_trg == W_trg.max(axis=0)).astype(int); X_trg_new = theta * H_src @ W_trg_prime + (1 - theta) * X_trg
```

## Evaluation signals

- Each column of W'_trg contains exactly one 1, located at the argmax position of the corresponding column of W_trg
- W'_trg has shape (k x n_trg) matching the cluster count k from the NMF dictionary H_src
- With theta = 0, the constructed X_trg_new equals the original X_trg exactly (convex-combination invariant)
- X_trg_new is non-negative and has shape (g x n_trg) in the shared gene space
- Clustering SC3 on X_trg_new improves agreement with target ground-truth labels relative to SC3 on the original target set

## Limitations

- Hard argmax thresholding discards the magnitude information in W_trg; the paper reports the simplified max-based version 'performed best and most consistently', but ties/ambiguous argmax cases remain resolved arbitrarily toward a single cluster
- Only the set of genes shared between source and target can be used; differing gene sets cause information loss and this handling was deferred to future work
- Performance depends on the quality of the source NMF dictionary H_src and on the choice of theta, which the paper selects via unsupervised Kernel Target Alignment (KTA) scores

## Evidence

- [methods] The target clustering matrix Wtrg' is a simplified version of Wtrg with ones at the positions of all column-wise maxima: [evidence span withheld — anchor status: not_found]
- [methods] Simplified max-based W'trg performed best: "it was found that taking the simplified version as described above performed best and most consistently for all scenarios under investigation"
- [methods] Construct Xtrgnew as convex combination: [evidence span withheld — anchor status: not_found]
- [readme] README augmented clustering mix-in: "augmented clustering of the target data with user defined mix-in of the source data influence."
