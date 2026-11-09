---
name: single-cell-active-learning-simulation
description: Use when benchmarking how many cells must be manually annotated to reach
  high annotation F1, comparing active learning classifiers/search strategies, or
  testing whether unlabeled novel cell types are flagged by predictive entropy.
when_to_use_negative:
- The goal is self-supervised label propagation without oracle labeling — use the
  self-training variant (label most confidently predicted cells by entropy) instead
  of active learning with ground-truth relabeling
- Expression data are raw counts, not log-normalized with variable features scaled
  — the classifiers require PCA on scaled, centered logcount data
- The dataset has no reliable ground truth labels to simulate the manual annotation
  oracle
edam_operation: http://edamontology.org/operation_3919
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: leader
  role: R package implementing the active learning benchmarking workflow, adaptive
    reweighting cell selection, and uncertainty-based cell selection loops
  repo: https://github.com/camlab-bioml/leader
- name: caret
  role: Trains the random forest and logistic regression classifiers on the first
    20 principal components of scaled logcount data
- name: SingleR
  role: Single-cell annotation method combined with the active learning loop for performance
    comparison
- name: scmap
  role: Single-cell annotation method combined with the active learning loop for performance
    comparison
- name: scran
  role: Marker gene derivation (findMarkers) used for marker-informed training set
    construction
- name: Seurat
  role: Clustering for adaptive reweighting cell selection feeding the balanced training
    sets
provenance:
  source_task_ids:
  - task_005
  - task_002
  source_papers:
  - doi: 10.1038/s41467-024-45198-y
    title: The impacts of active and self-supervised learning on efficient annotation
      of single-cell expression data
  source_content_hash: sha256:18e72ef72458d7ca915683ca12eb82145c768e73cfbc13e537dde129e056ebff
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
  - scran
  edam_operation: http://edamontology.org/operation_3919
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# single-cell-active-learning-simulation

## Summary

Simulates manual cell type annotation as an active learning loop on single-cell expression data: classifiers (random forest, logistic regression) trained on the first 20 PCs of scaled logcounts iteratively select the 10 unlabeled cells with highest predictive uncertainty for ground-truth labeling and retraining, using the leader benchmarking framework.

## When to use

Use when benchmarking how many cells must be manually annotated to reach high annotation F1, comparing active learning classifiers/search strategies, or testing whether unlabeled novel cell types are flagged by predictive entropy.

## When NOT to use

- The goal is self-supervised label propagation without oracle labeling — use the self-training variant (label most confidently predicted cells by entropy) instead of active learning with ground-truth relabeling
- Expression data are raw counts, not log-normalized with variable features scaled — the classifiers require PCA on scaled, centered logcount data
- The dataset has no reliable ground truth labels to simulate the manual annotation oracle

## Inputs

- Processed single-cell expression matrix (logcount, log-normalized; Seurat or SingleCellExperiment object)
- Ground truth cell type labels per cell
- Optional marker gene file (yaml of positive/negative markers per cell type) for marker-ranked training set fills
- Active learning configuration: initial training set size (20 cells), cells labeled per iteration (10), search strategy and percentile thresholds

## Outputs

- Per-iteration F1 scores for each classifier/search-strategy/threshold combination
- relative_f1_improvement.csv with one row per search-strategy/threshold combination giving (RF F1 − LR F1)/LR F1 averaged over ten splits and annotation methods
- Summary CSV of median scaled predictive entropy for removed versus present cell types, by cohort and classifier
- Trained classifier probability predictions and entropy scores for unlabeled cells

## How to apply

