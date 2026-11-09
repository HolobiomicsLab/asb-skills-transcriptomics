---
name: per-label-error-and-f1-evaluation
description: Use when comparing two or more marker selection methods on labeled scRNA-seq
  data and you need per-label classification error and f1 metrics aggregated across
  all cell labels (e.g., 40 synthetic labels) rather than pooled accuracy.
when_to_use_negative:
- When the analysis goal is marker ranking or panel design rather than measuring discrimination
  quality of already-selected marker sets — this skill assumes marker sets are the
  input and evaluation is the output
- When there is only one method and no baseline to compare against; the per-label
  table is designed for head-to-head comparison (scGeneFit vs one-vs-all)
- When the target task involves continuous trajectories or spatial placement of unlabeled
  cells rather than categorical cell label discrimination
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scGeneFit
  role: Produces the label-aware joint marker set whose per-label classification error
    and f1 are evaluated against the one-vs-all baseline
  repo: github.com/solevillar/scGeneFit
- name: COMET
  role: 'Baseline one-vs-all method context: COMET exhaustively selects sets of k
    markers and ranks them in one-vs-all tasks, the per-label discrimination paradigm
    against which joint selection is compared'
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-21453-4
    title: Optimal marker gene selection for cell type discrimination in single cell
      analyses
  source_content_hash: sha256:e5f6e6d86be72ae8a1fff035ce1d0049c9cff654627ace9085518ad6d9f684ec
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - COMET
  - scGeneFit
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# per-label-error-and-f1-evaluation

## Summary

A per-label evaluation protocol for marker gene selection in single-cell analyses: train a classifier on each candidate marker set and record classification error and f1 score for every cell label, enabling head-to-head comparison of label-aware methods (scGeneFit) against one-vs-all baselines.

## When to use

Use when comparing two or more marker selection methods on labeled scRNA-seq data and you need per-label classification error and f1 metrics aggregated across all cell labels (e.g., 40 synthetic labels) rather than pooled accuracy.

## When NOT to use

- When the analysis goal is marker ranking or panel design rather than measuring discrimination quality of already-selected marker sets — this skill assumes marker sets are the input and evaluation is the output
- When there is only one method and no baseline to compare against; the per-label table is designed for head-to-head comparison (scGeneFit vs one-vs-all)
- When the target task involves continuous trajectories or spatial placement of unlabeled cells rather than categorical cell label discrimination

## Inputs

- Gene expression matrix from scRNA-seq data (e.g., 1000 cells x 10,000 genes from the synthetic 40-label multimodal dataset packaged in the scGeneFit repository)
- Vector of cell labels per cell (e.g., 40 provided synthetic cell labels, or a hierarchical partition of cell type labels)
- Selected marker gene sets, one per method being compared (scGeneFit and one-vs-all)

## Outputs

- Per-label classification error and f1 score table for each method, saved as marker_selection_comparison.csv with one row per cell label
- Aggregate comparison verdict: whether the label-aware method achieves lower error or higher f1 than one-vs-all on the majority of labels

## How to apply

After obtaining marker sets from each method (e.g., scGeneFit's label-aware joint selection versus a one-vs-all baseline that scores each gene separately for each label against the rest), train and evaluate a classifier restricted to the selected markers and record per-label classification error and f1 score for every label. Aggregate the per-label error and f1 values for each method into a single CSV table with one row per label. The evaluation is justified by scGeneFit's design: it jointly recovers an optimal gene set that allows robust partitioning of labels, whereas traditional approaches test the discriminatory value of each gene separately, which can yield redundant markers. Judge success by verifying that the label-aware method achieves lower error (or higher f1) than the one-vs-all baseline on the majority of labels — as demonstrated on the 40-class synthetic multimodal simulation where the improvement persisted as the number of markers increased.

## Related tools

- **scGeneFit** (Produces the label-aware joint marker set whose per-label classification error and f1 are evaluated against the one-vs-all baseline) — github.com/solevillar/scGeneFit
- **COMET** (Baseline one-vs-all method context: COMET exhaustively selects sets of k markers and ranks them in one-vs-all tasks, the per-label discrimination paradigm against which joint selection is compared)

## Evaluation signals

- The output CSV (marker_selection_comparison.csv) contains exactly one row per cell label (e.g., 40 rows for the 40-class synthetic dataset) with columns for per-label error and f1 for each method
- The label-aware method achieves lower error (or higher f1) than one-vs-all on the majority of labels — check per-label sign of the difference
- The scGeneFit improvement over one-vs-all persists (or grows) as the number of selected markers increases, consistent with the reported scaling behavior on the 15-group/40-class simulation
- Classifier accuracy improves monotonically (or near-monotonically) with marker count for both methods, confirming the marker sets were correctly applied

## Limitations

- Both scGeneFit and one-vs-all methods assume cell type classes are linearly separable, which is reasonable in high dimensions but may not hold in lower dimensions
- No nonlinear dimension reduction is implemented; the authors plan to relax the linear separability assumption with more general nonlinear methods
- Marker selection supports only categorical labeling — it does not place unlabeled cells on trajectories or in spatial assays (a manifold constraint is envisioned future work)

## Evidence

- [abstract] scGeneFit selects gene markers that jointly optimize cell label recovery using label-aware compressive classification methods.: "Given single cell RNA-seq data and a set of cellular labels to discriminate, scGeneFit selects gene markers that jointly optimize cell label recovery using label-aware compressive"
- [intro] Joint versus per-gene selection rationale: "Where traditional approaches test the discriminatory value of each gene separately, our approach jointly recovers the optimal set of genes of a given size that allows robust partitioning of the given"
- [discussion] Linear separability assumption: "A key underlying assumption common to both scGeneFit and existing one-vs-all methods is that cell type classes are linearly separable, which is a reasonable assumption in high dimensions, but may not"
