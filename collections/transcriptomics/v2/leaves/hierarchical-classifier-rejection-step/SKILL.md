---
name: hierarchical-classifier-rejection-step
description: Use when a labeled single-cell training set and an unlabeled test set
  must be screened for cells that do not belong to any known population, i.e., rejection
  of potential unseen cell populations before hierarchical classification.
when_to_use_negative:
- 'Datasets are not aligned/integrated: scHPL is not a batch correction tool and requires
  datasets to be aligned before matching populations'
- The goal is batch correction or dataset integration rather than rejection of unseen
  cell populations
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scHPL
  role: Framework whose rejection mechanism is reconstructed; hierarchical progressive
    learning with one-class classifiers for detecting unknown populations
  repo: https://github.com/lcmmichielsen/scHPL
- name: scArches
  role: Recommended for aligning training and test datasets before running scHPL (via
    the treeArches framework)
  repo: https://github.com/theislab/scarches
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1038/s41467-021-23196-8
    title: Hierarchical progressive learning of cell identities in single-cell data
  source_content_hash: sha256:01df7bee260dd5bafd64fb618e557a190e261db0e70a7138d5590e73fe7736d1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lcmmichielsen/scHPL
  tools:
  - scArches
  - scHPL
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# hierarchical-classifier-rejection-step

## Summary

Reconstructs scHPL's first rejection step: cells from unseen populations are flagged by thresholding the reconstruction error obtained when projecting data through a PCA model fit only on known training populations, since unknown populations are poorly represented by the selected PCs.

## When to use

Use when a labeled single-cell training set and an unlabeled test set must be screened for cells that do not belong to any known population, i.e., rejection of potential unseen cell populations before hierarchical classification.

## When NOT to use

- Datasets are not aligned/integrated: scHPL is not a batch correction tool and requires datasets to be aligned before matching populations
- The goal is batch correction or dataset integration rather than rejection of unseen cell populations

## Inputs

- Aligned training AnnData with integrated representation in .X (known populations)
- Unlabeled test AnnData aligned to the training representation

## Outputs

- Per-cell reject/accept mask (rejection_mask.csv) flagging cells with PCA reconstruction error above the learned threshold
- Learned rejection threshold value (median across folds of the q=0.99 percentile of training reconstruction errors)

## How to apply

Load the aligned training AnnData (with the integrated representation in .X) and the unlabeled test AnnData using scHPL/anndata; datasets must be aligned beforehand as scHPL requires. Fit a PCA model with 100 components on the training data using scikit-learn and compute per-cell reconstruction errors; the rationale is that unknown populations, not used to learn the PCA transformation, are poorly represented by the selected PCs and therefore yield high reconstruction errors. In a nested 5-fold cross-validation loop over the training data, compute for each fold the q=0.99 percentile of reconstruction errors and take the median across folds as the learned rejection threshold. Project the unseen test data through the fitted PCA, compute per-cell reconstruction errors, and flag cells with errors above the threshold as rejected (potential unseen populations). Write the reject/accept mask together with the learned threshold value to rejection_mask.csv; surviving cells then pass to scHPL's second rejection step (one-class/linear SVM per tree node).

## Related tools

- **scHPL** (Framework whose rejection mechanism is reconstructed; hierarchical progressive learning with one-class classifiers for detecting unknown populations) — https://github.com/lcmmichielsen/scHPL
- **scArches** (Recommended for aligning training and test datasets before running scHPL (via the treeArches framework)) — https://github.com/theislab/scarches

## Examples

```
pca = PCA(n_components=100).fit(X_train); errs = np.sum((X_test - pca.inverse_transform(pca.transform(X_test)))**2, axis=1); mask = errs > threshold
```

## Evaluation signals

- Cells from known populations should mostly fall below the learned threshold, since the PCA was fit on their populations
- The learned threshold equals the median across 5 CV folds of the q=0.99 percentile of training reconstruction errors
- PCA model is fit with exactly 100 components on training data only, and the same model projects the test data
- Output rejection_mask.csv contains a valid mask over all test cells plus the threshold value
- Rejected cells (errors above threshold) should correspond to populations absent from the training annotations, verified downstream by marker gene expression or scHPL's one-class SVM rejection step

## Limitations

- The PCA transformation is learned only from known populations, so genuinely new populations must be poorly represented by the selected PCs to be caught; borderline populations may evade rejection
- Adding a new dataset requires rerunning the alignment and consequently scHPL with current batch correction tools
- Batch effects complicate attribution: it is hard to resolve whether mistakes stem from erroneous alignment or from mismatches created by scHPL
- For classifiers per node, performance decreases on populations with a small number of cells, more apparent for the one-class SVM

## Evidence

- [results] First rejection step via PCA reconstruction error: [evidence span withheld — anchor status: not_found]
- [discussion] Datasets must be aligned: "Since batch effects are inevitable when combining datasets, we require datasets to be aligned before running scHPL"
- [readme] One-class SVM for unknown populations: "For each node in the tree either a linear SVM, kNN, or one-class SVM, which enables the detection of unknown populations, is trained."
- [readme] Not a batch correction tool: "NOTE: scHPL is not a batch correction tool, we advise to align the datasets before matching the cell populations."
- [intro] Objective: rejection procedure with one-class classifier: "(iii) we adopt an advanced rejection procedure including a one-class classifier to be able to discover new cell (sub)populations"
