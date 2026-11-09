---
name: scaled-entropy-distribution-comparison
description: Use when validating whether predictive entropy from classifiers trained
  on labeled cells can flag cells whose ground truth labels were corrupted, e.g. simulating
  annotation mistakes before self-training cell selection.
when_to_use_negative:
- Labels have not been (and cannot be) corrupted or audited, so there is no mislabel
  status to compare against — use standard uncertainty-based active learning selection
  instead
- Input is already class posterior probabilities or entropies rather than expression
  count data — no need to retrain classifiers
- The goal is cluster-based subsampling for balanced training sets (LEADER adaptive
  reweighting), not mislabel detection by entropy
edam_topics:
- http://edamontology.org/topic_0092
- http://edamontology.org/topic_3474
tools:
- name: caret
  role: Trains the random forest and logistic regression classifiers on the first
    20 PCs of scaled and centered logcount expression data, whose class posterior
    probabilities yield the predictive entropies
- name: scuttle
  role: Calculates logcounts and normalizes with size factors prior to PCA input for
    the classifiers
- name: leader
  role: R package from the same study implementing cell selection heuristics; context
    for the self-training framework in which entropy-based confidence is applied
  repo: https://github.com/camlab-bioml/leader
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-45198-y
    title: The impacts of active and self-supervised learning on efficient annotation
      of single-cell expression data
  source_content_hash: sha256:f8d09e3ee50faf6905000bf1d949e56746cc875e0e60b0009e21648d9acaa164
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/camlab-bioml/leader
  tools:
  - caret
  - leader
  - scuttle
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0092
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# scaled-entropy-distribution-comparison

## Summary

Detects mislabeled cells in a self-training set by computing predictive entropy from random forest and logistic regression classifiers trained on a label-corrupted training sample, scaling entropies for cross-cohort comparability, and comparing entropy distributions of corrupted vs correctly labeled cells (Fig. 5D-style analysis).

## When to use

Use when validating whether predictive entropy from classifiers trained on labeled cells can flag cells whose ground truth labels were corrupted, e.g. simulating annotation mistakes before self-training cell selection.

## When NOT to use

- Labels have not been (and cannot be) corrupted or audited, so there is no mislabel status to compare against — use standard uncertainty-based active learning selection instead
- Input is already class posterior probabilities or entropies rather than expression count data — no need to retrain classifiers
- The goal is cluster-based subsampling for balanced training sets (LEADER adaptive reweighting), not mislabel detection by entropy

## Inputs

- SingleCellExperiment or equivalent object with raw/normalized counts and ground truth cell type labels in colData (e.g. cell_line_demuxlet from sc_mixology)
- Ten 50:50 train/test splits stratified by cell type
- 250-cell training sample per split with 10% of labels randomly corrupted
- First 20 principal components of scaled and centered logcount expression data

## Outputs

- mislabel_scaled_entropy.csv with columns: cohort, train split, corruption status (corrupted vs correct), classifier (random forest / logistic regression), scaled predictive entropy
- Scaled entropy distributions enabling Fig. 5D-style comparison of corrupted vs correctly labeled cells per cohort

## How to apply

From each of the ten 50:50 train splits stratified by cell type, randomly sample a 250-cell training subset with ground truth labels, then corrupt the labels of 10% of the sampled cells to simulate annotation mistakes. Normalize counts to logcounts with size factors via the scuttle R package, then train a random forest and a logistic regression (caret R package) on the first 20 principal components of the scaled and centered logcount expression data. Apply both classifiers back to the cells, compute each cell's predictive entropy from the class posterior probabilities, and scale the entropy values so they are comparable across cohorts. Tabulate scaled entropy per cell, separated by corruption status and classifier, and write mislabel_scaled_entropy.csv with columns for cohort, train split, corruption status, classifier, and scaled entropy. Validation: corrupted-label cells must show distinguishably higher scaled predictive entropy than correctly labeled cells in each cohort, supporting the paper's entropy-based confidence criterion for self-training cell selection.

## Related tools

- **caret** (Trains the random forest and logistic regression classifiers on the first 20 PCs of scaled and centered logcount expression data, whose class posterior probabilities yield the predictive entropies)
- **scuttle** (Calculates logcounts and normalizes with size factors prior to PCA input for the classifiers)
- **leader** (R package from the same study implementing cell selection heuristics; context for the self-training framework in which entropy-based confidence is applied) — https://github.com/camlab-bioml/leader

## Examples

```
rf <- caret::train(x = as.data.frame(pcs[, 1:20]), y = corrupted_labels, method = "rf"); postprob <- predict(rf, newdata = pcs, type = "prob"); entropy <- -rowSums(postprob * log(postprob))
```

## Evaluation signals

- Corrupted-label cells exhibit distinguishably higher scaled predictive entropy than correctly labeled cells in each cohort, consistent with the entropy-based confidence criterion
- mislabel_scaled_entropy.csv exists with the required columns: cohort, train split, corruption status, classifier, scaled entropy
- Exactly 10% of the 250-cell training sample per split carries corrupted labels, with the remainder retaining correct ground truth labels
- Scaled entropy values are comparable across cohorts (scaling applied) and both classifiers (random forest and logistic regression) are represented per split

## Limitations

- Ground truth cell type labels were taken from previous studies rather than manual annotation, introducing fundamental ambiguity in what a ground-truth label means in single-cell biology
- Manual annotation was only simulated using existing cell type annotations, not performed
- The optimal certainty threshold for cell selection in self-training was not identified and was left to future work

## Evidence

- [discussion] caret trains classifiers on PCs: "we trained a random forest and a logistic regression classifier using the first 20 principal components of scaled and centered logcount expression data as input using the caret R package54"
- [discussion] scuttle normalization: "we calculated logcounts and normalized with the size factor using the scuttle R package52"
- [discussion] entropy-based self-training selection: "We then selected the top 10, 50 and 100% most confidently predicted cells based on their entropy and labeled these using the prediction values obtained."
- [readme] ground truth labels in sc_mixology colData: [evidence span withheld — anchor status: not_found]
