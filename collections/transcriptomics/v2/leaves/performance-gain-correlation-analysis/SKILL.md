---
name: performance-gain-correlation-analysis
description: Use when you have before/after F1 scores from a self-training (pseudo-labelling)
  experiment across multiple annotation methods and want to test whether the improvement
  correlates with baseline performance.
when_to_use_negative:
- You only have a single annotation method's gain (correlation across methods is undefined;
  use the improvement table instead)
- Scores come from unsupervised clustering rather than labeled-annotation benchmarks,
  since the gain is defined against a ground-truth-only supervised baseline
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_2229
tools:
- name: leader (R package)
  role: Provides the active-learning/self-training framework (cell selection and pseudo-labelling)
    whose F1 gains are correlated with baseline F1
  repo: https://github.com/camlab-bioml/leader
- name: caret
  role: Trains the logistic regression and random forest classifiers on the first
    20 principal components of scaled/centered logcounts used in the self-training
    comparison
- name: SingleR
  role: One of the cell annotation methods whose self-training F1 gain vs. baseline
    F1 is correlated
- name: scmap
  role: One of the cell annotation methods whose self-training F1 gain vs. baseline
    F1 is correlated
- name: scuttle
  role: Computes logcounts and size factors used to build the PCA features for classifier
    training
- name: scran
  role: Single-cell processing supporting normalization in the benchmark workflow
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-024-45198-y
    title: The impacts of active and self-supervised learning on efficient annotation
      of single-cell expression data
  source_content_hash: sha256:9743997177de05f20c6a1b4fafa6703c1f87e80759c05a6d62fad7d892f3a829
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/camlab-bioml/leader
  tools:
  - SingleR
  - caret
  - leader
  - leader (R package)
  - scmap
  - scran
  - scuttle
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# performance-gain-correlation-analysis

## Summary

Quantifies the relationship between the F1-score gain from self-training (pseudo-labelling) and the baseline (ground-truth-only) F1 across cell type annotation methods, reproducing the trend in Fig. 5C of the active/self-supervised learning benchmark.

## When to use

Use when you have before/after F1 scores from a self-training (pseudo-labelling) experiment across multiple annotation methods and want to test whether the improvement correlates with baseline performance.

## When NOT to use

- You only have a single annotation method's gain (correlation across methods is undefined; use the improvement table instead)
- Scores come from unsupervised clustering rather than labeled-annotation benchmarks, since the gain is defined against a ground-truth-only supervised baseline

## Inputs

- self_training_f1_improvement.csv (one row per annotation method x percentage of self-labeled cells: 10%, 50%, 100%)
- baseline F1 scores from training on ground-truth labels only (active-learning-selected 100-cell training sets)
- Held-out stratified 50:50 test set predictions per annotation method

## Outputs

- self_training_gain_correlation.csv (correlation between self-training F1 gain and baseline F1 across annotation methods, matching Fig. 5C)

## How to apply

After retraining each annotation method (SingleR, scmap, SingleCellNet, logistic regression, random forest) on ground-truth plus self-labeled cells at 10%, 50% and 100% of most-confidently predicted unlabeled cells, evaluate F1 on the stratified 50:50 held-out test set and subtract the ground-truth-only baseline F1 to obtain per-method improvement values (self_training_f1_improvement.csv, matching Fig. 5B). Then compute the correlation between self-training F1 gain and the baseline F1 across annotation methods and write it to self_training_gain_correlation.csv, reproducing the trend in Fig. 5C. Rationale: the paper found that combining manually annotated data with confidently self-labeled cells increases performance especially for datasets with dissimilar cell types and few annotated cells, so the gain-baseline relationship reveals which methods benefit most from pseudo-labelling.

## Related tools

- **leader (R package)** (Provides the active-learning/self-training framework (cell selection and pseudo-labelling) whose F1 gains are correlated with baseline F1) — https://github.com/camlab-bioml/leader
- **caret** (Trains the logistic regression and random forest classifiers on the first 20 principal components of scaled/centered logcounts used in the self-training comparison)
- **SingleR** (One of the cell annotation methods whose self-training F1 gain vs. baseline F1 is correlated)
- **scmap** (One of the cell annotation methods whose self-training F1 gain vs. baseline F1 is correlated)
- **scuttle** (Computes logcounts and size factors used to build the PCA features for classifier training)
- **scran** (Single-cell processing supporting normalization in the benchmark workflow)

## Examples

```
Rscript compute_self_training_gain.R --improvement self_training_f1_improvement.csv --baseline ground_truth_f1.csv --output self_training_gain_correlation.csv
```

## Evaluation signals

- self_training_gain_correlation.csv exists with one correlation value per annotation method (or per method x percentage level) matching the trend in Fig. 5C
- self_training_f1_improvement.csv contains one row per annotation method x percentage of self-labeled cells (10/50/100%) with F1 relative to ground-truth-only baseline
- The direction and relative ordering of F1 improvements in the improvement table match the bars/curves in Fig. 5B (positive gain for the most effective methods at 10-50% self-labeling)
- F1 is computed on the same stratified 50:50 held-out test set as the baseline for a fair before/after comparison
- Self-labeled cell sets are selected by prediction entropy thresholds (top 10%, 50%, 100% most confidently predicted cells)

## Limitations

- Ground truth cell type labels were taken from previous studies rather than manual annotation, so what a 'correct' label means carries fundamental ambiguity
- Manual annotation was only simulated using existing cell type annotations; the optimal certainty threshold for selecting cells in self-training was not identified and left to future work
- Benchmarking chose to test many tools/hyperparameters at the expense of investigating a large number of datasets, limiting breadth of conditions in the correlation
- Circularity in cell type marker definition, especially for cell line data where markers derive from differential expression between cell lines

## Evidence

- [methods] Self-training: predict remaining training cells, label most confident predictions, add to training set: "We then selected the top 10, 50 and 100% most confidently predicted cells based on their entropy and labeled these using the prediction values obtained."
- [abstract] Combining manually annotated data with confidently self-labeled cells generally increases predictive performance: [evidence span withheld — anchor status: not_found]
- [discussion] Optimal certainty threshold for cell selection in self-training not identified; left to future work: "future work could focus on identifying the best certainty threshold to select cells to label using self-training"
- [methods] Splitting datasets into stratified train/test sets for evaluation: "Each dataset was then randomly split into ten different train and test datasets using a 50:50 split while stratifying by cell type"
- [methods] Classifier training setup on PCA features: "we trained a random forest and a logistic regression classifier using the first 20 principal components of scaled and centered logcount expression data as input using the caret R package"
