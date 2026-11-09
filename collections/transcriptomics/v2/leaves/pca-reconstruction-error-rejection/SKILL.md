---
name: pca-reconstruction-error-rejection
description: Use when an unlabeled single-cell test set may contain cell populations
  absent from a labeled training set, and you need to reject (rather than force-classify)
  cells before hierarchical classification with scHPL.
when_to_use_negative:
- Datasets are not yet aligned (scHPL is not a batch correction tool; alignment with
  scVI or scArches is required first)
- The goal is batch correction or integration rather than rejection of unknown populations
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scHPL
  role: Framework implementing the hierarchical progressive learning classifier with
    rejection of unknown populations; the PCA reconstruction-error threshold is its
    first rejection step
  repo: https://github.com/lcmmichielsen/scHPL
- name: scArches
  role: Recommended for aligning training and test datasets before running scHPL/rejection
  repo: https://github.com/theislab/scarches
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-23196-8
    title: Hierarchical progressive learning of cell identities in single-cell data
  source_content_hash: sha256:5c312bf6b19984877392349f09578a1d513663ec5116e7bdc993df0dd333684f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lcmmichielsen/scHPL
  tools:
  - scArches
  - scHPL
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# pca-reconstruction-error-rejection

## Summary

Reconstructs scHPL's first rejection step, which flags cells from unseen populations by thresholding PCA reconstruction errors: populations never used to learn the PCA transformation are poorly represented by the selected principal components and thus yield high reconstruction errors.

## When to use

Use when an unlabeled single-cell test set may contain cell populations absent from a labeled training set, and you need to reject (rather than force-classify) cells before hierarchical classification with scHPL.

## When NOT to use

- Datasets are not yet aligned (scHPL is not a batch correction tool; alignment with scVI or scArches is required first)
- The goal is batch correction or integration rather than rejection of unknown populations

## Inputs

- Aligned training AnnData with integrated representation in .X and labels
- Aligned, unlabeled test AnnData in the same feature space

## Outputs

- rejection_mask.csv containing the per-cell reject/accept mask
- Learned PCA reconstruction-error threshold value (median of 5-fold q=0.99 percentiles)

## How to apply

Load the aligned training AnnData (with the integrated representation in .X) and the unlabeled test AnnData using scHPL/anndata — datasets must be aligned beforehand because scHPL is not a batch-correction tool. Fit a PCA model with 100 components on the training data via scikit-learn and compute per-cell reconstruction errors. Learn the rejection threshold in a nested 5-fold cross-validation loop: for each fold compute the q=0.99 percentile of the reconstruction errors and take the median across folds as the threshold. Project the unseen test data through the fitted PCA, compute per-cell reconstruction errors, and flag cells whose errors exceed the threshold as rejected (potential unseen populations). Write the reject/accept mask together with the learned threshold to rejection_mask.csv.

## Related tools

- **scHPL** (Framework implementing the hierarchical progressive learning classifier with rejection of unknown populations; the PCA reconstruction-error threshold is its first rejection step) — https://github.com/lcmmichielsen/scHPL
- **scArches** (Recommended for aligning training and test datasets before running scHPL/rejection) — https://github.com/theislab/scarches

## Examples

```
from sklearn.decomposition import PCA; pca = PCA(n_components=100).fit(train_X); err = ((pca.inverse_transform(pca.transform(test_X)) - test_X)**2).mean(axis=1); reject = err > threshold
```

## Evaluation signals

- Reconstruction errors of training cells used for threshold fitting are well below the learned q=0.99-median threshold (only ~1% exceed it)
- Unknown populations not used to learn the PCA transformation show markedly higher reconstruction errors than known populations
- rejection_mask.csv is same length as the test AnnData with a boolean mask plus a scalar threshold value
- Rejected mask agrees with scHPL's one-class SVM rejection outputs on populations with enough cells (one-class SVM degrades more on small populations)

## Limitations

- Datasets must be aligned beforehand; adding a new dataset requires rerunning the alignment and scHPL
- Performance of rejection mechanisms decreases on populations with a small number of cells
- Threshold quality depends on the choice of 100 PCA components and the q=0.99 percentile; unseen-but-similar populations may fall below the threshold

## Evidence

- [results] First rejection step (PCA reconstruction error): [evidence span withheld — anchor status: not_found]
- [methods] Nested CV threshold: [evidence span withheld — anchor status: not_found]
- [results] Unknowns poorly represented by PCs: [evidence span withheld — anchor status: not_found]
- [readme] One-class SVM for detection: "For each node in the tree either a linear SVM, kNN, or one-class SVM, which enables the detection of unknown populations, is trained"
- [readme] Alignment prerequisite: "NOTE: scHPL is not a batch correction tool, we advise to align the datasets before matching the cell populations."
