---
name: marker-panel-benchmark-analysis
description: Use when single-cell gene expression data (e.g. scRNA-seq, synthetic
  multimodal simulations) comes with cell type labels and the goal is to test whether
  jointly optimized marker panels recover labels better than one-vs-all gene-by-gene
  scoring.
when_to_use_negative:
- The data lack categorical cell labels or the goal is to place unlabeled cells on
  trajectories or spatial locations — scGeneFit only supports categorical labeling.
- Cell type classes are expected to be non-linearly separable in the low-dimensional
  marker space; the method assumes linear separability, which is reasonable in high
  dimensions but may not hold otherwise.
- Input is already a reduced feature table or a fixed marker panel — the skill benchmarks
  marker selection on the full gene-by-cell expression matrix.
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scGeneFit
  role: Performs the label-aware joint marker selection being benchmarked; run on
    the expression matrix using the provided cell labels as the partition to discriminate
  repo: github.com/solevillar/scGeneFit
- name: COMET
  role: 'Reference one-vs-all selection strategy: exhaustively selects sets of k markers
    and ranks them in a one-vs-all task, used as the comparison baseline'
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-21453-4
    title: Optimal marker gene selection for cell type discrimination in single cell
      analyses
  source_content_hash: sha256:0651383203a6d62932b6d3eedd2c5debba95d835590789f3309f6cc31e87bf89
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - COMET
  - scGeneFit
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# marker-panel-benchmark-analysis

## Summary

Benchmark scGeneFit's label-aware joint marker selection against one-vs-all marker selection (the strategy used by methods such as COMET) on gene expression data with cellular labels, quantifying per-label classification error and f1 across increasing marker-set sizes.

## When to use

Use when single-cell gene expression data (e.g. scRNA-seq, synthetic multimodal simulations) comes with cell type labels and the goal is to test whether jointly optimized marker panels recover labels better than one-vs-all gene-by-gene scoring.

## When NOT to use

- The data lack categorical cell labels or the goal is to place unlabeled cells on trajectories or spatial locations — scGeneFit only supports categorical labeling.
- Cell type classes are expected to be non-linearly separable in the low-dimensional marker space; the method assumes linear separability, which is reasonable in high dimensions but may not hold otherwise.
- Input is already a reduced feature table or a fixed marker panel — the skill benchmarks marker selection on the full gene-by-cell expression matrix.

## Inputs

- Gene expression matrix (scRNA-seq or synthetic multimodal data, e.g. 1000 cells x 10,000 genes)
- Vector of cellular labels to discriminate (e.g. 40 synthetic classes over 15 functional groups)
- Target marker set size (number of markers k)

## Outputs

- Per-label classification error and f1 for scGeneFit and one-vs-all selection
- Aggregated CSV comparison table (e.g. marker_selection_comparison.csv, one row per label)
- Selected marker gene sets per method and per marker-count setting

## How to apply

Load the expression matrix and cell labels (e.g. the synthetic 40-label multimodal dataset with 15 functional groups packaged in the scGeneFit repository, or a CBMC/mouse cortex scRNA-seq dataset). Run scGeneFit's label-aware joint marker selection using the provided labels as the partition to discriminate, which jointly recovers the optimal gene set of a given size rather than testing each gene separately. In parallel run the one-vs-all baseline, which scores each gene separately for each label against all remaining labels (as COMET-style ranking does). For each method, train and evaluate a classifier on the selected markers, recording per-label classification error and f1 across all labels while scaling the target number of markers. Aggregate per-label error and f1 for both methods into a single CSV (e.g. marker_selection_comparison.csv with one row per label) and verify that scGeneFit achieves lower error (or higher f1) than one-vs-all on the majority of labels.

## Related tools

- **scGeneFit** (Performs the label-aware joint marker selection being benchmarked; run on the expression matrix using the provided cell labels as the partition to discriminate) — github.com/solevillar/scGeneFit
- **COMET** (Reference one-vs-all selection strategy: exhaustively selects sets of k markers and ranks them in a one-vs-all task, used as the comparison baseline)

## Evaluation signals

- scGeneFit achieves lower per-label classification error (or higher f1) than one-vs-all on the majority of the 40 synthetic cell labels
- The improvement persists across increasing numbers of selected markers (scaling with marker count)
- Output CSV has exactly one row per label with paired error and f1 values for both methods
- Selected marker sets are less redundant than the one-vs-all baseline's markers
- On the spectral toy model, scGeneFit recovers the informative features (e.g. the two circle-coordinate features) whenever the target marker set size is at least 2

## Limitations

- A key underlying assumption is that cell type classes are linearly separable, which is reasonable in high dimensions but may not hold in lower dimensions.
- No nonlinear dimension reduction methods are implemented; relaxing the linear separability assumption is planned future work.
- Marker selection is limited to categorical labeling; no support for placing unlabeled cells on trajectories or in spatial assays (a manifold constraint is envisioned future work).

## Evidence

- [abstract] scGeneFit selects gene markers that jointly optimize cell label recovery: "Given single cell RNA-seq data and a set of cellular labels to discriminate, scGeneFit selects gene markers that jointly optimize cell label recovery"
- [intro] One-vs-all baseline characterization: "One such method, COMET10, exhaustively select sets of k markers that it then ranks in a one-vs-all task."
- [intro] Joint vs per-gene selection rationale: "Where traditional approaches test the discriminatory value of each gene separately, our approach jointly recovers the optimal set of genes of a given size"
- [results] Spectral toy model sanity check: "scGeneFit selected as markers the two features representing the circle coordinates whenever the size of the target marker set was ≥2"
- [discussion] Linear separability assumption: "cell type classes are linearly separable, which is a reasonable assumption in high dimensions, but may not"
