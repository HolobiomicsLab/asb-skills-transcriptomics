---
name: baseline-classifier-comparison-chetah-svmrejection
description: Use when comparing cell-type classifiers on benchmark scRNA-seq data
  (simulated, PBMC-FACS, AMB2018) to quantify hierarchical F1-scores and unseen-population
  rejection rates for scHPL's linear vs one-class SVM against CHETAH and SVMrejection.
when_to_use_negative:
- Datasets have not been batch-corrected/aligned yet — scHPL is not a batch correction
  tool and requires alignment (e.g., with scVI or scArches) before matching cell populations
- The goal is progressive atlas building or updating a classification tree with new
  populations (use treeArches) rather than a fixed cross-validated classifier comparison
  on predefined trees
- Cell populations violate the hierarchical-tree assumption, e.g., dedifferentiation
  such as beta-to-alpha cell conversions in type 2 diabetes
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scHPL
  role: Trains the hierarchical classification tree with a linear SVM or one-class
    SVM at every node (train module), predicts held-out fold labels including rejection
    of unseen populations (predict module), and computes HF1-scores and rejection
    percentages (evaluate module)
  repo: https://github.com/lcmmichielsen/scHPL
- name: CHETAH
  role: Published baseline classifier whose median HF1-score on AMB (0.83) is compared
    against scHPL's linear SVM (0.94) and one-class SVM (0.80) results from Abdelaal
    et al.
- name: scArches
  role: Recommended for aligning the PBMC-FACS/AMB datasets before running scHPL,
    since scHPL requires pre-aligned datasets
  repo: https://github.com/theislab/scarches
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-23196-8
    title: Hierarchical progressive learning of cell identities in single-cell data
  source_content_hash: sha256:f1009c368603407a3b75ccf75108578dd18abd2ee6f7dd353149dbe0bad20e7a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lcmmichielsen/scHPL
  tools:
  - CHETAH
  - scArches
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

# baseline-classifier-comparison-chetah-svmrejection

## Summary

Benchmarks scHPL's hierarchical classifiers (linear SVM vs one-class SVM at each tree node) against CHETAH and SVMrejection baselines on simulated, PBMC-FACS, and AMB2018 scRNA-seq datasets, reporting median HF1-scores and rejection percentages per dataset and classifier.

## When to use

Use when comparing cell-type classifiers on benchmark scRNA-seq data (simulated, PBMC-FACS, AMB2018) to quantify hierarchical F1-scores and unseen-population rejection rates for scHPL's linear vs one-class SVM against CHETAH and SVMrejection.

## When NOT to use

- Datasets have not been batch-corrected/aligned yet — scHPL is not a batch correction tool and requires alignment (e.g., with scVI or scArches) before matching cell populations
- The goal is progressive atlas building or updating a classification tree with new populations (use treeArches) rather than a fixed cross-validated classifier comparison on predefined trees
- Cell populations violate the hierarchical-tree assumption, e.g., dedifferentiation such as beta-to-alpha cell conversions in type 2 diabetes

## Inputs

- Aligned, labeled AnnData objects for simulated, PBMC-FACS, and AMB2018 benchmarks (from Zenodo: doi.org/10.5281/zenodo.4557712 and doi.org/10.5281/zenodo.3357167)
- Predefined hierarchical classification trees per benchmark
- Labels and batch indicators stored as metadata columns per scHPL learn-function input format
- Published CHETAH and SVMrejection benchmark results from Abdelaal et al. (for the AMB comparison)

## Outputs

- Per-fold HF1-scores and rejection percentages for linear SVM and one-class SVM variants on each dataset
- Median HF1-score and median rejection percentage per dataset/classifier (e.g., AMB: linear SVM 0.94, CHETAH 0.83, one-class SVM 0.80)
- hf1_rejection_results.csv containing all medians, rejection percentages, and the AMB baseline comparison

## How to apply

Load the aligned, labeled AnnData objects for the simulated, PBMC-FACS, and AMB2018 benchmarks together with the predefined hierarchical trees, in scHPL's learn-function input format with labels and batch indicators as metadata columns. Train a hierarchical classification tree per classifier variant — either a linear SVM or a one-class SVM at every node — using scHPL's train module, splitting into 10 cross-validation folds for simulated and PBMC-FACS and the 5 folds of Abdelaal et al. for AMB. Predict labels on each held-out fold with scHPL's predict module, recording cells rejected as unseen populations by the one-class-SVM variant. Compute per-fold HF1-scores and rejection percentages with scHPL's evaluate module, aggregate to medians per dataset and classifier, and on AMB compare the scHPL median HF1-scores against the published CHETAH and SVMrejection results from Abdelaal et al. The rationale is that the linear SVM and one-class SVM differ mainly in rejection ability: the one-class SVM rejects unknown populations but loses accuracy, especially on small populations.

## Related tools

- **scHPL** (Trains the hierarchical classification tree with a linear SVM or one-class SVM at every node (train module), predicts held-out fold labels including rejection of unseen populations (predict module), and computes HF1-scores and rejection percentages (evaluate module)) — https://github.com/lcmmichielsen/scHPL
- **CHETAH** (Published baseline classifier whose median HF1-score on AMB (0.83) is compared against scHPL's linear SVM (0.94) and one-class SVM (0.80) results from Abdelaal et al.)
- **scArches** (Recommended for aligning the PBMC-FACS/AMB datasets before running scHPL, since scHPL requires pre-aligned datasets) — https://github.com/theislab/scarches

## Examples

```
pip install scHPL
```

## Evaluation signals

- Reproduced median HF1-scores and rejection percentages match the values in the paper's results tables for each dataset/classifier combination
- Linear SVM classification accuracy is equal to or higher than the one-class SVM on all datasets, with the one-class SVM showing a larger performance decrease on populations with a small number of cells
- On AMB, the linear SVM median HF1-score (~0.94) exceeds CHETAH (0.83) and the one-class SVM (0.80)
- The one-class-SVM variant produces a recorded set of rejected cells (unseen populations) on each held-out fold, absent for the linear-SVM variant
- All medians and rejection percentages are written to hf1_rejection_results.csv with one row per dataset/classifier

## Limitations

- Datasets must be aligned before running scHPL; adding a new dataset requires rerunning the alignment and consequently scHPL with current batch correction tools
- The one-class SVM shows a more apparent performance decrease on populations with a small number of cells
- Batch effects make it hard to determine whether tree mistakes stem from erroneous alignment or mismatches created by scHPL

## Evidence

- [methods] For each node in the tree either a linear SVM, kNN, or one-class SVM is trained.: "For each node in the tree either a linear SVM, kNN, or one-class SVM is trained."
- [discussion] one-class SVM worse on small populations: "For both classifiers, we saw a decrease in performance on populations with a small number of cells, but for the one-class SVM this effect was more apparent."
- [readme] datasets available via Zenodo: [evidence span withheld — anchor status: external_doc]
- [readme] alignment requirement: "NOTE: scHPL is not a batch correction tool, we advise to align the datasets before matching the cell populations."
- [abstract] scHPL purpose: "Here, we present scHPL, a hierarchical progressive learning method which allows continuous learning from single-cell data"
