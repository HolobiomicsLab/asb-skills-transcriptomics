---
name: balanced-accuracy-benchmark-evaluation
description: Use when quantifying how cell type imbalance (e.g., 450:50 or 400:25
  per cohort) affects the relative performance of selection strategies for single-cell
  type annotation, with results laid out as in Fig. 3C and Supplementary Figs. 13-30.
when_to_use_negative:
- The goal is comparing full annotation algorithms such as SingleR, scmap, or SingleCellNet
  rather than benchmarking cell selection strategies for training-set construction
- The dataset has only one cell type or already contains exactly balanced classes,
  making the imbalance-vs-balance comparison undefined
- Ground truth labels are unavailable or unreliable, since the balanced accuracy metric
  and relabeling loop require verified labels from prior studies
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: leader
  role: 'R package implementing adaptive reweighting: clusters all cells with Seurat
    and samples equal numbers of cells per cluster (cluster_based mode) or per marker-assigned
    cell type (marker_based mode) to build balanced training subsets'
  repo: https://github.com/camlab-bioml/leader
- name: Seurat
  role: Clusters cells prior to equal-per-cluster sampling in adaptive reweighting;
    NormalizeData, FindVariableFeatures, and ScaleData prepare the input object
- name: caret
  role: Trains the random forest and logistic regression classifiers used to score
    uncertainty for active learning and to predict held-out test cell labels each
    round
- name: scuttle
  role: Computes logcounts normalized by size factor used as classifier input features
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-45198-y
    title: The impacts of active and self-supervised learning on efficient annotation
      of single-cell expression data
  source_content_hash: sha256:d4763cbf8fa2aed096dafb8437825d88351105917e0b13b11297108e1e4595c5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/camlab-bioml/leader
  tools:
  - Seurat
  - caret
  - leader
  - scuttle
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# balanced-accuracy-benchmark-evaluation

## Summary

Benchmark cell-selection strategies (active learning, adaptive reweighting via leader, random selection) under cell type imbalance by training random forest and logistic regression classifiers with caret on scaled, centered logcount features and comparing balanced accuracy between imbalanced and balanced artificial datasets, standardized by the balanced result.

## When to use

Use when quantifying how cell type imbalance (e.g., 450:50 or 400:25 per cohort) affects the relative performance of selection strategies for single-cell type annotation, with results laid out as in Fig. 3C and Supplementary Figs. 13-30.

## When NOT to use

- The goal is comparing full annotation algorithms such as SingleR, scmap, or SingleCellNet rather than benchmarking cell selection strategies for training-set construction
- The dataset has only one cell type or already contains exactly balanced classes, making the imbalance-vs-balance comparison undefined
- Ground truth labels are unavailable or unreliable, since the balanced accuracy metric and relabeling loop require verified labels from prior studies

## Inputs

- Processed Seurat objects with log-normalized gene expression and unique cell IDs in seu@meta.data (e.g., sc_mixology lung cancer cell lines, breast cancer cell-line Atlas, Tabula Sapiens, Liver Cell Atlas, PDAC snRNA-seq, CyTOF bone marrow data)
- Ground truth cell type labels per cell (e.g., cell_line_demuxlet column for sc_mixology)
- Optional marker gene yaml file for marker-aware adaptive reweighting (positive/negative markers per cell type)
- Dataset construction definitions: two-type (450/50 vs 250/250) and five-type (400/25x4 vs 100x5) imbalanced and balanced counterparts

## Outputs

- imbalance_benchmark.csv containing balanced accuracy per selection method, dataset construction, and classifier
- Standardized imbalance effect: change in balanced accuracy between imbalanced and balanced datasets divided by balanced accuracy (Fig. 3C layout, Supplementary Figs. 13-30)
- Balanced accuracy trajectories over active learning rounds for random forest and logistic regression classifiers

## How to apply

