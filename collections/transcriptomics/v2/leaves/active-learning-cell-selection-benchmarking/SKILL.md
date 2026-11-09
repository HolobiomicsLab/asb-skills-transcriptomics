---
name: active-learning-cell-selection-benchmarking
description: Use when building small labeled training sets (e.g., 100–500 cells) from
  single-cell expression data and deciding between active learning, adaptive reweighting,
  and random sampling for annotation model training.
when_to_use_negative:
- The goal is self-supervised training-set expansion (pseudo-labeling confident predictions)
  rather than comparing uncertainty-based active learning against cluster-based and
  random selection
- Input is not single-cell expression data (e.g., bulk RNA mixtures or proportional
  data), since the selection methods operate on individual cell clusters and per-cell
  marker expression
- Only one classifier or one dataset will be evaluated, since the skill's conclusion
  ('no single cell selection method is consistently best') derives from aggregating
  ranks across 6 datasets and 6 annotation methods
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0091
tools:
- name: leader
  role: R package implementing cluster-based and marker-informed adaptive reweighting
    cell selection on a processed Seurat object
  repo: https://github.com/camlab-bioml/leader
- name: Seurat
  role: Clustering of cells prior to equal-per-cluster sampling in adaptive reweighting;
    NormalizeData/FindVariableFeatures/ScaleData preprocessing for leader input
- name: SingleR
  role: Annotation classifier trained on each candidate training set with default
    parameters
- name: scmap
  role: Annotation classifier trained on each candidate training set
- name: SingleCellNet
  role: Annotation classifier trained on each candidate training set with default
    parameters
- name: caret
  role: Trains the random forest classifier on the first 20 principal components of
    scaled and centered logcounts within the active learning loop
- name: scuttle
  role: Computes logcounts and size-factor normalization of the expression matrices
    before splitting and selection
- name: HDCytoData
  role: Downloads the CyTOF bone marrow dataset (24 cell types, 10 mice) used as one
    of the six benchmarks
- name: scran
  role: findMarkers-based derivation of cell-line marker definitions used in marker-aware
    selection
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-45198-y
    title: The impacts of active and self-supervised learning on efficient annotation
      of single-cell expression data
  source_content_hash: sha256:51db1639d6f49da9ebcc882584277054677e0c2068445f2487702d70ab6a709c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/camlab-bioml/leader
  tools:
  - HDCytoData
  - Seurat
  - SingleCellNet
  - SingleR
  - caret
  - leader
  - leader (R package)
  - scmap
  - scran
  - scuttle
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# active-learning-cell-selection-benchmarking

## Summary

A benchmarking protocol for comparing cell selection strategies—active learning (maximum-entropy and lowest-max-probability search), cluster-based and marker-informed adaptive reweighting, and random selection—when constructing training sets for single-cell type annotation classifiers across RNA-Seq and CyTOF datasets.

## When to use

Use when building small labeled training sets (e.g., 100–500 cells) from single-cell expression data and deciding between active learning, adaptive reweighting, and random sampling for annotation model training.

## When NOT to use

- The goal is self-supervised training-set expansion (pseudo-labeling confident predictions) rather than comparing uncertainty-based active learning against cluster-based and random selection
- Input is not single-cell expression data (e.g., bulk RNA mixtures or proportional data), since the selection methods operate on individual cell clusters and per-cell marker expression
- Only one classifier or one dataset will be evaluated, since the skill's conclusion ('no single cell selection method is consistently best') derives from aggregating ranks across 6 datasets and 6 annotation methods

## Inputs

- Six public single-cell datasets: breast cancer cell-line Atlas (figshare), five lung cancer cell lines from sc_mixology, untreated PDAC snRNA-Seq (SCP1089, subsampled to 6,000 cells), CyTOF bone marrow dataset via HDCytoData, Liver Cell Atlas, Tabula Sapiens release 1.0
- Marker files: 20 most sensitive PanglaoDB markers for Tabula datasets, author-provided markers for the snRNA-Seq dataset (positive/negative markers in YAML for leader)
- Processed Seurat objects (log-normalized, variable features identified and scaled) for leader input
- Ground truth cell type labels in colData (e.g., cell_line_demuxlet for sc_mixology)

## Outputs

- fig3b_selection_method_ranks.csv — median balanced accuracy and sensitivity ranks per selection method across datasets and classifiers
- Trained annotation models per dataset x classifier x selection-method x training-set-size combination (100/250/500 cells)
- Balanced accuracy and sensitivity metrics on held-out test cells for every combination

