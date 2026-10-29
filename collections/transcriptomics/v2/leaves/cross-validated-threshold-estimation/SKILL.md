---
name: cross-validated-threshold-estimation
description: Use when predicting labels for an unlabeled scRNA-seq dataset against
  a trained hierarchical classifier and you need a data-driven cutoff to reject cells
  that belong to no known population.
when_to_use_negative:
- Datasets are not yet aligned/batch-corrected — scHPL requires alignment beforehand
  (e.g. with scVI or scArches) and is not itself a batch correction tool
- The goal is to assign labels within known populations rather than detect unseen
  populations; use the trained classification tree prediction alone
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3512
tools:
- name: scHPL
  role: Framework whose rejection mechanism the cross-validated threshold supports;
    detects unknown populations in its hierarchical classification tree
  repo: https://github.com/lcmmichielsen/scHPL
- name: scArches
  role: Recommended upstream tool for aligning training and test datasets before threshold
    estimation
  repo: https://github.com/theislab/scarches
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-23196-8
    title: Hierarchical progressive learning of cell identities in single-cell data
  source_content_hash: sha256:324b7f155317552e6f655389cff571fb4a3221228c8e77fe3b8107904a003b72
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lcmmichielsen/scHPL
  tools:
  - scArches
  - scHPL
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3512
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# cross-validated-threshold-estimation

## Summary

Estimates a per-dataset rejection threshold from cross-validated reconstruction-error percentiles so that scHPL-style classifiers can flag cells from unseen cell populations instead of forcing a known label.

## When to use

Use when predicting labels for an unlabeled scRNA-seq dataset against a trained hierarchical classifier and you need a data-driven cutoff to reject cells that belong to no known population.

## When NOT to use

- Datasets are not yet aligned/batch-corrected — scHPL requires alignment beforehand (e.g. with scVI or scArches) and is not itself a batch correction tool
- The goal is to assign labels within known populations rather than detect unseen populations; use the trained classification tree prediction alone

## Inputs

- Aligned training AnnData with integrated representation in .X
- Unlabeled test AnnData aligned to the same feature space
- Percentile parameter q (e.g. 0.99) and number of CV folds (5)

## Outputs

- Per-cell reject/accept mask for the test data (e.g. rejection_mask.csv)
- Learned rejection threshold value (median of fold-level q=0.99 percentiles)
- Fitted PCA transformation and per-cell reconstruction errors

## How to apply

Fit a PCA model with 100 components on the aligned training AnnData (integrated representation in .X), since populations never used to fit the PCA are poorly represented by the selected PCs and yield high reconstruction errors. In a nested 5-fold cross-validation loop over the training data, compute per-cell reconstruction errors and, for each fold, take the q=0.99 percentile of those errors; the median of the five fold percentiles becomes the rejection threshold. Project the unseen test data through the fitted PCA, compute per-cell reconstruction errors, and flag cells with errors above the threshold as rejected. Write the resulting accept/reject mask together with the learned threshold value (e.g. to rejection_mask.csv). Datasets must be aligned beforehand (e.g. with scVI or scArches), because scHPL is not a batch correction tool.

## Related tools

- **scHPL** (Framework whose rejection mechanism the cross-validated threshold supports; detects unknown populations in its hierarchical classification tree) — https://github.com/lcmmichielsen/scHPL
- **scArches** (Recommended upstream tool for aligning training and test datasets before threshold estimation) — https://github.com/theislab/scarches

## Examples

```
from sklearn.decomposition import PCA; pca = PCA(n_components=100).fit(X_train); err = ((X_test - pca.inverse_transform(pca.transform(X_test)))**2).sum(1); mask = err > threshold
```

## Evaluation signals

- The learned threshold equals the median of the per-fold q=0.99 percentiles of training reconstruction errors across the 5 folds
- The fraction of rejected test cells is small and concentrated in cells far from the training manifold, consistent with unknown-population detection
- The reject/accept mask has one boolean entry per test cell and matches the number of rows in the test AnnData
- Reconstruction errors for held-out training folds cluster at or below the threshold, while genuinely novel populations exceed it
- Training and test data were verified as aligned before running (scHPL prerequisite)

## Limitations

- scHPL is not a batch correction tool; misalignment can cause false rejections or erroneous tree mismatches
- Performance of the related one-class rejection components decreases on populations with a small number of cells
- Adding a new dataset requires rerunning the alignment and consequently retraining/re-thresholding with scHPL

## Evidence

- [other] rejection via PCA reconstruction error: [evidence span withheld — anchor status: not_found]
- [intro] one-class rejection objective: "we adopt an advanced rejection procedure including a one-class classifier to be able to discover new cell (sub)populations"
- [discussion] alignment prerequisite: "Since batch effects are inevitable when combining datasets, we require datasets to be aligned before running scHPL"
- [readme] README on unknown-population detection: "For each node in the tree either a linear SVM, kNN, or one-class SVM, which enables the detection of unknown populations, is trained"
