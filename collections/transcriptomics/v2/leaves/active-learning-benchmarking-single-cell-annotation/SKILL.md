---
name: active-learning-benchmarking-single-cell-annotation
description: Use when a single-cell expression dataset has a marker file (author-provided,
  data-derived, or PanglaoDB top-20) and you need an informative initial 20-cell training
  set for an active learning annotation loop, benchmarked against random initialization.
when_to_use_negative:
- No marker file exists for the cell types in the dataset and none can be derived
  or sourced from PanglaoDB — the marker-ranked arm cannot be constructed
- The task is adaptive reweighting/subsampling of a full dataset for balance (cluster-based
  leader selection), not choosing a small initial active learning training set
- The ground-truth cell type labels are unknown or unverifiable, so classifier F1
  and uncertainty-based cell selection cannot be computed
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: leader
  role: R package implementing cell selection (cluster-based and marker-aware adaptive
    reweighting) used to run the active learning loop and supply the marker-informed
    initialization
  repo: https://github.com/camlab-bioml/leader
- name: caret
  role: Trains the random forest and logistic regression classifiers on the first
    20 principal components of scaled and centered logcount expression inside the
    active learning loop
- name: Seurat
  role: Provides the processed seurat object input format (NormalizeData, FindVariableFeatures,
    ScaleData) required by leader's adaptive_reweighting
- name: PanglaoDB
  role: Source of the 20 most sensitive markers per cell type used to build Tabula
    dataset marker files
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-45198-y
    title: The impacts of active and self-supervised learning on efficient annotation
      of single-cell expression data
  source_content_hash: sha256:fbb3889ed5b831a438ff815414794b4702a8d70048d9ccda61950c6d16ae3f0b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/camlab-bioml/leader
  tools:
  - PanglaoDB
  - Seurat
  - caret
  - leader
  - leader (R package)
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# active-learning-benchmarking-single-cell-annotation

## Summary

Initializes active learning for single-cell annotation by selecting the first 20 training cells by ranking average cell-type marker gene expression (rather than uniform random sampling), then running the leader/caret active learning loop to measure F1 gain and cell-type coverage.

## When to use

Use when a single-cell expression dataset has a marker file (author-provided, data-derived, or PanglaoDB top-20) and you need an informative initial 20-cell training set for an active learning annotation loop, benchmarked against random initialization.

## When NOT to use

- No marker file exists for the cell types in the dataset and none can be derived or sourced from PanglaoDB — the marker-ranked arm cannot be constructed
- The task is adaptive reweighting/subsampling of a full dataset for balance (cluster-based leader selection), not choosing a small initial active learning training set
- The ground-truth cell type labels are unknown or unverifiable, so classifier F1 and uncertainty-based cell selection cannot be computed

## Inputs

- Processed single-cell expression dataset (log-normalized; e.g. from Zenodo 10.5281/zenodo.10403475 or sc_mixology SingleCellExperiment objects)
- Marker file mapping expected cell types to marker genes (author-provided, data-derived, or 20 most sensitive PanglaoDB markers)
- Ground-truth cell type labels per cell (e.g. cell_line_demuxlet in sc_mixology colData)
- Active learning configuration: 20 initial cells, 10 highest-uncertainty cells per iteration, first 20 principal components

## Outputs

- represented_cell_type_proportions.csv — proportion of ground-truth cell types represented in the initial 20-cell set per train-test split (cf. Fig. 2B)
- marker_vs_random_f1_improvement.csv — F1-score of marker-ranked minus random initialization per split (cf. Fig. 2C, Supplementary Figs. 6-7)

## How to apply

Load each deposited single-cell dataset (e.g. Zenodo 10.5281/zenodo.10403475) with its marker file — author-provided, data-derived from differential expression between cell lines, or the 20 most sensitive PanglaoDB markers for Tabula — and split it into ten 50:50 train/test partitions stratified by cell type, mirroring the benchmark design. For each split build the initial 20-cell training set two ways: (a) rank candidate cells on average marker gene expression per cell type and take the top-ranked cells, and (b) uniform random selection. Run the active learning loop with the leader package: train random forest and logistic regression classifiers on the first 20 principal components of scaled, centered logcount expression (caret package), label the 10 highest-uncertainty cells per iteration using ground truth, and retrain. Record per split the proportion of ground-truth cell types represented in the initial 20-cell set and the marker-ranked minus random F1-score difference. Rationale: marker-ranked initialization represents more cell types in the small initial set, which is where the benefit is largest when few cells are labeled.

## Related tools

- **leader** (R package implementing cell selection (cluster-based and marker-aware adaptive reweighting) used to run the active learning loop and supply the marker-informed initialization) — https://github.com/camlab-bioml/leader
- **caret** (Trains the random forest and logistic regression classifiers on the first 20 principal components of scaled and centered logcount expression inside the active learning loop)
- **Seurat** (Provides the processed seurat object input format (NormalizeData, FindVariableFeatures, ScaleData) required by leader's adaptive_reweighting)
- **PanglaoDB** (Source of the 20 most sensitive markers per cell type used to build Tabula dataset marker files)

## Examples

```
markers <- get_markers(system.file("extdata", "markers.yml", package = "leader")); adaptive_reweighting(seu, markers, n_requested_cells = 20, mode = "marker_based")
```

## Evaluation signals

- Marker-ranked initialization should represent a higher proportion of ground-truth cell types in the 20-cell initial set than random selection across the ten 50:50 stratified splits (Fig. 2B pattern)
- marker_vs_random_f1_improvement.csv should show predominantly positive F1 differences, especially in the earliest labeling iterations when few cells are labeled (Fig. 2C pattern)
- Both CSVs must contain one row per train-test split (10 splits per dataset) with the expected field names and plausible value ranges (proportions in [0,1]; F1 improvements within [-1,1])
- Classifiers must be trained on exactly the first 20 principal components of scaled and centered logcount data, and 10 highest-uncertainty cells must be labeled per iteration
- Verify marker files parse with leader's get_markers() and follow the yaml structure with cell_types, positive and optional negative markers

## Limitations

- Circularity in marker definition, especially for cell line data where markers derive from differential expression between the same cell lines being classified
- Ground truth cell type labels come from previous studies, and manual annotation was simulated rather than performed
- Dropout at the single-cell level may cause specific markers not to be expressed, complicating marker-ranked ranking — not addressed in the benchmarking
- Marker files may be uncertain sources (e.g. snRNA-Seq marker file adapted from author personal communication)

## Evidence

- [intro] Research question (iv): can prior knowledge of cell type marker genes be used to improve the initial training set?: "(iv) can prior knowledge of cell type marker genes be used to improve the initial training set?"
- [abstract] Abstract finding on marker knowledge improving accuracy: "In addition, we demonstrate that having prior knowledge of cell type markers improves annotation accuracy."
- [discussion] Train/test split design: "Each dataset was then randomly split into ten different train and test datasets using a 50:50 split while stratifying by cell type"
- [discussion] Active learning loop mechanics: "we selected the set of 10 cells with the highest uncertainty to label using ground truth data. After this set of cells is labeled, the classifier is trained again and the loop repeats."
- [discussion] Classifier training specification: "we trained a random forest and a logistic regression classifier using the first 20 principal components of scaled and centered logcount expression data as input using the caret R package54"
- [readme] Marker file structure and get_markers usage: "The first row must contain the word `cell_types`. Each indented element is an expected cell type with required positive labels and optional negative labels."
