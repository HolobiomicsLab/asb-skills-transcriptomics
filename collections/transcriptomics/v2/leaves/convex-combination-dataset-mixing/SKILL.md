---
name: convex-combination-dataset-mixing
description: Use when a small target scRNA-Seq dataset must be improved for downstream
  clustering by mixing in structure learned from a large annotated source dataset
  via NMF, without concatenating source and target cells.
when_to_use_negative:
- Source and target datasets share only a small set of transcripts — the paper notes
  a loss of (probably) vital information is inevitable since only genes present in
  both datasets can be used
- The goal is to combine cells from source and target into one matrix — this skill
  explicitly transfers knowledge without concatenating the two datasets
- No source cluster annotations (labels) are available to initialize/anchor the source
  NMF, undermining the transfer of source cell-type knowledge
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scRNA (cmd_generate_data.py)
  role: Python framework implementing data generation, source NMF modeling, and target
    transfer/mixing steps producing X_trg_new
  repo: https://github.com/nicococo/scRNA
- name: SC3
  role: Exemplary downstream clustering method applied to the modified target dataset
    X_trg_new
- name: scikit-learn NMF
  role: Elastic-net regularized NMF used to learn H_src and W_trg cited in the paper
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41598-019-56911-z
    title: Using transfer learning from prior reference knowledge to improve the clustering
      of single-cell RNA-Seq data
  source_content_hash: sha256:7c584b537d5d7d74fc934f1f3a211ff22bb8f9a7e5315a5065a006e37c4a85b4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nicococo/scRNA
  tools:
  - SC3
  - scRNA (cmd_generate_data.py)
  - scikit-learn NMF
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# convex-combination-dataset-mixing

## Summary

Constructs a new target dataset X_trg_new as the convex combination theta * H_src * W'_trg + (1 - theta) * X_trg, blending the source-dictionary reconstruction of the target with the original target expression matrix so knowledge from a large labeled source is mixed in without concatenating the two datasets.

## When to use

Use when a small target scRNA-Seq dataset must be improved for downstream clustering by mixing in structure learned from a large annotated source dataset via NMF, without concatenating source and target cells.

## When NOT to use

- Source and target datasets share only a small set of transcripts — the paper notes a loss of (probably) vital information is inevitable since only genes present in both datasets can be used
- The goal is to combine cells from source and target into one matrix — this skill explicitly transfers knowledge without concatenating the two datasets
- No source cluster annotations (labels) are available to initialize/anchor the source NMF, undermining the transfer of source cell-type knowledge

## Inputs

- Source dictionary H_src (g genes x k clusters) learned by elastic-net regularized NMF of the source expression matrix X_src
- Target coefficient matrix W_trg (k x n_trg cells) learned against the fixed H_src
- Original pre-processed target expression matrix X_trg (g x n_trg, shared gene space, cell/gene filtered and log-transformed)
- Mixture parameter theta in [0,1]

## Outputs

- Binary target clustering matrix W'_trg in {0,1}^(k x n_trg) with exactly one 1 per column at the argmax of W_trg
- New target dataset X_trg_new (g x n_trg), e.g. saved to 'X_trg_new.npz' together with H_src and W'_trg
- Modified target dataset fed to a downstream clustering algorithm (e.g. SC3)

## How to apply

After NMF of the source data (elastic-net regularized, W_src initialized with one-hot-encoded source cluster labels) yields the dictionary H_src (g x k), fix H_src and learn the target coefficient matrix W_trg by minimizing ||X_trg - H_src * W||_Fro^2. Simplify W_trg to the binary membership matrix W'_trg in {0,1}^(k x n_trg) by setting ones at column-wise argmax positions — the paper found this simplified max-based version performed best and most consistently. Then form X_trg_new = theta * H_src * W'_trg + (1 - theta) * X_trg, where theta in [0,1] controls the mix-in of source influence; theta is automatically selected via unsupervised Kernel Target Alignment (KTA) scores. Save H_src, W'_trg, and X_trg_new (e.g. to 'X_trg_new.npz') and feed X_trg_new to a downstream clusterer such as SC3.

## Related tools

- **scRNA (cmd_generate_data.py)** (Python framework implementing data generation, source NMF modeling, and target transfer/mixing steps producing X_trg_new) — https://github.com/nicococo/scRNA
- **SC3** (Exemplary downstream clustering method applied to the modified target dataset X_trg_new)
- **scikit-learn NMF** (Elastic-net regularized NMF used to learn H_src and W_trg cited in the paper)

## Examples

```
X_trg_new = theta * H_src @ W_trg_bin + (1 - theta) * X_trg; np.savez('X_trg_new.npz', H_src=H_src, W_trg_bin=W_trg_bin, X_trg_new=X_trg_new)
```

## Evaluation signals

- X_trg_new has shape (g x n_trg) and is non-negative
- Setting theta = 0 recovers the original target dataset exactly (X_trg_new == X_trg)
- Each column of W'_trg contains exactly one 1 located at the argmax of the corresponding column of W_trg
- SC3 clustering of X_trg_new improves over SC3 on the original target set (assessed via KTA scores and, where available, ground-truth accuracy)

## Limitations

- Only genes present in both source and target can be used; differing gene sets cause loss of potentially vital information
- Performance depends on the mixture parameter theta, which must be selected (via KTA) per setting
- The max-based binarization of W_trg discards soft membership information, though it was found to perform best across investigated scenarios

## Evidence

- [methods] Convex combination construction: [evidence span withheld — anchor status: not_found]
- [methods] Binarization of W_trg: [evidence span withheld — anchor status: not_found]
- [methods] Automatic theta selection: "θ is automatically chosen via an unsupervised assessment of the clustering quality through Kernel Target Alignment (KTA) scores76"
- [intro] No concatenation guarantee: "To enable knowledge transfer without having to combine the two datasets and at the same time guarantee a target clustering to be independent of the cell types of the source data"
- [readme] Repository availability: "A Python framework for single-cell RNA-Seq clustering with special focus on transfer learning."
