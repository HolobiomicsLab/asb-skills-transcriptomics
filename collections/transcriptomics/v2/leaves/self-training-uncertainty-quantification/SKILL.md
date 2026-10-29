---
name: self-training-uncertainty-quantification
description: Use when you have a labeled single-cell training set and need to quantify
  per-cell annotation uncertainty (predictive entropy) to identify or flag potentially
  mislabeled cells before self-training.
when_to_use_negative:
- The goal is active learning cell selection (choosing which unlabeled cells to annotate
  next) rather than auditing labels of an already-labeled training set
- The analysis concerns marker file corruption (replacing 10-100% of marker genes
  with random highly expressed genes) rather than cell label corruption
- Input is raw count data without normalization to logcounts and PCA, since classifiers
  are trained on the first 20 PCs of scaled and centered logcounts
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: caret
  role: Trains the random forest and logistic regression classifiers used to compute
    predictive entropy of mislabeled vs correctly labeled cells
- name: scuttle
  role: Calculates logcounts and normalization with size factors, the expression values
    underlying the PCA input features
- name: leader
  role: R package implementing the paper's cell selection heuristics that self-training
    confidence thresholds feed into
  repo: https://github.com/camlab-bioml/leader
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-45198-y
    title: The impacts of active and self-supervised learning on efficient annotation
      of single-cell expression data
  source_content_hash: sha256:c7c816e05b737e7f3068c16b446d1c738929a529ce95d6ea3e393b0b4141347e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/camlab-bioml/leader
  tools:
  - caret
  - leader
  - scuttle
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# self-training-uncertainty-quantification

## Summary

Detects mislabeled cells within a training set by corrupting a fraction of ground truth labels, training random forest and logistic regression classifiers on the corrupted labels, and showing that corrupted cells exhibit higher scaled predictive entropy derived from class posterior probabilities.

## When to use

Use when you have a labeled single-cell training set and need to quantify per-cell annotation uncertainty (predictive entropy) to identify or flag potentially mislabeled cells before self-training.

## When NOT to use

- The goal is active learning cell selection (choosing which unlabeled cells to annotate next) rather than auditing labels of an already-labeled training set
- The analysis concerns marker file corruption (replacing 10-100% of marker genes with random highly expressed genes) rather than cell label corruption
- Input is raw count data without normalization to logcounts and PCA, since classifiers are trained on the first 20 PCs of scaled and centered logcounts

## Inputs

- Processed single-cell expression data (logcounts, scaled and centered, e.g. SingleCellExperiment from scuttle normalization)
- Training subsets of 250 cells sampled from ten 50:50 train/test splits stratified by cell type
- Ground truth cell type labels, with 10% deliberately corrupted to simulate annotation mistakes
- First 20 principal components of the scaled expression data

## Outputs

- mislabel_scaled_entropy.csv with columns for cohort, train split, corruption status, classifier, and scaled entropy
- Per-cell predictive entropy values from random forest and logistic regression posterior probabilities
- Comparison showing corrupted-label cells have distinguishably higher scaled entropy (Fig. 5D)

## How to apply

From each train split (ten 50:50 splits stratified by cell type), sample a 250-cell training subset with ground truth cell type labels, then corrupt 10% of the labels by reassigning incorrect cell types to simulate annotation mistakes. Train a random forest and a logistic regression classifier on the corrupted sample with the caret R package, using the first 20 principal components of scaled and centered logcount expression data as input. Apply each classifier back to the cells, compute predictive entropy from the class posterior probabilities, and scale the entropy values so they are comparable across cohorts. Tabulate scaled entropy per cell, separating corrupted-label from correctly labeled cells, per cohort; the rationale (Fig. 5D) is that corrupted-label cells show clearly increased entropy, providing the confidence criterion for selecting cells in self-training.

## Related tools

- **caret** (Trains the random forest and logistic regression classifiers used to compute predictive entropy of mislabeled vs correctly labeled cells)
- **scuttle** (Calculates logcounts and normalization with size factors, the expression values underlying the PCA input features)
- **leader** (R package implementing the paper's cell selection heuristics that self-training confidence thresholds feed into) — https://github.com/camlab-bioml/leader

## Examples

```
model <- caret::train(x = as.data.frame(pcs[, 1:20]), y = corrupted_labels, method = "rf"); probs <- predict(model, type = "prob")
```

## Evaluation signals

- Corrupted-label cells (10% of the 250-cell sample) exhibit clearly higher scaled predictive entropy than correctly labeled cells across train splits, for both random forest and logistic regression
- mislabel_scaled_entropy.csv contains the expected columns: cohort, train split, corruption status, classifier, and scaled entropy
- Entropy values are scaled so comparisons are valid across cohorts
- Ten independent 50:50 splits stratified by cell type are processed and all appear in the output table
- Classifiers are trained on exactly the first 20 principal components of scaled and centered logcounts, matching the paper's input feature set

## Limitations

- Ground truth cell type labels were derived from previous studies rather than manual annotation, so there is fundamental ambiguity in what a ground-truth label means
- The optimal certainty threshold for selecting cells in self-training was not identified and was left to future work
- Dropout at the single-cell level could complicate annotation when specific markers are not expressed, and was not addressed
- Manual annotation was only simulated using existing annotations, not performed, so real annotator error patterns may differ

## Evidence

- [discussion] Classifiers trained on PCs of scaled logcounts: "we trained a random forest and a logistic regression classifier using the first 20 principal components of scaled and centered logcount expression data"
- [discussion] Self-training selects most confidently predicted cells by entropy: "We then selected the top 10, 50 and 100% most confidently predicted cells based on their entropy and labeled these using the prediction values obtained."
- [intro] Entropy was the active learning uncertainty criterion evaluated: "(ii) what (if any) active learning search strategy such as maximum entropy search performs best?"
- [discussion] Normalization of expression prior to feature computation: "we calculated logcounts and normalized with the size factor using the scuttle R package52"
