---
name: label-corruption-perturbation-analysis
description: 'Use when you need to validate or reproduce the entropy-based confidence
  criterion for self-training by simulating annotation mistakes: perturbing known
  ground-truth labels and testing whether predictive entropy separates corrupted from
  correct cells.'
when_to_use_negative:
- The perturbation target is the marker file rather than cell labels — the paper separately
  corrupts 10/25/50/75/100% of markers by replacing them with random genes not in
  the original marker file, which is a different test
- The goal is active learning cell selection or self-training on unlabeled data with
  genuinely predicted labels, not auditing an existing labeled training set for annotation
  mistakes
- The data lack logcount expression suitable for PCA-based classifier input (e.g.,
  non-transcriptomic modalities where scaled and centered logcount expression is not
  defined)
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: caret
  role: Trains the random forest and logistic regression classifiers on the corrupted
    training sample using the first 20 PCs of scaled and centered logcount expression
    data
- name: scuttle
  role: Calculates logcounts and normalizes with the size factor prior to PCA and
    classifier training
- name: leader
  role: R package implementing the paper's heuristic cell-selection approaches, providing
    the surrounding self-training/active-learning context in which the entropy criterion
    is validated
  repo: https://github.com/camlab-bioml/leader
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-024-45198-y
    title: The impacts of active and self-supervised learning on efficient annotation
      of single-cell expression data
  source_content_hash: sha256:849c36802d9b07f227a2bd7508cbebb27b6c3920b410d3b67e097049bf6fa169
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
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 5
---

# label-corruption-perturbation-analysis

## Summary

Deliberately corrupts a fraction of ground-truth cell type labels (e.g., 10% of a 250-cell training sample) and shows that predictive entropy from caret-trained random forest/logistic regression classifiers (first 20 PCs of scaled logcount expression) flags the mislabeled cells, validating the entropy-based confidence criterion used for self-training cell selection.

## When to use

Use when you need to validate or reproduce the entropy-based confidence criterion for self-training by simulating annotation mistakes: perturbing known ground-truth labels and testing whether predictive entropy separates corrupted from correct cells.

## When NOT to use

- The perturbation target is the marker file rather than cell labels — the paper separately corrupts 10/25/50/75/100% of markers by replacing them with random genes not in the original marker file, which is a different test
- The goal is active learning cell selection or self-training on unlabeled data with genuinely predicted labels, not auditing an existing labeled training set for annotation mistakes
- The data lack logcount expression suitable for PCA-based classifier input (e.g., non-transcriptomic modalities where scaled and centered logcount expression is not defined)

## Inputs

- Single-cell logcount expression matrix (e.g., SingleCellExperiment from sc_mixology lung cancer cell lines or other cohorts) with size-factor normalization via scuttle
- Ground truth cell type labels (e.g., cell_line_demuxlet in colData for sc_mixology)
- Ten 50:50 train/test splits stratified by cell type
- Corruption parameters: sample size (250 cells) and corruption fraction (10% of labels)

## Outputs

- mislabeled-cell entropy table written to mislabel_scaled_entropy.csv with columns for cohort, train split, corruption status, classifier, and scaled entropy
- Per-cohort scaled predictive entropy values for caret-trained random forest and logistic regression classifiers, separated into corrupted vs. correctly labeled cells

## How to apply

From each train split (ten 50:50 splits stratified by cell type), randomly sample 250 cells with their ground truth labels, then corrupt 10% of the labels by reassigning incorrect cell types to simulate annotation mistakes. Train a random forest and a logistic regression classifier on the corrupted sample using the caret R package, with the first 20 principal components of scaled and centered logcount expression data (generated with scuttle) as input. Apply the classifiers back to the cells and compute each cell's predictive entropy from the class posterior probabilities, scaling entropy values so they are comparable across cohorts. Tabulate scaled entropy per cohort, train split, corruption status, and classifier; correctness requires corrupted-label cells to show distinguishably higher scaled entropy than correctly labeled cells, mirroring the Fig. 5D comparison. This ties label noise to the self-training workflow, where the top most confidently predicted cells are selected based on their entropy.

## Related tools

- **caret** (Trains the random forest and logistic regression classifiers on the corrupted training sample using the first 20 PCs of scaled and centered logcount expression data)
- **scuttle** (Calculates logcounts and normalizes with the size factor prior to PCA and classifier training)
- **leader** (R package implementing the paper's heuristic cell-selection approaches, providing the surrounding self-training/active-learning context in which the entropy criterion is validated) — https://github.com/camlab-bioml/leader

## Examples

```
rf_fit <- caret::train(x = pcs[, 1:20], y = corrupted_labels, method = "rf"); probs <- predict(rf_fit, type = "prob"); entropy <- -rowSums(probs * log(probs))
```

## Evaluation signals

- In each cohort and train split, corrupted-label cells (10% of 250 cells) exhibit clearly increased scaled predictive entropy relative to correctly labeled cells
- The entropy separation holds consistently for both caret classifiers (random forest and logistic regression)
- Output file mislabel_scaled_entropy.csv contains the expected columns: cohort, train split, corruption status, classifier, and scaled entropy
- Entropy values are computed from class posterior probabilities and scaled so they are comparable across cohorts
- Reproduces the Fig. 5D comparison of scaled entropy between corrupted and correctly labeled cells

## Limitations

- Ground truth cell type labels were taken from previous studies rather than manual annotation, so there is fundamental ambiguity in what a ground-truth label means; the manual annotation process was only simulated using existing annotations
- The optimal certainty threshold for cell selection in self-training was not identified and is left to future work
- The article notes reproducibility gaps: no statistical predetermination of sample size, no randomization, and investigators were not blinded to allocation

## Evidence

- [discussion] caret trained on 20 PCs of logcount data: "we trained a random forest and a logistic regression classifier using the first 20 principal components of scaled and centered logcount expression data as input using the caret R package54"
- [discussion] ten 50:50 stratified splits: "Each dataset was then randomly split into ten different train and test datasets using a 50:50 split while stratifying by cell type"
- [discussion] self-training selects most confident cells by entropy: "We then selected the top 10, 50 and 100% most confidently predicted cells based on their entropy and labeled these using the prediction values obtained."
- [discussion] logcount normalization with scuttle: "we calculated logcounts and normalized with the size factor using the scuttle R package52"
- [readme] ground truth labels location in sc_mixology data: [evidence span withheld — anchor status: not_found]
