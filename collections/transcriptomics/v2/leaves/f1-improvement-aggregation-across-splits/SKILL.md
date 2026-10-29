---
name: f1-improvement-aggregation-across-splits
description: Use when two active learning classifiers (e.g., random forest vs logistic
  regression) have been evaluated with F1 across multiple train/test splits and you
  must aggregate into a single relative improvement metric per search/threshold combination.
when_to_use_negative:
- When comparing a classifier other than random forest vs logistic regression, the
  standardization by the LR F1 baseline is not meaningful
- When the initial training set is not randomly selected (e.g., marker-informed selection
  via leader adaptive reweighting), the RF-over-LR improvement finding this aggregation
  tests does not apply
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_3474
- http://edamontology.org/topic_3308
tools:
- name: caret
  role: R package used to train the random forest and logistic regression classifiers
    whose F1 scores feed the relative improvement aggregation
- name: leader
  role: R package implementing the active learning / cell selection workflow from
    which the F1 scores across loop iterations are aggregated
  repo: https://github.com/camlab-bioml/leader
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-45198-y
    title: The impacts of active and self-supervised learning on efficient annotation
      of single-cell expression data
  source_content_hash: sha256:5390d1159c12a37c3ba238e5845eede22abfff145b8be4f7264fef3d2c100a8b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/camlab-bioml/leader
  tools:
  - SingleR
  - caret
  - leader
  - scmap
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_3474
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# f1-improvement-aggregation-across-splits

## Summary

Aggregates per-iteration F1 scores from active learning runs into a relative improvement score — (RF F1 minus LR F1) standardized by the LR F1 — averaged over ten 50:50 stratified train/test splits and annotation methods, to compare random forest vs logistic regression active learning performance.

## When to use

Use when two active learning classifiers (e.g., random forest vs logistic regression) have been evaluated with F1 across multiple train/test splits and you must aggregate into a single relative improvement metric per search/threshold combination.

## When NOT to use

- When comparing a classifier other than random forest vs logistic regression, the standardization by the LR F1 baseline is not meaningful
- When the initial training set is not randomly selected (e.g., marker-informed selection via leader adaptive reweighting), the RF-over-LR improvement finding this aggregation tests does not apply

## Inputs

- Per-iteration F1 scores for random forest and logistic regression active learning runs, per search strategy (entropy, maximum-probability) and threshold (highest, 95th percentile, 75th percentile)
- Ten 50:50 train/test splits stratified by cell type from each dataset
- Results from SingleR and scmap annotation method runs
- Processed single-cell expression matrices and ground-truth cell type labels from Zenodo deposit 10.5281/zenodo.10403475

## Outputs

- relative_f1_improvement.csv with one row per search-strategy/threshold combination and the aggregated relative F1 improvement values
- Aggregated relative improvement scores reproducing the ordering and magnitudes of Fig. 2A and Supplementary Figs. 4-5

## How to apply

After running the active learning loop (random forest and logistic regression trained on the first 20 principal components of scaled and centered logcount data via the caret R package), collect F1 scores at each loop iteration for every classifier/search-strategy/threshold combination (entropy and maximum-probability search; highest, 95th percentile, and 75th percentile thresholds). For each combination, compute the relative improvement score as (RF F1 minus LR F1) standardized by the LR F1, which normalizes the comparison against the logistic regression baseline. Average these scores across the ten 50:50 train/test splits stratified by cell type and across the annotation methods (SingleR, scmap). Write the aggregated values, one row per search-strategy/threshold combination, to relative_f1_improvement.csv. The expected outcome is that the random forest active learning model shows positive relative improvement when the initial 20 training cells are randomly selected, reproducing Fig. 2A and Supplementary Figs. 4-5.

## Related tools

- **caret** (R package used to train the random forest and logistic regression classifiers whose F1 scores feed the relative improvement aggregation)
- **leader** (R package implementing the active learning / cell selection workflow from which the F1 scores across loop iterations are aggregated) — https://github.com/camlab-bioml/leader

## Examples

```
rel_imp <- mean((rf_f1 - lr_f1) / lr_f1); write.csv(data.frame(rel_imp), "relative_f1_improvement.csv", row.names = FALSE)
```

## Evaluation signals

- relative_f1_improvement.csv contains exactly one row per search-strategy/threshold combination (entropy and maximum-probability at highest, 95th percentile, and 75th percentile thresholds)
- Each improvement value equals the mean of (RF F1 minus LR F1) divided by LR F1 across the ten stratified 50:50 splits and the SingleR/scmap annotation methods
- The ordering and magnitudes of values match Fig. 2A and Supplementary Figs. 4-5, with random forest showing positive relative improvement under random initial selection of 20 cells
- F1 scores were computed across active learning iterations where 10 cells with the highest uncertainty were labeled and the classifier retrained each loop

## Limitations

- Ground-truth labels were derived from previous studies rather than manual annotation, so the F1 baseline itself carries labeling ambiguity
- The experiments were not randomized and no statistical method was used to predetermine sample size, so aggregated improvement scores inherit these reproducibility gaps
- Benchmarking chose to test many hyperparameters and annotation tools at the expense of a large number of datasets, limiting generalization of the aggregated improvement estimates

## Evidence

- [methods] relative improvement score formula: [evidence span withheld — anchor status: not_found]
- [discussion] ten stratified splits: "Each dataset was then randomly split into ten different train and test datasets using a 50:50 split while stratifying by cell type"
- [discussion] classifier training inputs: "we trained a random forest and a logistic regression classifier using the first 20 principal components of scaled and centered logcount expression data as input using the caret R package"
- [discussion] active learning loop: "we selected the set of 10 cells with the highest uncertainty to label using ground truth data. After this set of cells is labeled, the classifier is trained again and the loop repeats."
- [readme] package availability: "An R package implementing the heuristic approaches introduced in this work may be found at https://github.com/camlab-bioml/leader"
