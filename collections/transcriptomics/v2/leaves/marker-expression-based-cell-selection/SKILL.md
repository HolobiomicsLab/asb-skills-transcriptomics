---
name: marker-expression-based-cell-selection
description: Use when building the initial training set for active learning cell type
  annotation and a marker file (author-provided, data-derived, or PanglaoDB-derived)
  is available; especially beneficial when few cells will be labeled initially.
when_to_use_negative:
- No prior marker knowledge exists for the expected cell types — the article's adaptive
  reweighting without markers (cluster-based mode, Seurat clustering with equal per-cluster
  sampling) is the intended fallback
- Markers are unreliable or highly circular (e.g. cell line datasets where markers
  are derived by differential expression between the very cell lines being annotated)
- The task is balanced subsampling of an already-annotated dataset rather than selecting
  an initial annotation training set
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: leader
  role: 'Performs the marker-aware cell selection: get_markers() reads the marker
    YAML and adaptive_reweighting(..., mode = "marker_based") assigns clusters to
    cell types by average marker expression and selects cells'
  repo: https://github.com/camlab-bioml/leader
- name: caret
  role: Trains the random forest and logistic regression classifiers on the first
    20 principal components used to evaluate marker-ranked vs random initial selection
    via F1-score
- name: Seurat
  role: Prepares the input object (NormalizeData, FindVariableFeatures, ScaleData)
    required before leader cell selection
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-024-45198-y
    title: The impacts of active and self-supervised learning on efficient annotation
      of single-cell expression data
  source_content_hash: sha256:60bc70fd3a620c203f4d4fb044b9064679c88a35fa4143be1977ae5304550b83
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
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# marker-expression-based-cell-selection

## Summary

Selects the initial labeled training set for active learning single-cell annotation by ranking candidate cells on average marker gene expression per expected cell type (positive and negative markers), rather than random selection, so that more ground-truth cell types are represented in the first 20 labeled cells.

## When to use

Use when building the initial training set for active learning cell type annotation and a marker file (author-provided, data-derived, or PanglaoDB-derived) is available; especially beneficial when few cells will be labeled initially.

## When NOT to use

- No prior marker knowledge exists for the expected cell types — the article's adaptive reweighting without markers (cluster-based mode, Seurat clustering with equal per-cluster sampling) is the intended fallback
- Markers are unreliable or highly circular (e.g. cell line datasets where markers are derived by differential expression between the very cell lines being annotated)
- The task is balanced subsampling of an already-annotated dataset rather than selecting an initial annotation training set

## Inputs

- Processed log-normalized Seurat object with unique cell IDs in meta.data and scaled variable features
- Marker YAML file listing positive (and optional negative) marker genes per expected cell type (e.g. CD3/CD8 for CD8 T cells, B220 as a negative marker)
- Single-cell expression dataset with ground-truth cell type labels (e.g. sc_mixology, Tabula Sapiens, CyTOF via HDCytoData)
- Ten 50:50 train/test partitions stratified by cell type per dataset

## Outputs

- Marker-ranked initial 20-cell training set (selected cell IDs)
- represented_cell_type_proportions.csv (proportion of ground-truth cell types represented in the initial set per split, cf. Fig. 2B)
- marker_vs_random_f1_improvement.csv (F1-score difference of marker-ranked minus random initialization per split, cf. Fig. 2C and Supplementary Figs. 6-7)

## How to apply

Prepare a processed, log-normalized Seurat object (NormalizeData, FindVariableFeatures with vst nfeatures=2000, ScaleData) whose meta.data rows carry unique cell IDs. Encode expected cell types in a YAML marker file with required positive markers and optional negative markers per cell type, then read it with leader's get_markers() and run adaptive_reweighting with mode = "marker_based"; leader computes average marker expression for each cell type (and cluster) and putsatively assigns cell types so selected cells are spread across expected types. In the benchmark design, the initial 20-cell set is built twice per split — marker-ranked vs uniform random — on each of ten 50:50 train/test partitions stratified by cell type, then active learning proceeds with random forest and logistic regression (caret, first 20 PCs of scaled/centered logcounts), labeling the 10 highest-uncertainty cells per iteration. Rationale: marker-ranked initialization represents more cell types up front, which raises F1 classification scores compared to random selection, most markedly when few cells are labeled.

## Related tools

- **leader** (Performs the marker-aware cell selection: get_markers() reads the marker YAML and adaptive_reweighting(..., mode = "marker_based") assigns clusters to cell types by average marker expression and selects cells) — https://github.com/camlab-bioml/leader
- **caret** (Trains the random forest and logistic regression classifiers on the first 20 principal components used to evaluate marker-ranked vs random initial selection via F1-score)
- **Seurat** (Prepares the input object (NormalizeData, FindVariableFeatures, ScaleData) required before leader cell selection)

## Examples

```
markers <- get_markers(system.file("extdata", "markers.yml", package = "leader")); adaptive_reweighting(seu, markers, n_requested_cells = 20, mode = "marker_based")
```

## Evaluation signals

- The marker-ranked initial 20-cell set represents a higher proportion of ground-truth cell types than the random set across the ten 50:50 stratified splits (Fig. 2B pattern)
- Per-split F1 improvement (marker-ranked minus random) is predominantly positive, especially when few cells are labeled (Fig. 2C, Supplementary Figs. 6-7 pattern)
- Output CSVs contain the expected fields with plausible value ranges (proportions in [0,1]; F1 improvements within [-1,1])
- The marker YAML parses with get_markers() and each expected cell type carries at least one positive marker; the 'cell_types' header is present as the first row
- Selected cell IDs returned by adaptive_reweighting resolve to unique rownames of seu@meta.data and match n_requested_cells

## Limitations

- Circularity in marker definition, especially for cell line data where markers derive from differential expression between the cell lines themselves
- A marker file for the snRNA-Seq dataset was adapted from personal communication with authors rather than a published resource, and Tabula markers were the 20 most sensitive PanglaoDB markers — marker quality directly conditions the gain
- Single-cell dropout could complicate cell type annotation if specific cell type markers are not expressed; this was not addressed in the benchmarking

## Evidence

- [methods] Marker-aware adaptive reweighting assigns clusters via marker expression: "the average expression of a set of marker genes provided by the user is calculated for each cell type and cluster individually"
- [abstract] Marker-ranked initialization outperforms random selection: "In addition, we demonstrate that having prior knowledge of cell type markers improves annotation accuracy."
- [methods] Benchmark split design: "Each dataset was then randomly split into ten different train and test datasets using a 50:50 split while stratifying by cell type"
- [discussion] Classifier used to score initial selection strategies: "we trained a random forest and a logistic regression classifier using the first 20 principal components of scaled and centered logcount expression data as input using the caret R package54"
- [readme] Leader marker-aware selection mechanism: "In the marker aware version, each cluster is putatively assigned to a cell type using the average expression of marker genes for each expected cell type provided by the user."
- [discussion] Source of marker files for Tabula datasets: "the marker files for the tabula datasets were created based on the 20 most sensitive markers from the PanglaoDB database45"
- [discussion] Circularity caveat: "There is also circularity in the way some of our cell type markers are defined, especially for the cell line data, as these are derived from a differential expression between the cell lines"