Retrieve processed expression matrices and ground truth labels (e.g., from Zenodo 10.5281/zenodo.10403475), then randomly split each dataset into ten 50:50 train/test splits stratified by cell type and initialize each run with a 20-cell training set. Train a random forest and a logistic regression classifier (caret R package) on the first 20 principal components of scaled and centered logcount expression data within the leader package workflow. At each loop iteration, rank unlabeled cells by predictive uncertainty — entropy and maximum-probability search at the highest, 95th percentile, and 75th percentile thresholds — select the 10 cells with the highest uncertainty, label them with ground truth, and retrain. To test detection of unlabeled novel cell types, construct 20-cell initial training sets in which the target type occurs 0, 1, 2, or 3 times (random or marker-gene ranked fills) and compare scaled predictive entropies for that type; excluded types show generally higher entropy, most drastically for logistic regression. Compute F1 scores per classifier/search/threshold combination across iterations and derive relative improvement as (RF F1 − LR F1) standardized by the LR F1, averaged over splits and annotation methods (SingleR, scmap).

## Related tools

- **leader** (R package implementing the active learning benchmarking workflow, adaptive reweighting cell selection, and uncertainty-based cell selection loops) — https://github.com/camlab-bioml/leader
- **caret** (Trains the random forest and logistic regression classifiers on the first 20 principal components of scaled logcount data)
- **SingleR** (Single-cell annotation method combined with the active learning loop for performance comparison)
- **scmap** (Single-cell annotation method combined with the active learning loop for performance comparison)
- **scran** (Marker gene derivation (findMarkers) used for marker-informed training set construction)
- **Seurat** (Clustering for adaptive reweighting cell selection feeding the balanced training sets)

## Examples

```
markers <- get_markers(system.file("extdata", "markers.yml", package = "leader")); adaptive_reweighting(seu, markers, n_requested_cells = 40, mode = "marker_based")
```

## Evaluation signals

- With random initial selection of 20 cells, the random forest active learning model outperforms logistic regression: relative_f1_improvement.csv shows positive relative improvement values matching Fig. 2A and Supplementary Figs. 4-5 ordering and magnitudes
- relative_f1_improvement.csv contains one row per search-strategy/threshold combination (entropy and maximum-probability at highest/95th/75th percentile)
- For the unlabeled cell type experiment, median scaled entropy for excluded (0-cell) cell types exceeds that when 1, 2, or 3 cells are included, most drastically for logistic regression
- F1 scores increase monotonically (or near-monotonically) with active learning iterations as labeled cells accumulate
- Train/test splits are stratified by cell type at 50:50 with ten replicates, and each run initializes from exactly 20 cells

## Limitations

- Ground truth labels were taken from previous studies rather than manual annotation, so there is fundamental ambiguity in what a ground-truth cell type label means
- The manual annotation process was only simulated using existing cell type annotations, not performed
- Marker definitions can be circular, especially for cell line data where markers derive from differential expression between the cell lines
- No sample size predetermination, randomization, or blinding of investigators; benchmarking traded dataset breadth for many tools/hyperparameters tested
- The optimal certainty threshold for cell selection in self-training was not identified and is left to future work

## Evidence

- [methods] Each dataset was then randomly split into ten different train and test datasets using a 50:50 split while stratifying by cell type: "Each dataset was then randomly split into ten different train and test datasets using a 50:50 split while stratifying by cell type"
- [methods] we selected the set of 10 cells with the highest uncertainty to label using ground truth data. After this set of cells is labeled, the classifier is trained again and the loop repeats.: "we selected the set of 10 cells with the highest uncertainty to label using ground truth data. After this set of cells is labeled, the classifier is trained again and the loop repeats."
- [methods] we trained a random forest and a logistic regression classifier using the first 20 principal components of scaled and centered logcount expression data as input using the caret R package: "we trained a random forest and a logistic regression classifier using the first 20 principal components of scaled and centered logcount expression data as input using the caret R package54"
- [discussion] relying on manual annotation for this work was not feasible due to the large number of experiments performed. Instead, we simulated the manual annotation process using existing cell type annotations: "relying on manual annotation for this work was not feasible due to the large number of experiments performed. Instead, we simulated the manual annotation process using existing cell type annotations"
- [readme] the average expression of a set of marker genes provided by the user is calculated for each cell type and cluster individually: "In the marker aware version, each cluster is putatively assigned to a cell type using the average expression of marker genes for each expected cell type provided by the user."
