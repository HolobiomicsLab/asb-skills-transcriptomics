---
name: synthetic-single-cell-label-classification
description: Use when a synthetic or real scRNA-seq dataset has many (tens of) cell
  labels and you must decide whether joint, label-aware marker selection outperforms
  per-label one-vs-all gene scoring for label classification.
when_to_use_negative:
- The goal is marker selection for continuous trajectories or spatial placement rather
  than categorical cell labels — the article limits scGeneFit to categorical labeling
  and defers manifold constraints to future work
- Cell-type classes are expected to be nonlinearly separable in low dimensions — both
  scGeneFit and one-vs-all assume linear separability, reasonable in high dimensions
  but may fail otherwise
- There is only one or a handful of labels with no label hierarchy — the joint, hierarchy-aware
  advantage over one-vs-all is unlikely to manifest
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_0621
- http://edamontology.org/topic_3308
tools:
- name: scGeneFit
  role: Performs the label-aware joint marker selection being benchmarked, using the
    40 cell labels as the partition to discriminate
  repo: github.com/solevillar/scGeneFit
- name: COMET
  role: 'Represents the one-vs-all style ranking baseline: exhaustively selecting
    sets of k markers ranked in a one-vs-all task, against which scGeneFit is compared'
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-021-21453-4
    title: Optimal marker gene selection for cell type discrimination in single cell
      analyses
  source_content_hash: sha256:7a5b36326f8c026ae813026da714514bce8f1cc754c133f393b4a2d0ebea096a
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - COMET
  - scGeneFit
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_0621
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# synthetic-single-cell-label-classification

## Summary

Benchmarks label-aware joint marker selection (scGeneFit) against one-vs-all marker scoring on a synthetic multimodal single-cell gene expression simulation (15 functional groups, 40 classes, 1000 cells, 10,000 genes) to test whether joint selection improves cell-label classification and how it scales with marker count.

## When to use

Use when a synthetic or real scRNA-seq dataset has many (tens of) cell labels and you must decide whether joint, label-aware marker selection outperforms per-label one-vs-all gene scoring for label classification.

## When NOT to use

- The goal is marker selection for continuous trajectories or spatial placement rather than categorical cell labels — the article limits scGeneFit to categorical labeling and defers manifold constraints to future work
- Cell-type classes are expected to be nonlinearly separable in low dimensions — both scGeneFit and one-vs-all assume linear separability, reasonable in high dimensions but may fail otherwise
- There is only one or a handful of labels with no label hierarchy — the joint, hierarchy-aware advantage over one-vs-all is unlikely to manifest

## Inputs

- Synthetic multimodal gene expression matrix (1000 cells x 10,000 genes) from the scGeneFit repository data files
- Cell labels partitioned into 15 functional groups and 40 classes
- Target marker set sizes (sweep over number of markers)
- Optional generative model choice: multivariate normal or generalized multivariate normal with Poisson link

## Outputs

- marker_selection_comparison.csv with one row per label and per-label classification error and f1 for scGeneFit and one-vs-all
- Selected marker sets per method and per marker count
- Aggregate accuracy-vs-number-of-markers curves showing the scGeneFit-over-one-vs-all improvement

## How to apply

Load the synthetic 40-label multimodal expression matrix and cell labels from the data files packaged in the scGeneFit repository (github.com/solevillar/scGeneFit). Run scGeneFit's label-aware joint marker selection on the expression matrix using the 40 provided cell labels as the partition to discriminate, then run the one-vs-all baseline that scores each gene separately for each label against all remaining labels. Sweep the target number of markers so accuracy-vs-marker-count scaling can be assessed. For each method, train and evaluate a classifier on the selected markers and record per-label classification error and f1 across all 40 labels. Aggregate per-label error and f1 into a single CSV table with one row per label and verify scGeneFit achieves lower error (or higher f1) than one-vs-all on the majority of the 40 labels. Repeat under both a multivariate normal model and a generalized multivariate normal model with a Poisson link to confirm robustness of the conclusion.

## Related tools

- **scGeneFit** (Performs the label-aware joint marker selection being benchmarked, using the 40 cell labels as the partition to discriminate) — github.com/solevillar/scGeneFit
- **COMET** (Represents the one-vs-all style ranking baseline: exhaustively selecting sets of k markers ranked in a one-vs-all task, against which scGeneFit is compared)

## Examples

```
from scGeneFit import get_markers; markers = get_markers(X, labels, num_markers=20)
```

## Evaluation signals

- marker_selection_comparison.csv has exactly one row per synthetic label (40 rows) with both error and f1 recorded for scGeneFit and one-vs-all
- scGeneFit achieves lower error (or higher f1) than one-vs-all on the majority of the 40 synthetic labels
- The classification-accuracy gap between scGeneFit and one-vs-all grows or persists as the number of selected markers increases
- The same qualitative conclusion reproduces under both the multivariate normal and the Poisson-link generalized multivariate normal generative models
- Per-label error/f1 values fall within plausible classification ranges (error in [0,1], f1 in [0,1]) with no missing labels

## Limitations

- Both scGeneFit and one-vs-all methods assume cell type classes are linearly separable, which is reasonable in high dimensions but may not hold in lower dimensions
- No nonlinear dimension reduction methods are implemented; the authors plan future work to relax the linear separability assumption
- Marker selection is limited to categorical labeling; placing unlabeled cells on trajectories or in spatial assays (manifold constraint) is envisioned future work
- The result is demonstrated on synthetic simulations (1000 cells, 10,000 genes, 40 classes); transfer to other datasets must be re-verified

## Evidence

- [abstract] Joint selection jointly optimizes label recovery: "Given single cell RNA-seq data and a set of cellular labels to discriminate, scGeneFit selects gene markers that jointly optimize cell label recovery"
- [intro] Joint vs per-gene baseline rationale: "Where traditional approaches test the discriminatory value of each gene separately, our approach jointly recovers the optimal set of genes of a given size"
- [discussion] Linear separability assumption: "A key underlying assumption common to both scGeneFit and existing one-vs-all methods is that cell type classes are linearly separable, which is a reasonable assumption in high dimensions, but may not"
- [abstract] Fewer-marker recovery of hierarchy: "the markers found by our method improves the recovery of the cell type hierarchy with fewer markers than existing methods using a computationally efficient and principled optimization"
