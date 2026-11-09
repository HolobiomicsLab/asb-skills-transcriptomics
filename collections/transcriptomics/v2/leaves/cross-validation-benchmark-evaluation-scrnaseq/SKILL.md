---
name: cross-validation-benchmark-evaluation-scrnaseq
description: Use when benchmarking cell-type classifiers on scRNA-seq data with predefined
  hierarchical trees, requiring k-fold cross-validated HF1-scores, rejection rates
  of unseen populations, and comparison to published baselines.
when_to_use_negative:
- Datasets are not yet aligned (batch-corrected) — scHPL requires datasets to be aligned
  before matching cell populations and is not a batch correction tool
- The goal is de novo tree construction rather than evaluating performance of a predefined
  tree with known folds
- Cell population labels are missing or at inconsistent resolutions without a defined
  hierarchical structure to train against
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scHPL
  role: Trains hierarchical classification trees with linear SVM or one-class SVM
    per node, predicts labels on held-out folds, and computes HF1-scores and rejection
    percentages via its train, predict, and evaluate modules
  repo: https://github.com/lcmmichielsen/scHPL
- name: CHETAH
  role: Published baseline classifier whose median F1-score (0.83 on AMB) is compared
    against scHPL's reproduced median HF1-score (0.94)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-23196-8
    title: Hierarchical progressive learning of cell identities in single-cell data
  source_content_hash: sha256:c696c1958af7bc4da2c90fa4f80a9395b034c07cabfef13aa4625259c9c1816b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lcmmichielsen/scHPL
  tools:
  - CHETAH
  - scHPL
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# cross-validation-benchmark-evaluation-scrnaseq

## Summary

Reproduce cross-validated benchmarking of scHPL's linear SVM versus one-class SVM classifiers across simulated, PBMC-FACS, and AMB datasets by computing median hierarchical F1 (HF1) scores and rejection percentages, and comparing against CHETAH and SVMrejection baselines.

## When to use

Use when benchmarking cell-type classifiers on scRNA-seq data with predefined hierarchical trees, requiring k-fold cross-validated HF1-scores, rejection rates of unseen populations, and comparison to published baselines.

## When NOT to use

- Datasets are not yet aligned (batch-corrected) — scHPL requires datasets to be aligned before matching cell populations and is not a batch correction tool
- The goal is de novo tree construction rather than evaluating performance of a predefined tree with known folds
- Cell population labels are missing or at inconsistent resolutions without a defined hierarchical structure to train against

## Inputs

- Aligned, labeled AnnData objects for simulated, PBMC-FACS, and AMB2018 datasets with labels and batch indicators as metadata columns
- Predefined hierarchical classification trees for each benchmark
- Published CHETAH and SVMrejection baseline F1-scores from Abdelaal et al. for AMB comparison
- Cross-validation fold definitions (10 folds for simulated/PBMC-FACS; 5 folds from Abdelaal et al. for AMB)

## Outputs

- Median HF1-score per dataset and classifier variant
- Median rejection percentage per dataset and classifier variant
- hf1_rejection_results.csv containing medians, rejection percentages, and AMB baseline comparison

## How to apply

Load the aligned, labeled AnnData objects for the simulated, PBMC-FACS, and AMB benchmarks together with their predefined hierarchical trees, formatted per scHPL's learn-function input requirements. For each dataset and each classifier variant (linear SVM and one-class SVM), split into 10 cross-validation folds for simulated and PBMC-FACS, or the 5 folds of Abdelaal et al. for AMB, and train the hierarchical classification tree using scHPL's train module with either a linear SVM or one-class SVM at every node. Predict labels on held-out folds with scHPL's predict module, recording which cells are rejected as unseen populations for the one-class-SVM variant. Compute per-fold HF1-scores and rejection percentages with scHPL's evaluate module, aggregate to median HF1 and median rejection percentage per dataset and classifier, and compare AMB results against published CHETAH and SVMrejection results from Abdelaal et al. (median F1 0.83 and 0.94 for linear SVM, 0.80 for one-class SVM).

## Related tools

- **scHPL** (Trains hierarchical classification trees with linear SVM or one-class SVM per node, predicts labels on held-out folds, and computes HF1-scores and rejection percentages via its train, predict, and evaluate modules) — https://github.com/lcmmichielsen/scHPL
- **CHETAH** (Published baseline classifier whose median F1-score (0.83 on AMB) is compared against scHPL's reproduced median HF1-score (0.94))

## Examples

```
import scHPL; tree = scHPL.train(data=adata, tree=predefined_tree, classifier='linear_svm'); predictions = scHPL.predict(tree=tree, data=adata_test); hf1, rejection = scHPL.evaluate(predictions=predictions, labels=labels_test)
```

## Evaluation signals

- Reproduced median HF1-scores match the values reported in the paper's results tables for each dataset/classifier combination (e.g., linear SVM median F1 of 0.94 on AMB vs CHETAH's 0.83 and one-class SVM's 0.80)
- Median rejection percentages per dataset and classifier match the paper's reported rejection rates
- The output hf1_rejection_results.csv contains all required medians, rejection percentages, and the AMB baseline comparison
- Linear SVM accuracy is equal to or higher than one-class SVM on all datasets, consistent with the paper's reported finding
- Rejection behavior is only observed/recorded for the one-class-SVM variant, not the linear-SVM variant

## Limitations

- Datasets must be aligned before running scHPL since batch effects are inevitable when combining datasets; scHPL is not a batch correction tool
- One-class SVM performance decreases more on populations with a small number of cells compared to the linear SVM
- Adding a new dataset requires rerunning the alignment and consequently scHPL with current batch correction tools

## Evidence

- [discussion] Linear SVM accuracy equal to or higher than one-class SVM on all datasets; one-class SVM performance decreases more on small populations: "For both classifiers, we saw a decrease in performance on populations with a small number of cells, but for the one-class SVM this effect was more apparent."
- [discussion] Datasets must be aligned before running scHPL: "Since batch effects are inevitable when combining datasets, we require datasets to be aligned before running scHPL"
- [methods] Train hierarchical classifier with linear SVM or one-class SVM per node: "For each node in the tree either a linear SVM, kNN, or one-class SVM is trained."
- [readme] Datasets available from Zenodo for reproduction: [evidence span withheld — anchor status: external_doc]
