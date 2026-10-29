---
name: stratified-train-test-splitting
description: Use when benchmarking cell type annotation or cell selection methods
  on labeled single-cell expression data and repeated, class-balanced performance
  estimates (e.g. balanced accuracy, F1) are needed to compare strategies fairly.
when_to_use_negative:
- Data lack reliable ground-truth cell type labels, since stratification depends on
  the existing annotation (the authors relied on labels derived from previous studies
  and did not perform manual annotation)
- The goal is a single train/test split evaluation rather than the ten-replicate benchmark
  design used to produce the paper's figures and uncertainty ranges
- Extremely rare cell types with fewer members than needed for proportional 50:50
  representation, where stratified splits cannot preserve class balance in both partitions
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_2229
tools:
- name: leader
  role: R package whose active learning and adaptive reweighting cell selection loops
    are evaluated on each of the ten stratified train/test splits
  repo: https://github.com/camlab-bioml/leader
- name: caret
  role: Used to train the random forest and logistic regression classifiers on each
    training split and predict held-out test cells
- name: Seurat
  role: Provides the processed log-normalized, scaled Seurat objects whose cells are
    partitioned into the stratified splits
provenance:
  source_task_ids:
  - task_004
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-45198-y
    title: The impacts of active and self-supervised learning on efficient annotation
      of single-cell expression data
  source_content_hash: sha256:6305501aa51dc14e220831f12bced173e2986e65da251a17b806ea602f22b35e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/camlab-bioml/leader
  tools:
  - Seurat
  - caret
  - leader
  - leader (R package)
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# Stratified train-test splitting for single-cell benchmarking

## Summary

Randomly split each single-cell dataset into ten 50:50 train/test partitions while stratifying by cell type, providing the repeated evaluation framework used across the active learning, marker-initialization, and imbalance benchmarks of the Geuenich et al. 2024 study.

## When to use

Use when benchmarking cell type annotation or cell selection methods on labeled single-cell expression data and repeated, class-balanced performance estimates (e.g. balanced accuracy, F1) are needed to compare strategies fairly.

## When NOT to use

- Data lack reliable ground-truth cell type labels, since stratification depends on the existing annotation (the authors relied on labels derived from previous studies and did not perform manual annotation)
- The goal is a single train/test split evaluation rather than the ten-replicate benchmark design used to produce the paper's figures and uncertainty ranges
- Extremely rare cell types with fewer members than needed for proportional 50:50 representation, where stratified splits cannot preserve class balance in both partitions

## Inputs

- Processed single-cell expression dataset with ground-truth cell type labels per cell (Seurat object, SingleCellExperiment, or equivalent)
- Cell-level metadata table containing the ground-truth cell type annotation column
- Artificially constructed imbalanced/balanced two-type and five-type datasets (450/50, 250/250, 400/25x4, 100-per-type) for the imbalance benchmark

## Outputs

- Ten stratified 50:50 train/test partition pairs per dataset, with cell IDs assigned to each partition
- Per-split performance metrics (balanced accuracy, F1-score) recorded per selection method, dataset construction, and classifier
- Result CSVs such as imbalance_benchmark.csv, represented_cell_type_proportions.csv, and marker_vs_random_f1_improvement.csv keyed by train-test split

## How to apply

For each deposited single-cell expression dataset (e.g. the sc_mixology lung cancer cell lines, Tabula Sapiens, Liver Cell Atlas, CyTOF bone marrow data), randomly partition the cells into ten different train and test datasets using a 50:50 split while stratifying by cell type so every ground-truth cell type is represented proportionally in both partitions. Use each of the ten splits as an independent replicate of the downstream experiment: the training half seeds the active learning loop (initial 20 cells, then 10 highest-uncertainty cells per iteration for random forest / logistic regression classifiers from caret) or the LEADER adaptive reweighting selection, while the held-out test half is scored with balanced accuracy or F1 each round. For the imbalance benchmark, apply the same ten stratified splits to both artificial imbalanced constructions (450 vs 50 cells for two-type data; 400 vs 25-per-minority for five-type data) and their balanced counterparts (250/250 and 100-per-type) so imbalanced-versus-balanced changes in balanced accuracy are computed on matched partitions. Aggregate results across all ten splits per dataset to reproduce the per-split point estimates shown in Fig. 2B-2C, Fig. 3C and Supplementary Figs. 6-7 and 13-30.

## Related tools

- **leader** (R package whose active learning and adaptive reweighting cell selection loops are evaluated on each of the ten stratified train/test splits) — https://github.com/camlab-bioml/leader
- **caret** (Used to train the random forest and logistic regression classifiers on each training split and predict held-out test cells)
- **Seurat** (Provides the processed log-normalized, scaled Seurat objects whose cells are partitioned into the stratified splits)

## Examples

```
set.seed(42); splits <- lapply(1:10, function(i) createDataPartition(seu$cell_type, p = 0.5, list = FALSE))
```

## Evaluation signals

- Each dataset yields exactly ten distinct 50:50 train/test splits, with cell type proportions preserved across train and test partitions in every split
- Per-split metrics (balanced accuracy, F1) are recorded for every selection method, dataset construction, and classifier combination, matching the layout of Fig. 3C and Supplementary Figs. 13-30
- Marker-ranked versus random initialization F1 improvements (marker_vs_random_f1_improvement.csv) show positive values and the pattern of Fig. 2C, indicating splits support the expected comparison
- Imbalance benchmark CSV contains, for each method, the change in balanced accuracy between imbalanced and balanced datasets standardized by the balanced dataset, with values derived from matched stratified splits

## Limitations

- Ground truth cell type labels were taken from previous studies rather than manual annotation, so split quality inherits any ambiguity in the original labels
- No randomization or blinding of investigators was performed, and no statistical method was used to predetermine sample size, per the authors' reproducibility statement
- Benchmarking traded breadth of datasets for depth of tools and hyperparameters tested, so the ten-split design covers only the six curated datasets in this study

## Evidence

- [discussion] Each dataset was then randomly split into ten different train and test datasets using a 50:50 split while stratifying by cell type: "Each dataset was then randomly split into ten different train and test datasets using a 50:50 split while stratifying by cell type"
- [discussion] Active learning loop parameters: "we selected the set of 10 cells with the highest uncertainty to label using ground truth data. After this set of cells is labeled, the classifier is trained again and the loop repeats."
- [discussion] Reproducibility caveat: "No statistical method was used to predetermine sample size. No data were excluded from the analyses. The experiments were not randomized."
- [discussion] Imbalanced dataset construction: "we created two imbalanced datasets with 450 cells of one type and 50 of another, with cell types that were similar and distinct from each other"
