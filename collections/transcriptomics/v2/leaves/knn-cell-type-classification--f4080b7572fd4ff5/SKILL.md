---
name: knn-cell-type-classification
description: Use when comparing the discriminatory power of alternative marker gene
  panels (optimized, curated, or random) on labeled scRNA-seq cells, by measuring
  held-out kNN label classification error in the reduced marker space.
when_to_use_negative:
- The goal is marker selection itself rather than evaluating a fixed set of candidate
  markers — scGeneFit performs the joint optimization step upstream
- The labeling is a hierarchy or manifold/trajectory constraint rather than a flat
  categorical partition — the linear-separability assumption and categorical labeling
  do not cover those settings
- Cell-type classes are not linearly separable in the reduced marker dimensions (a
  reasonable assumption in high dimensions, but may fail in low-dimensional marker
  panels)
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: scGeneFit
  role: Produces the jointly optimized marker set whose discriminatory power is evaluated
    via kNN classification in the reduced marker space
  repo: github.com/solevillar/scGeneFit
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-021-21453-4
    title: Optimal marker gene selection for cell type discrimination in single cell
      analyses
  source_content_hash: sha256:556939f2f9ad944ebf5d220163dbecac92ea366e9d354274dc39dc40ee7c5930
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - scGeneFit
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# knn-cell-type-classification

## Summary

Trains and evaluates a k-nearest-neighbor classifier in a reduced marker-gene space (e.g., scGeneFit-selected, curated, or random panels) on single-cell RNA-seq data with flat cell-type labels, using label classification error on a held-out test split as the discriminatory-power metric, as done on the CBMC CITE-seq study.

## When to use

Use when comparing the discriminatory power of alternative marker gene panels (optimized, curated, or random) on labeled scRNA-seq cells, by measuring held-out kNN label classification error in the reduced marker space.

## When NOT to use

- The goal is marker selection itself rather than evaluating a fixed set of candidate markers — scGeneFit performs the joint optimization step upstream
- The labeling is a hierarchy or manifold/trajectory constraint rather than a flat categorical partition — the linear-separability assumption and categorical labeling do not cover those settings
- Cell-type classes are not linearly separable in the reduced marker dimensions (a reasonable assumption in high dimensions, but may fail in low-dimensional marker panels)

## Inputs

- Normalized single-cell RNA-seq expression matrix (log2(1+X) transformed), e.g., CBMC CITE-seq data from GEO accession GSE100866
- Flat cell-type label vector, one label per assayed cell
- Marker gene sets to compare: scGeneFit-selected panel, curated marker panel, and a randomly sampled panel matched in panel size

## Outputs

- Per-marker-set label classification error rates on the held-out test split (e.g., 8.05% scGeneFit, 9.90% curated, 30% random)
- CSV table with one error entry per marker set, preserving the expected error ordering

## How to apply

Normalize the single-cell expression matrix with a log2(1+X) transform, then reduce it to the candidate marker sets to compare (e.g., the jointly optimized scGeneFit panel, a curated panel, and a random panel matched in size). Split cells into a 70% training and 30% test partition, and train a k-nearest-neighbor classifier separately on each marker set's 70% training split. Evaluate each classifier on the held-out 30% and compute label classification error. Write one error value per marker set to a CSV table; the jointly optimized panel should achieve the lowest error (in the CBMC benchmark: 8.05% scGeneFit vs 9.90% curated vs 30% random). The rationale is that joint marker selection, rather than per-gene one-vs-all testing, yields a robust, less redundant set that better supports label partitioning in the reduced space.

## Related tools

- **scGeneFit** (Produces the jointly optimized marker set whose discriminatory power is evaluated via kNN classification in the reduced marker space) — github.com/solevillar/scGeneFit

## Examples

```
from sklearn.neighbors import KNeighborsClassifier; knn = KNeighborsClassifier().fit(X_train_markers, y_train); err = 1 - knn.score(X_test_markers, y_test)
```

## Evaluation signals

- The emitted CSV contains exactly one classification-error entry per marker set (scGeneFit, curated, random)
- The scGeneFit marker set yields the lowest held-out classification error, matching the reported ordering (8.05% < 9.90% < 30%)
- All three marker sets are matched in panel size and evaluated on the same 70/30 train/test partition
- Held-out test cells are never seen during kNN training for any marker set
- Random-marker error is substantially worse than the optimized and curated panels, confirming the labels are being recovered from marker expression rather than leakage

## Limitations

- A key underlying assumption is that cell type classes are linearly separable, which is reasonable in high dimensions but may fail in reduced marker spaces
- No nonlinear dimension reduction methods are implemented; relaxing the linear separability assumption is planned future work
- Marker selection and evaluation are limited to categorical labeling; no support for placing unlabeled cells on trajectories or in spatial assays

## Evidence

- [abstract] joint optimization of markers evaluated by classification: "scGeneFit selects gene markers that jointly optimize cell label recovery using label-aware compressive classification methods."
- [results] CBMC benchmark dataset: "we applied scGeneFit to a cord blood mononuclear cell (CBMC) study containing isolated cells from cord blood"
- [intro] method scope: labels for marker selection: "we developed scGeneFit, a rigorous and efficient approach for marker selection in the context of scRNA-seq data with a given hierarchical partition of labels."
- [discussion] linear separability assumption: "A key underlying assumption common to both scGeneFit and existing one-vs-all methods is that cell type classes are linearly separable"
