---
name: adaptive-reweighting-cluster-based-cell-selection
description: Use when a scRNA-seq or CyTOF training set is imbalanced (e.g. 450 vs
  50 cells or 400 vs 25 per minority type) and you need a cell-selection baseline
  to rebalance it against active learning or random selection.
when_to_use_negative:
- When ground-truth cell type labels are available for all cells and simple stratified
  random selection or supervised active learning with uncertainty sampling is feasible
  and preferred
- When cells cannot be meaningfully clustered (e.g. very few cells, or clusters do
  not correspond to cell types), since equal per-cluster sampling assumes cluster
  structure approximates cell type composition
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_2229
tools:
- name: leader
  role: 'R package implementing adaptive reweighting: clusters the Seurat object and
    samples equal numbers of cells per cluster (cluster_based mode) or per marker-assigned
    cell type (marker_based mode)'
  repo: https://github.com/camlab-bioml/leader
- name: Seurat
  role: Used to cluster all cells (NormalizeData, FindVariableFeatures, ScaleData)
    prior to equal-per-cluster cell selection
- name: caret
  role: Trains the random forest and logistic regression classifiers on scaled, centered
    logcount features to evaluate the balanced subsets by balanced accuracy
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-024-45198-y
    title: The impacts of active and self-supervised learning on efficient annotation
      of single-cell expression data
  source_content_hash: sha256:13d189c58f5ab26659fde27c1c6c34063a89310171256ffc55f6bd2825d37b0c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/camlab-bioml/leader
  tools:
  - Seurat
  - caret
  - leader
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# adaptive-reweighting-cluster-based-cell-selection

## Summary

A heuristic cell selection procedure that subsamples a single-cell dataset by clustering all cells (e.g. with Seurat) and drawing equal numbers of cells from each cluster, aiming to capture rare cell types and produce more balanced training sets for annotation benchmarks.

## When to use

Use when a scRNA-seq or CyTOF training set is imbalanced (e.g. 450 vs 50 cells or 400 vs 25 per minority type) and you need a cell-selection baseline to rebalance it against active learning or random selection.

## When NOT to use

- When ground-truth cell type labels are available for all cells and simple stratified random selection or supervised active learning with uncertainty sampling is feasible and preferred
- When cells cannot be meaningfully clustered (e.g. very few cells, or clusters do not correspond to cell types), since equal per-cluster sampling assumes cluster structure approximates cell type composition

## Inputs

- Processed, log-normalized Seurat object with scaled variable features and unique cell IDs in meta.data rownames
- Optional YAML marker file listing positive (and optional negative) marker genes per expected cell type (for the marker_based mode)

## Outputs

- Subset of cell_id's defining a more balanced subsampled training dataset
- Balanced accuracy values per selection method, dataset construction (imbalanced vs balanced), and classifier, written to imbalance_benchmark.csv (Fig. 3C / Supplementary Figs. 13-30 layout)

## How to apply

Start from a processed Seurat object with log-normalized expression whose most variable features (e.g. 2000 features via 'vst') have been identified and scaled; ensure unique cell IDs are the rownames of the meta.data dataframe. Cluster all cells with Seurat, then sample the same number of cells from each cluster using the leader package's adaptive_reweighting function in mode = "cluster_based", requesting the desired number of cells (e.g. n_requested_cells = 40). The returned cell_id's define the more balanced subsampled dataset, which is then fed into the benchmarking loop: random forest and logistic regression classifiers trained on scaled, centered logcount features (first 20 PCs via caret), evaluated by balanced accuracy across ten 50:50 stratified train/test splits. Rationale: unlike active learning, no ground-truth labels or iterative uncertainty queries are needed; clustering approximates cell type structure so equal per-cluster sampling recovers rare populations. A marker-aware variant can assign each cluster a putative cell type via mean marker gene expression from a YAML marker file before equal sampling.

## Related tools

- **leader** (R package implementing adaptive reweighting: clusters the Seurat object and samples equal numbers of cells per cluster (cluster_based mode) or per marker-assigned cell type (marker_based mode)) — https://github.com/camlab-bioml/leader
- **Seurat** (Used to cluster all cells (NormalizeData, FindVariableFeatures, ScaleData) prior to equal-per-cluster cell selection)
- **caret** (Trains the random forest and logistic regression classifiers on scaled, centered logcount features to evaluate the balanced subsets by balanced accuracy)

## Examples

```
adaptive_reweighting(seu, n_requested_cells = 40, mode = "cluster_based")
```

## Evaluation signals

- The Seurat object is log normalized, has variable features identified and scaled, and unique cell IDs as meta.data rownames before selection
- adaptive_reweighting returns a cell_id subset whose size matches n_requested_cells and in which minority/rare cell types are represented (more balanced composition than the input)
- Subsets serve as baselines in the active learning loop: balanced accuracy computed on each of ten 50:50 stratified train/test splits for random forest and logistic regression on the first 20 PCs
- Change in balanced accuracy between imbalanced and balanced datasets, standardized by the balanced dataset, is computed and written to imbalance_benchmark.csv matching Fig. 3C and Supplementary Figs. 13-30 layout

## Limitations

- Ground truth cell type labels were derived from previous studies rather than manual annotation, so there is ambiguity in what a ground-truth label means; manual annotation was only simulated using existing annotations
- Adaptive reweighting is a heuristic that assumes clustering structure reflects cell types; the article found active learning generally outperformed both random and adaptive reweighting in imbalanced settings
- Dropout at the single-cell level could complicate annotation when specific cell type markers are not expressed, and this phenomenon was not addressed in the benchmarking

## Evidence

- [readme] Leader generates subsampled datasets by clustering all cells and selecting the desired number of cells from these clusters.: "Leader generates subsampled datasets by clustering all cells and selecting the desired number of cells from these clusters."
- [discussion] To accomplish this, we cluster with Seurat13. Next, we sampled cells from the resulting clusters such that the same number of cells is taken from each cluster.: "To accomplish this, we cluster with Seurat13. Next, we sampled cells from the resulting clusters such that the same number of cells is taken from each cluster."
- [results] Active learning approaches generally outperformed both random and adaptive reweighting approaches in imbalanced settings: [evidence span withheld — anchor status: not_found]
- [discussion] we trained a random forest and a logistic regression classifier using the first 20 principal components of scaled and centered logcount expression data as input using the caret R package: "we trained a random forest and a logistic regression classifier using the first 20 principal components of scaled and centered logcount expression data as input using the caret R package54"