Construct artificial datasets: two-type datasets with 450 cells of one type and 50 of another using similar and distinct cell type pairs, plus 250/250 balanced counterparts; five-type datasets with 400 cells of one type and 25 of each remaining type, plus 100-per-type balanced counterparts, per cohort. Randomly split each into ten 50:50 train/test splits stratified by cell type, then run the active learning loop (label the 10 cells with highest classifier uncertainty at each iteration, retrain, repeat) starting from a 100-cell training set, alongside adaptive reweighting (Seurat clustering with equal cells sampled per cluster via leader) and random selection baselines. Train random forest and logistic regression classifiers with caret on the first 20 principal components of scaled, centered logcount expression (logcounts normalized via scuttle) to label held-out test cells each round, and compute balanced accuracy per method, dataset construction, and classifier. Finally compute the change in balanced accuracy between imbalanced and balanced datasets standardized by the balanced value and write results to imbalance_benchmark.csv.

## Related tools

- **leader** (R package implementing adaptive reweighting: clusters all cells with Seurat and samples equal numbers of cells per cluster (cluster_based mode) or per marker-assigned cell type (marker_based mode) to build balanced training subsets) — https://github.com/camlab-bioml/leader
- **Seurat** (Clusters cells prior to equal-per-cluster sampling in adaptive reweighting; NormalizeData, FindVariableFeatures, and ScaleData prepare the input object)
- **caret** (Trains the random forest and logistic regression classifiers used to score uncertainty for active learning and to predict held-out test cell labels each round)
- **scuttle** (Computes logcounts normalized by size factor used as classifier input features)

## Examples

```
adaptive_reweighting(seu, n_requested_cells = 40, mode = "cluster_based")
```

## Evaluation signals

- imbalance_benchmark.csv exists and contains balanced accuracy values for every combination of selection method (active learning, adaptive reweighting, random), dataset construction (imbalanced and balanced), and classifier (random forest, logistic regression)
- Each dataset has ten 50:50 train/test splits stratified by cell type, with imbalanced two-type data at 450:50 and five-type data at 400:25/25/25/25
- Standardized imbalance effect values are near zero when imbalanced and balanced versions perform identically; active learning shows a larger performance drop for similar cell type pairs than distinct ones
- Active learning starts from a 100-cell training set and selects exactly 10 highest-uncertainty cells per iteration before retraining
- Balanced accuracy (not plain accuracy) is the reported metric, ensuring majority-class dominance does not inflate scores in the 450:50 setting

## Limitations

- Ground truth cell type labels were taken from previous studies rather than manual annotation, creating fundamental ambiguity in what a ground-truth label means
- Manual annotation was only simulated using existing cell type annotations; no statistical predetermination of sample size, randomization, or blinding was used
- Dropout at the single-cell level could complicate annotation when specific markers are not expressed, and this was not addressed in the benchmarking
- Cell line markers derive from differential expression between cell lines, introducing circularity in marker-based (marker_based mode) selection
- Benchmarking traded breadth of datasets/modalities for depth of tools and hyperparameters tested

## Evidence

- [discussion] Active learning loop with 10 highest-uncertainty cells: "we selected the set of 10 cells with the highest uncertainty to label using ground truth data. After this set of cells is labeled, the classifier is trained again and the loop repeats."
- [discussion] caret classifiers on scaled logcount PCs: "we trained a random forest and a logistic regression classifier using the first 20 principal components of scaled and centered logcount expression data as input using the caret R package"
- [discussion] Imbalanced dataset construction: "we created two imbalanced datasets with 450 cells of one type and 50 of another, with cell types that were similar and distinct from each other"
- [discussion] Adaptive reweighting via Seurat clustering: "To accomplish this, we cluster with Seurat13. Next, we sampled cells from the resulting clusters such that the same number of cells is taken from each cluster."
- [abstract] leader package availability: "An R package implementing the heuristic approaches introduced in this work may be found at https://github.com/camlab-bioml/leader"
- [readme] leader README: cluster-based invocation: "adaptive_reweighting(seu, n_requested_cells = 40, mode = "cluster_based")"
