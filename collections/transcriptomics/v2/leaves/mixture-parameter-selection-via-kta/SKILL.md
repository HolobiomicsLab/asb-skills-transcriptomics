---
name: mixture-parameter-selection-via-kta
description: 'Use when applying TransferCluster to a small scRNA-Seq target dataset:
  after learning Wtrg against the source NMF dictionary Hsrc, theta must be chosen
  without ground-truth labels to construct Xtrg_new = theta*Hsrc*W''trg + (1-theta)*Xtrg.'
when_to_use_negative:
- No source reference dataset is available, so there is no Hsrc/W'trg reconstruction
  to mix — plain TargetCluster (SC3 on the target alone) is the correct route
- Ground-truth target labels exist and a supervised criterion could be used directly;
  KTA is an unsupervised substitute for cases without labels
- Source and target share only a small set of transcripts — the paper notes only the
  shared gene set can be used and calls this loss of vital information a deferred
  open problem
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scRNA (scRNA-target.sh)
  role: Command-line implementation that sweeps the mixture range, computes KTA scores
    to select theta, and clusters the resulting Xtrg_new
  repo: https://github.com/nicococo/scRNA
- name: SC3
  role: Downstream consensus clustering applied to the KTA-selected modified target
    dataset Xtrg_new
provenance:
  source_task_ids:
  - task_001
  - task_004
  source_papers:
  - doi: 10.1038/s41598-019-56911-z
    title: Using transfer learning from prior reference knowledge to improve the clustering
      of single-cell RNA-Seq data
  source_content_hash: sha256:7b6050677a80915c0721ed35bcdbc6c959efce0189fd9c5ad200a86c55301233
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nicococo/scRNA
  tools:
  - MAGIC
  - NMF
  - SC3
  - Seurat
  - scRNA (scRNA-target.sh)
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# mixture-parameter-selection-via-kta

## Summary

Unsupervised selection of the mixture parameter theta that balances the reconstructed target dataset against the original target data in the scRNA transfer learning framework, using Kernel Target Alignment (KTA) scores as a label-free cluster-quality proxy before SC3 clustering.

## When to use

Use when applying TransferCluster to a small scRNA-Seq target dataset: after learning Wtrg against the source NMF dictionary Hsrc, theta must be chosen without ground-truth labels to construct Xtrg_new = theta*Hsrc*W'trg + (1-theta)*Xtrg.

## When NOT to use

- No source reference dataset is available, so there is no Hsrc/W'trg reconstruction to mix — plain TargetCluster (SC3 on the target alone) is the correct route
- Ground-truth target labels exist and a supervised criterion could be used directly; KTA is an unsupervised substitute for cases without labels
- Source and target share only a small set of transcripts — the paper notes only the shared gene set can be used and calls this loss of vital information a deferred open problem

## Inputs

- Pre-processed log-transformed target expression matrix Xtrg (after cell filter, gene filter, pseudo-count-1 log-transformation)
- Source NMF dictionary Hsrc (elastic-net-regularized NMF of the source dataset, e.g. Usoskin GSE59739 with 4402 shared genes)
- Binary target membership matrix W'trg obtained by column-wise maxima of Wtrg
- Pre-specified mixture (theta) range and SC3 cluster range (default 6,7,8 in the repo CLI)

## Outputs

- Selected mixture parameter theta (maximum KTA score over the sweep)
- Modified target dataset Xtrg_new used as SC3 input
- Per-cluster-range predicted target labels (tsv) and summarizing accuracy/t-SNE figures from scRNA-target.sh

## How to apply

First perform NMF of the source data with elastic net regularization initialized from one-hot-encoded source labels, learn the target data matrix Wtrg given Hsrc, and simplify it to the binary membership matrix W'trg via column-wise maxima (the paper reports the simplified version 'performed best and most consistently for all scenarios'). Then compute Xtrg_new as a convex combination of the reconstructed dataset Hsrc*W'trg and the original log-transformed target expression matrix Xtrg. Sweep theta over a pre-specified range, score each candidate with Kernel Target Alignment (KTA), and select theta automatically via the unsupervised KTA assessment rather than any supervised accuracy measure. Finally pass Xtrg_new to SC3 over the cluster range; the same theta selection underlies TransferCluster results on simulated data (complete/incomplete/no overlap) and on the Hockley (GSE102962) target with the Usoskin (GSE59739) source. The README exposes this via the target-side command line script, where cluster range and mixture range can slow the run and --no-tsne speeds it up.

## Related tools

- **scRNA (scRNA-target.sh)** (Command-line implementation that sweeps the mixture range, computes KTA scores to select theta, and clusters the resulting Xtrg_new) — https://github.com/nicococo/scRNA
- **SC3** (Downstream consensus clustering applied to the KTA-selected modified target dataset Xtrg_new)

## Examples

```
scRNA-target.sh --model src_c8.npz --cluster-range 6,7,8 --no-tsne
```

## Evaluation signals

- KTA score attains a clear maximum at the selected theta across the pre-specified mixture range
- Resulting TransferCluster ARI exceeds both TargetCluster and ConcatenateCluster in complete- and incomplete-overlap simulations, and does not significantly degrade (overlapping 95% CIs) in the no-overlap setting
- On Hockley+Usoskin, chosen-theta TransferCluster separates mNP/mNFa in the majority of 1000 SC3 replications (paper reports 919/1000 with MAGIC) and outperforms baselines
- W'trg contains only binary (0/1) entries with exactly one maximum per cell, confirming column-wise-maxima simplification before mixing
- Selected-theta clustering of target cells is independent of source-only cell types (source clusters absent from target do not appear in target labels)

## Limitations

- KTA is an unsupervised proxy; if it ranks thetas poorly, the convex combination can pull the target toward source structure that is uninformative or harmful (no-overlap scenario risks degradation)
- Mixture-range and cluster-range sweeps plus t-SNE plotting can be very slow (README suggests --no-tsne to speed up)
- Only the genes shared between source and target can be used, and the paper defers handling of differing gene sets to future work

## Evidence

- [methods] Automatic selection of mixture parameter theta via KTA: "θ is automatically chosen via an unsupervised assessment of the clustering quality through Kernel Target Alignment (KTA) scores"
- [methods] Convex combination construction: [evidence span withheld — anchor status: not_found]
- [methods] Simplification of Wtrg: [evidence span withheld — anchor status: not_found]
- [readme] README mixture mix-in and runtime caveat: "augmented clustering of the target data with user defined mix-in of the source data influence"
- [readme] KTA is unsupervised (no labels needed): "Command line output shows a number of results: unsupervised and supervised (if no ground truth labels are given this will remain 0.) accuracy measures."
