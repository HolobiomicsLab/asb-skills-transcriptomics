---
name: hf1-hierarchical-metric-computation
description: Use when evaluating hierarchical cell-type classifiers on scRNA-seq benchmarks
  where predictions must be scored with hierarchical F1 and the fraction of cells
  rejected as unseen populations must be reported per fold.
when_to_use_negative:
- scHPL is not a batch correction tool — do not use it to align datasets; alignment
  (e.g., with scVI or scArches) must be done beforehand
- Datasets are not aligned or labels lack a consistent terminology across studies
  — metrics on misaligned data are not comparable to published HF1 values
- The evaluation target is a flat, non-hierarchical classifier without rejection,
  where plain per-population F1 (not hierarchical F1) is the appropriate metric
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scHPL
  role: Trains hierarchical classification trees (linear SVM or one-class SVM per
    node), predicts labels with rejection of unseen populations, and provides the
    evaluate module for computing HF1-scores and rejection percentages
  repo: https://github.com/lcmmichielsen/scHPL
- name: CHETAH
  role: Published hierarchical classifier used as baseline comparison for median F1-score
    on the AMB dataset (0.83 vs scHPL's 0.94)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-23196-8
    title: Hierarchical progressive learning of cell identities in single-cell data
  source_content_hash: sha256:86b099aa45fbe1eb1f6d2f113d0fdb44aff6b55a1b8e291b029f6bfeae1eb528
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lcmmichielsen/scHPL
  tools:
  - CHETAH
  - scHPL
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# hf1-hierarchical-metric-computation

## Summary

Computes hierarchical F1 (HF1) scores and rejection percentages to evaluate hierarchical cell-type classifiers such as scHPL on scRNA-seq benchmarks, capturing both annotation accuracy at different label resolutions and the ability to reject unseen populations.

## When to use

Use when evaluating hierarchical cell-type classifiers on scRNA-seq benchmarks where predictions must be scored with hierarchical F1 and the fraction of cells rejected as unseen populations must be reported per fold.

## When NOT to use

- scHPL is not a batch correction tool — do not use it to align datasets; alignment (e.g., with scVI or scArches) must be done beforehand
- Datasets are not aligned or labels lack a consistent terminology across studies — metrics on misaligned data are not comparable to published HF1 values
- The evaluation target is a flat, non-hierarchical classifier without rejection, where plain per-population F1 (not hierarchical F1) is the appropriate metric

## Inputs

- Aligned, labeled AnnData objects for simulated, PBMC-FACS, and AMB2018 benchmark datasets
- Predefined hierarchical classification trees per benchmark
- Cross-validation fold assignments (10 folds for simulated and PBMC-FACS; 5 folds of Abdelaal et al. for AMB)
- Published baseline results (CHETAH, SVMrejection from Abdelaal et al.) for AMB comparison

## Outputs

- Per-fold HF1-scores per dataset and classifier variant (linear SVM vs one-class SVM)
- Per-fold rejection percentages of cells classified as unseen populations
- Median HF1 and median rejection percentage aggregated per dataset/classifier
- hf1_rejection_results.csv containing medians, rejection percentages, and AMB baseline comparison

## How to apply

Load aligned, labeled AnnData objects (simulated, PBMC-FACS, AMB2018) together with the predefined hierarchical trees in scHPL's learn-function input format. Split data into 10 cross-validation folds (simulated and PBMC-FACS) or the 5 folds of Abdelaal et al. (AMB), train the hierarchical tree with a linear SVM or one-class SVM at every node via scHPL's train module, and predict held-out folds with scHPL's predict module, recording cells rejected as unseen populations for the one-class-SVM variant. Compute per-fold HF1-scores and rejection percentages with scHPL's evaluate module, then aggregate to median HF1 and median rejection percentage per dataset and classifier. On AMB, compare the scHPL median HF1-scores against published CHETAH and SVMrejection results from Abdelaal et al. (median F1 0.94 vs 0.83 and 0.80). Validation: reproduced median HF1-scores and rejection percentages should match the paper's reported values for each dataset/classifier combination.

## Related tools

- **scHPL** (Trains hierarchical classification trees (linear SVM or one-class SVM per node), predicts labels with rejection of unseen populations, and provides the evaluate module for computing HF1-scores and rejection percentages) — https://github.com/lcmmichielsen/scHPL
- **CHETAH** (Published hierarchical classifier used as baseline comparison for median F1-score on the AMB dataset (0.83 vs scHPL's 0.94))

## Examples

```
pip install scHPL
```

## Evaluation signals

- Reproduced median HF1-scores match the values reported in the paper's results tables for each dataset/classifier combination
- Reproduced median rejection percentages for the one-class-SVM variant match the paper's reported values
- On AMB, scHPL's linear-SVM median F1 reaches 0.94, exceeding CHETAH (0.83) and the one-class SVM (0.80)
- Linear SVM accuracy is equal to or higher than one-class SVM on all datasets in the cross-validation results
- hf1_rejection_results.csv contains medians, rejection percentages, and the AMB baseline comparison as expected from the workflow

## Limitations

- Performance decreases on cell populations with a small number of cells, and this effect is more apparent for the one-class SVM
- Datasets must be aligned before running scHPL; when a new dataset is added, the alignment — and consequently also scHPL — has to be rerun
- Batch effects make it hard to determine whether mistakes stem from erroneous alignment or mismatches created by scHPL, affecting metric interpretation

## Evidence

- [methods] For each node in the tree either a linear SVM, kNN, or one-class SVM is trained.: "For each node in the tree either a linear SVM, kNN, or one-class SVM is trained."
- [methods] The trained classification tree can be used to predict the labels of a new unlabeled dataset.: "The trained classification tree can be used to predict the labels of a new unlabeled dataset."
- [discussion] Small-population performance drop for one-class SVM: "For both classifiers, we saw a decrease in performance on populations with a small number of cells, but for the one-class SVM this effect was more apparent."
- [discussion] Alignment requirement for scHPL inputs: "Since batch effects are inevitable when combining datasets, we require datasets to be aligned before running scHPL"
- [readme] Benchmark datasets downloadable from Zenodo: [evidence span withheld — anchor status: external_doc]
