---
name: imbalanced-dataset-construction
description: Use when benchmarking cell selection or annotation methods under controlled
  cell type imbalance, e.g. building artificial 450:50 or 400/25 datasets from cohorts
  with known ground-truth cell type labels.
when_to_use_negative:
- The goal is to evaluate annotation on naturally imbalanced data rather than a controlled,
  constructed imbalance experiment — the artificial 450:50 / 400:25 designs answer
  a specific relative-performance question
- Cell type labels are uncertain or absent; these constructions require reliable ground-truth
  labels from previous studies for stratified splitting and balanced accuracy computation
- Only balanced data analysis is needed — constructing imbalanced counterparts adds
  cost without benefit unless imbalance effects are the research question
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_3308
tools:
- name: leader
  role: R package providing adaptive reweighting cell selection (cluster-based and
    marker-based modes) used as the balanced-subsampling strategy benchmarked against
    active learning and random selection
  repo: https://github.com/camlab-bioml/leader
- name: Seurat
  role: Clusters cells within the adaptive reweighting step so equal numbers of cells
    can be sampled per cluster
- name: caret
  role: Trains the random forest and logistic regression classifiers on the first
    20 principal components of scaled and centered logcount expression data at each
    active learning round
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-45198-y
    title: The impacts of active and self-supervised learning on efficient annotation
      of single-cell expression data
  source_content_hash: sha256:363e1f12c6b53d8415d7f1f6d70e67d79a0adf23e6f63145fd3a42f5bac95efa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/camlab-bioml/leader
  tools:
  - Seurat
  - caret
  - leader
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# imbalanced-dataset-construction

## Summary

Construct artificial imbalanced and balanced single-cell datasets (two-type 450:50 and five-type 400/25-per-minority designs, plus balanced counterparts) to quantify how cell type imbalance and cell type similarity affect selection strategies (active learning, adaptive reweighting, random) for cell type annotation.

## When to use

Use when benchmarking cell selection or annotation methods under controlled cell type imbalance, e.g. building artificial 450:50 or 400/25 datasets from cohorts with known ground-truth cell type labels.

## When NOT to use

- The goal is to evaluate annotation on naturally imbalanced data rather than a controlled, constructed imbalance experiment — the artificial 450:50 / 400:25 designs answer a specific relative-performance question
- Cell type labels are uncertain or absent; these constructions require reliable ground-truth labels from previous studies for stratified splitting and balanced accuracy computation
- Only balanced data analysis is needed — constructing imbalanced counterparts adds cost without benefit unless imbalance effects are the research question

## Inputs

- Processed single-cell datasets with ground-truth cell type labels per cohort (e.g. breast cancer cell lines, sc_mixology lung cancer cell lines, Tabula Sapiens, Liver Cell Atlas, PDAC snRNA-seq, CyTOF bone marrow)
- Normalized logcount expression matrices (Seurat objects, log-normalized with variable features scaled)
- Ground-truth cell type label annotations per cell

## Outputs

- Artificial imbalanced datasets (450:50 two-type; 400/25-per-minority five-type) and balanced counterparts (250:250; 100-per-type)
- Ten stratified 50:50 train/test splits per constructed dataset
- Balanced accuracy values per selection method, dataset construction, and classifier
- imbalance_benchmark.csv with imbalance-vs-balanced standardized accuracy change matching Fig. 3C and Supplementary Figs. 13-30

## How to apply

Build artificial datasets per cohort: for two-type designs create 450 cells of one type and 50 of another using both similar and distinct cell type pairs, plus their 250:250 balanced counterparts; for five-type designs create 400 cells of one type with 25 of each remaining type, plus 100-per-type balanced counterparts. Randomly split each dataset into ten 50:50 train/test splits stratified by cell type. Starting from a 100-cell training set, run the active learning loop labeling the 10 cells with highest classifier uncertainty per iteration, alongside adaptive reweighting (Seurat clustering with equal cells sampled per cluster via leader) and random selection baselines. Train random forest and logistic regression classifiers (caret, on scaled, centered logcount expression) to label held-out test cells each round, then compute balanced accuracy per method, dataset construction, and classifier. Finally compute the change in balanced accuracy between imbalanced and balanced datasets standardized by the balanced value, and write results to imbalance_benchmark.csv matching the Fig. 3C and Supplementary Figs. 13-30 layout.

## Related tools

- **leader** (R package providing adaptive reweighting cell selection (cluster-based and marker-based modes) used as the balanced-subsampling strategy benchmarked against active learning and random selection) — https://github.com/camlab-bioml/leader
- **Seurat** (Clusters cells within the adaptive reweighting step so equal numbers of cells can be sampled per cluster)
- **caret** (Trains the random forest and logistic regression classifiers on the first 20 principal components of scaled and centered logcount expression data at each active learning round)

## Examples

```
adaptive_reweighting(seu, n_requested_cells = 40, mode = "cluster_based")
```

## Evaluation signals

- Each constructed dataset matches the specified design exactly: two-type 450:50 (and 250:250 balanced) for similar and distinct cell type pairs; five-type 400 + 4x25 (and 100-per-type balanced)
- All splits are 50:50 and stratified by cell type; there are ten replicate splits per dataset construction
- Active learning iterations start from a 100-cell training set and label exactly 10 highest-uncertainty cells per round before retraining
- Balanced accuracy is computed for every combination of selection method (active learning, adaptive reweighting, random), dataset construction (imbalanced vs balanced), and classifier (random forest, logistic regression)
- Output CSV reproduces the standardized (imbalanced minus balanced)/balanced accuracy change layout of Fig. 3C and Supplementary Figs. 13-30

## Limitations

- Ground truth cell type labels were taken from previous studies rather than manual annotation, introducing fundamental ambiguity in what a ground-truth label means in single-cell biology
- Manual annotation was only simulated using existing cell type annotations; the experiments were not randomized and no sample size predetermination was used
- Benchmarking chose to cover many hyperparameters and annotation tools at the expense of investigating a large number of datasets

## Evidence

- [discussion] Two-type imbalanced design: "we created two imbalanced datasets with 450 cells of one type and 50 of another, with cell types that were similar and distinct from each other"
- [discussion] Train/test split design: "Each dataset was then randomly split into ten different train and test datasets using a 50:50 split while stratifying by cell type"
- [discussion] Active learning loop: "we selected the set of 10 cells with the highest uncertainty to label using ground truth data. After this set of cells is labeled, the classifier is trained again and the loop repeats."
- [discussion] Adaptive reweighting sampling: "we cluster with Seurat13. Next, we sampled cells from the resulting clusters such that the same number of cells is taken from each cluster."
- [discussion] Classifier training with caret: "we trained a random forest and a logistic regression classifier using the first 20 principal components of scaled and centered logcount expression data as input using the caret R package54"
- [readme] LEADER cluster-based selection: "Leader generates subsampled datasets by clustering all cells and selecting the desired number of cells from these clusters."
- [discussion] Ground truth label caveat: "we relied here on “ground truth” cell type labels derived from previous studies"
