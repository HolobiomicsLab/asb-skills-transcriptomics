---
name: classifier-f1-score-evaluation
description: Use when you need to quantify single-cell annotation accuracy of classifiers
  trained on actively selected labeled cells, comparing strategies (e.g., marker-ranked
  vs random initial selection) via F1-score per stratified split.
when_to_use_negative:
- The dataset has no reliable ground-truth cell type labels — the paper notes 'ground
  truth' labels were derived from previous studies and that label meaning in single-cell
  biology is fundamentally ambiguous
- You are evaluating cell selection/subsampling itself (e.g., leader adaptive reweighting)
  rather than downstream classifier performance on labeled test cells
- Markers are circularly defined (e.g., cell line markers derived from differential
  expression between the very cell lines being classified), inflating F1 improvements
edam_topics:
- http://edamontology.org/topic_3474
tools:
- name: caret
  role: Trains the random forest and logistic regression classifiers on the first
    20 principal components and supports F1-score evaluation of their predictions
- name: leader
  role: Runs the active learning loop and marker-informed cell selection that generates
    the labeled training sets whose classifiers are F1-scored
  repo: https://github.com/camlab-bioml/leader
- name: Seurat
  role: Provides the log-normalized, variable-feature-scaled expression object used
    as classifier input
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-45198-y
    title: The impacts of active and self-supervised learning on efficient annotation
      of single-cell expression data
  source_content_hash: sha256:1704a608dc14ccf5968e01b64ffe42089f56f507a8e25e8c8a643c9eae80a12d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/camlab-bioml/leader
  tools:
  - Seurat
  - caret
  - leader
  - leader (R package)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# classifier-f1-score-evaluation

## Summary

Evaluates cell type classifiers (random forest and logistic regression trained on the first 20 principal components of scaled, centered logcounts) by F1-score on held-out test cells, across ten 50:50 train/test splits stratified by cell type, to quantify how active learning and initialization strategies affect annotation accuracy.

## When to use

Use when you need to quantify single-cell annotation accuracy of classifiers trained on actively selected labeled cells, comparing strategies (e.g., marker-ranked vs random initial selection) via F1-score per stratified split.

## When NOT to use

- The dataset has no reliable ground-truth cell type labels — the paper notes 'ground truth' labels were derived from previous studies and that label meaning in single-cell biology is fundamentally ambiguous
- You are evaluating cell selection/subsampling itself (e.g., leader adaptive reweighting) rather than downstream classifier performance on labeled test cells
- Markers are circularly defined (e.g., cell line markers derived from differential expression between the very cell lines being classified), inflating F1 improvements

## Inputs

- Processed, log-normalized single-cell expression dataset (e.g., Seurat object or SingleCellExperiment) with unique cell IDs
- Ground-truth cell type labels for each cell (from previous studies), stratified by cell type for splitting
- Marker gene file (yaml for leader: positive/negative markers per expected cell type) for the marker-ranked initialization arm
- Definition of candidate initial training sets (e.g., top 20 cells by average marker expression vs 20 uniformly random cells)

## Outputs

- Per-split F1-score for each classifier (random forest, logistic regression) at each active learning iteration
- Difference-in-F1 table comparing initialization strategies (e.g., marker_vs_random_f1_improvement.csv)
- Per-split representation statistics of cell types in the initial training set (e.g., represented_cell_type_proportions.csv)

## How to apply

Split each single-cell expression dataset into ten 50:50 train/test partitions while stratifying by cell type. Train a random forest and a logistic regression classifier on the first 20 principal components of scaled and centered logcount expression data using the caret R package. Within the active learning loop, label the 10 highest-uncertainty cells per iteration with ground truth, retrain, and score the classifier with F1 on the untouched test cells after each iteration. To isolate the effect of an initialization strategy (e.g., marker-ranked initial 20 cells vs uniform random selection), compute the difference in F1-score between the two conditions per train-test split, especially at low labeled-cell budgets, and write per-split results as a CSV table (e.g., marker_vs_random_f1_improvement.csv).

## Related tools

- **caret** (Trains the random forest and logistic regression classifiers on the first 20 principal components and supports F1-score evaluation of their predictions)
- **leader** (Runs the active learning loop and marker-informed cell selection that generates the labeled training sets whose classifiers are F1-scored) — https://github.com/camlab-bioml/leader
- **Seurat** (Provides the log-normalized, variable-feature-scaled expression object used as classifier input)

## Examples

```
caret::confusionMatrix(pred = predict(rf_mod, test_pcs), ref = test_labels, mode = "everything")
```

## Evaluation signals

- F1-scores fall in [0,1] and both classifier types (random forest, logistic regression) are scored on every one of the ten 50:50 splits per dataset
- Marker-ranked minus random initialization F1 difference is positive, especially when few cells are labeled, matching Fig. 2C / Supplementary Figs. 6-7 patterns
- Marker-ranked initialization represents more ground-truth cell types within the initial 20-cell set (Fig. 2B pattern) before any active learning iterations
- Output CSVs contain the expected fields (per-split F1 improvement, represented cell type proportions) with plausible value ranges
- Test cells remain unlabeled throughout the loop; uncertainty-based labeling draws only from the training partition (10 highest-uncertainty cells per iteration)

## Limitations

- 'Ground truth' cell type labels were taken from previous studies rather than manual annotation, so F1 is measured against potentially ambiguous labels
- Circularity in marker definition, especially for cell line data where markers derive from differential expression between the cell lines, can bias F1 comparisons
- Dropout at the single-cell level may cause markers not to be expressed, degrading classifier performance in ways not addressed by the benchmark
- The experiments were not randomized and investigators were not blinded, per the paper's reproducibility disclosure

## Evidence

- [discussion] train/test split design: "Each dataset was then randomly split into ten different train and test datasets using a 50:50 split while stratifying by cell type"
- [discussion] classifier training on PCs via caret: "we trained a random forest and a logistic regression classifier using the first 20 principal components of scaled and centered logcount expression data as input using the caret R package"
- [discussion] active learning relabeling loop: "we selected the set of 10 cells with the highest uncertainty to label using ground truth data. After this set of cells is labeled, the classifier is trained again and the loop repeats."
- [readme] required input preprocessing: "The input seurat oject must be log normalized and the most variable features must be identified and scaled prior to cell selection with LEADER"