## How to apply

Preprocess each expression matrix by computing logcounts and size-factor normalization with scuttle, then randomly split each dataset into ten 50:50 train/test partitions stratified by cell type. Construct training sets of 100, 250, and 500 labeled cells per split using maximum-entropy and lowest-max-probability active learning (iteratively labeling the 10 most uncertain cells, then retraining the classifier), cluster-based adaptive reweighting (Seurat clustering with equal numbers of cells sampled per cluster, implemented in the leader R package via mode='cluster_based'), marker-informed adaptive reweighting (averaging user-provided marker gene expression per cell type and cluster before equal sampling, mode='marker_based'), and random selection. Train SingleR, scmap, SingleCellNet, CyTOF-LDA, and caret-based random forest (on the first 20 principal components of scaled and centered logcounts) plus SVM classifiers on each training set. Evaluate balanced accuracy and sensitivity on held-out test cells for every dataset x classifier x selection-method combination, then aggregate results into median balanced accuracy and sensitivity ranks per selection method across datasets and classifiers.

## Related tools

- **leader** (R package implementing cluster-based and marker-informed adaptive reweighting cell selection on a processed Seurat object) — https://github.com/camlab-bioml/leader
- **Seurat** (Clustering of cells prior to equal-per-cluster sampling in adaptive reweighting; NormalizeData/FindVariableFeatures/ScaleData preprocessing for leader input)
- **SingleR** (Annotation classifier trained on each candidate training set with default parameters)
- **scmap** (Annotation classifier trained on each candidate training set)
- **SingleCellNet** (Annotation classifier trained on each candidate training set with default parameters)
- **caret** (Trains the random forest classifier on the first 20 principal components of scaled and centered logcounts within the active learning loop)
- **scuttle** (Computes logcounts and size-factor normalization of the expression matrices before splitting and selection)
- **HDCytoData** (Downloads the CyTOF bone marrow dataset (24 cell types, 10 mice) used as one of the six benchmarks)
- **scran** (findMarkers-based derivation of cell-line marker definitions used in marker-aware selection)

## Examples

```
markers <- get_markers(system.file("extdata", "markers.yml", package = "leader")); adaptive_reweighting(seu, markers, n_requested_cells = 40, mode = "marker_based")
```

## Evaluation signals

- The aggregated fig3b_selection_method_ranks.csv reproduces the Fig. 3B pattern: active learning outperforms random selection and adaptive reweighting across most datasets, with adaptive reweighting competitive in some situations
- All ten train/test splits per dataset are 50:50 and stratified by cell type, and training sets contain exactly 100, 250, or 500 labeled cells
- Balanced accuracy and sensitivity are computed on held-out test cells for every dataset x classifier x selection-method combination before rank aggregation
- Active learning loop iterates by labeling the 10 cells with highest uncertainty, retraining, and repeating until the requested training-set size is reached
- Marker-aware leader runs require a valid YAML marker file whose first line is 'cell_types', with positive (and optional negative) markers per expected cell type

## Limitations

- Ground truth cell type labels were taken from previous studies rather than manual annotation, creating fundamental ambiguity in what a ground-truth label means
- There is circularity in the way some cell type markers are defined, especially for the cell line data, as these derive from differential expression between the cell lines
- The experiments were not randomized and investigators were not blinded; no statistical method was used to predetermine sample size
- Dropout at the single-cell level could complicate annotation if specific cell type markers are not expressed; the snRNA-Seq marker file was adapted from author personal communication rather than a published resource
- Benchmarking prioritized many hyperparameters and annotation tools at the expense of investigating a large number of datasets

## Evidence

- [discussion] Each dataset was then randomly split into ten different train and test datasets using a 50:50 split while stratifying by cell type: "Each dataset was then randomly split into ten different train and test datasets using a 50:50 split while stratifying by cell type"
- [discussion] Active learning uncertainty loop: "we selected the set of 10 cells with the highest uncertainty to label using ground truth data. After this set of cells is labeled, the classifier is trained again and the loop repeats."
- [abstract] Adaptive reweighting introduced and competitive: "We introduce adaptive reweighting, a heuristic procedure tailored to single-cell data—including a marker-aware version—that shows competitive performance with existing approaches."
- [readme] Leader cluster-based selection mechanism: "Leader generates subsampled datasets by clustering all cells and selecting the desired number of cells from these clusters."
- [discussion] Preprocessing normalization step: "we calculated logcounts and normalized with the size factor using the scuttle R package52"
