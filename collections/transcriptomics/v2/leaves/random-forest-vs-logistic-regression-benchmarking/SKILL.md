---
name: random-forest-vs-logistic-regression-benchmarking
description: Use when benchmarking which classifier (random forest vs logistic regression)
  better drives active learning cell selection from single-cell expression data when
  the initial training set is randomly chosen and may not contain all cell types.
when_to_use_negative:
- When the initial training set is guaranteed to contain all cell types (e.g. marker-informed
  or adaptive-reweighting selection), since the RF-vs-LR advantage was established
  specifically under random initial selection of 20 cells
- When the goal is cell selection/subsampling itself (adaptive reweighting, self-training)
  rather than comparing classifier backends within an active learning loop
- When data are not log-normalized single-cell expression matrices with ground-truth
  labels; CyTOF-style or other feature tables without cell type annotations are out
  of scope for this comparison
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3474
tools:
- name: leader
  role: R package implementing the active learning / adaptive reweighting workflow
    for cell selection used to run the classifier comparison
  repo: https://github.com/camlab-bioml/leader
- name: caret
  role: Trains the random forest and logistic regression classifiers on the first
    20 principal components of scaled, centered logcount expression data
- name: SingleR
  role: One of the single-cell annotation algorithms whose results feed into the averaged
    relative F1 improvement scores
- name: scmap
  role: One of the single-cell annotation algorithms whose results feed into the averaged
    relative F1 improvement scores
- name: Seurat
  role: Provides normalization, variable feature detection and scaling of the input
    object required before cell selection and classifier training
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-45198-y
    title: The impacts of active and self-supervised learning on efficient annotation
      of single-cell expression data
  source_content_hash: sha256:ef1d41094d637a2932fe396e87a6506682a3ae2931debfd73b2f8175aefa7aa5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/camlab-bioml/leader
  tools:
  - Seurat
  - SingleR
  - caret
  - leader
  - scmap
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# random-forest-vs-logistic-regression-benchmarking

## Summary

Reproduces the article's comparison of random forest vs logistic regression classifiers used as the base learner inside an active learning loop for single-cell annotation, quantifying RF's relative F1 improvement over LR under random initial training-set selection.

## When to use

Use when benchmarking which classifier (random forest vs logistic regression) better drives active learning cell selection from single-cell expression data when the initial training set is randomly chosen and may not contain all cell types.

## When NOT to use

- When the initial training set is guaranteed to contain all cell types (e.g. marker-informed or adaptive-reweighting selection), since the RF-vs-LR advantage was established specifically under random initial selection of 20 cells
- When the goal is cell selection/subsampling itself (adaptive reweighting, self-training) rather than comparing classifier backends within an active learning loop
- When data are not log-normalized single-cell expression matrices with ground-truth labels; CyTOF-style or other feature tables without cell type annotations are out of scope for this comparison

## Inputs

- Processed single-cell expression matrices (logcount) with ground-truth cell type labels from Zenodo deposit 10.5281/zenodo.10403475
- Seurat object with log-normalized expression, variable features identified, and genes scaled (for leader-based selection)
- Optional cell-type marker YAML file with positive and negative markers per expected cell type

## Outputs

- relative_f1_improvement.csv with one row per search-strategy/threshold combination containing relative F1 improvement values
- F1 score trajectories across active learning iterations for each classifier/search/threshold combination

## How to apply

Load processed single-cell expression matrices and ground-truth cell type labels from Zenodo deposit 10.5281/zenodo.10403475, then randomly split each dataset into ten 50:50 train/test splits stratified by cell type, initializing each active learning run with 20 randomly selected cells. Using the leader package workflow, train a random forest and a logistic regression classifier with the caret R package on the first 20 principal components of scaled and centered logcount expression data. At each loop iteration, rank unlabeled cells by predictive uncertainty using entropy and maximum-probability search at the highest, 95th percentile, and 75th percentile thresholds, label the 10 cells with highest uncertainty against ground truth, and retrain. For each classifier/search/threshold combination compute F1 scores across iterations and derive the relative improvement score as (RF F1 minus LR F1) standardized by the LR F1. Average relative improvement scores over the ten splits and the annotation methods (SingleR, scmap), writing the aggregate to relative_f1_improvement.csv to match Fig. 2A and Supplementary Figs. 4–5.

## Related tools

- **leader** (R package implementing the active learning / adaptive reweighting workflow for cell selection used to run the classifier comparison) — https://github.com/camlab-bioml/leader
- **caret** (Trains the random forest and logistic regression classifiers on the first 20 principal components of scaled, centered logcount expression data)
- **SingleR** (One of the single-cell annotation algorithms whose results feed into the averaged relative F1 improvement scores)
- **scmap** (One of the single-cell annotation algorithms whose results feed into the averaged relative F1 improvement scores)
- **Seurat** (Provides normalization, variable feature detection and scaling of the input object required before cell selection and classifier training)

## Examples

```
devtools::install_github("camlab-bioml/leader"); library(leader); seu <- ScaleData(FindVariableFeatures(NormalizeData(seu)), features = all.genes); cells <- adaptive_reweighting(seu, n_requested_cells = 40, mode = "cluster_based")
```

## Evaluation signals

- relative_f1_improvement.csv contains exactly one row per search-strategy (entropy, maximum-probability) and threshold (highest, 95th, 75th percentile) combination
- Positive relative improvement values across combinations, replicating the ordering and magnitudes of Fig. 2A and Supplementary Figs. 4–5 (RF outperforming LR under random 20-cell initialization)
- Relative improvement is computed as (RF F1 − LR F1) / LR F1 and averaged over ten 50:50 stratified train/test splits and the SingleR/scmap annotation methods
- Classifiers are trained on exactly the first 20 principal components of scaled and centered logcount data, with 10 cells labeled per loop iteration
- Each run initializes from 20 randomly selected cells so results reflect realistic single-cell conditions without all cell types in the initial set

## Limitations

- 'Ground truth' cell type labels were taken from previous studies rather than manual annotation, creating ambiguity in what a ground-truth label means; the manual annotation process was only simulated using existing annotations
- Benchmarking chose many hyperparameters and annotation tools at the expense of the number of datasets investigated
- No statistical method was used to predetermine sample size, experiments were not randomized, and investigators were not blinded
- Results depend on random initial selection of 20 cells; behavior when the initial set is guaranteed to include all cell types was not established here

## Evidence

- [results] Using random initial selection of 20 cells, the random forest active learning model outperformed the logistic regression model, replicating prior findings (Fig. 2A, Supplementary Figs. 4 and 5).: [evidence span withheld — anchor status: not_found]
- [discussion] Each dataset was then randomly split into ten different train and test datasets using a 50:50 split while stratifying by cell type: "Each dataset was then randomly split into ten different train and test datasets using a 50:50 split while stratifying by cell type"
- [discussion] we trained a random forest and a logistic regression classifier using the first 20 principal components of scaled and centered logcount expression data as input using the caret R package: "we trained a random forest and a logistic regression classifier using the first 20 principal components of scaled and centered logcount expression data as input using the caret R package54"
- [discussion] we selected the set of 10 cells with the highest uncertainty to label using ground truth data. After this set of cells is labeled, the classifier is trained again and the loop repeats.: "we selected the set of 10 cells with the highest uncertainty to label using ground truth data. After this set of cells is labeled, the classifier is trained again and the loop repeats."
- [readme] An R package implementing the heuristic approaches introduced in this work may be found at https://github.com/camlab-bioml/leader: "An R package implementing the heuristic approaches introduced in this work may be found at https://github.com/camlab-bioml/leader"
