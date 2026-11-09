---
name: cell-type-label-hierarchy-classification
description: Use when benchmarking marker gene panels against a two-layer cell-type
  label hierarchy on scRNA-seq data, or when testing whether fewer hierarchy-aware
  markers match a one-vs-all baseline panel.
when_to_use_negative:
- Labels are not organized as a hierarchical partition of cell types, and only flat
  one-vs-all discrimination is required
- The task is continuous labeling (trajectories or spatial placement) rather than
  categorical cell-type labels — the article explicitly defers manifold-constrained
  marker selection to future work
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: scGeneFit
  role: Performs hierarchical marker gene selection that jointly optimizes recovery
    of the two-layer cell-type label hierarchy on the Zeisel scRNA-seq data
  repo: github.com/solevillar/scGeneFit
- name: COMET
  role: Referenced one-vs-all style baseline framework that exhaustively selects and
    ranks sets of k markers in a one-vs-all task
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-21453-4
    title: Optimal marker gene selection for cell type discrimination in single cell
      analyses
  source_content_hash: sha256:85adee0295ae66920dabdd7245968dfb1d6a59d1417d6565261ee907ed622ef7
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - COMET
  - scGeneFit
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# cell-type-label-hierarchy-classification

## Summary

Reproduce scGeneFit's hierarchical marker-selection benchmark on the Zeisel mouse cortex/hippocampus scRNA-seq dataset (GSE60361), comparing hierarchical scGeneFit panels of 30 and 40 markers against a 48-marker one-vs-all baseline on first-layer cell-label classification error.

## When to use

Use when benchmarking marker gene panels against a two-layer cell-type label hierarchy on scRNA-seq data, or when testing whether fewer hierarchy-aware markers match a one-vs-all baseline panel.

## When NOT to use

- Labels are not organized as a hierarchical partition of cell types, and only flat one-vs-all discrimination is required
- The task is continuous labeling (trajectories or spatial placement) rather than categorical cell-type labels — the article explicitly defers manifold-constrained marker selection to future work

## Inputs

- Zeisel mouse cortex/hippocampus scRNA-seq dataset (GEO accession GSE60361)
- Two-layer cell-type label hierarchy annotation for the Zeisel cells
- Target marker panel sizes (30, 40, and 48 genes)

## Outputs

- First-layer classification error table for the three marker panels (CSV)
- Three selected marker gene lists (scGeneFit-30, scGeneFit-40, one-vs-all-48) (CSV)

## How to apply

Download the Zeisel mouse cortical scRNA-seq data (GEO accession GSE60361) together with its two-layer cell-type label hierarchy annotation. Run the hierarchical variant of scGeneFit — which jointly optimizes cell label recovery using label-aware compressive classification rather than testing genes separately — with the two-layer labels to select target panels of 30 and 40 marker genes. Run the one-vs-all baseline by ranking genes on one-vs-all label discrimination and retaining the top 48 markers. Classify held-out cells at the first layer of the hierarchy for each of the three panels and record classification error; the expected reference values are 8.75% (scGeneFit-30), 6.44% (scGeneFit-40), and 9.74% (one-vs-all-48). Write the error table and the three marker gene lists to CSV files.

## Related tools

- **scGeneFit** (Performs hierarchical marker gene selection that jointly optimizes recovery of the two-layer cell-type label hierarchy on the Zeisel scRNA-seq data) — github.com/solevillar/scGeneFit
- **COMET** (Referenced one-vs-all style baseline framework that exhaustively selects and ranks sets of k markers in a one-vs-all task)

## Examples

```
from scGeneFit.functions import get_markers; markers_30 = get_markers(data, labels_hierarchy, num_markers=30)
```

## Evaluation signals

- First-layer classification error on held-out cells matches 8.75% for scGeneFit-30, 6.44% for scGeneFit-40, and 9.74% for the one-vs-all-48 baseline
- scGeneFit panels with 30 or 40 markers perform on par with (or better than) the 48-marker one-vs-all panel, demonstrating hierarchy-aware marker efficiency
- Three marker gene lists are exported with exactly 30, 40, and 48 unique genes respectively
- Selected markers form a robust, less redundant set than one-vs-all markers, consistent with the paper's claim of reduced redundancy
- Error table and marker lists are written as well-formed CSV files with matching row counts

## Limitations

- Both scGeneFit and one-vs-all methods assume cell type classes are linearly separable, which is reasonable in high dimensions but may not hold in lower dimensions
- No nonlinear dimension reduction methods are implemented; relaxing the linear separability assumption is planned future work
- Marker selection is limited to categorical labeling; no support for placing unlabeled cells on trajectories or in spatial assays

## Evidence

- [results] scGeneFit with 30 or 40 markers performed on par with the one-vs-all method using 48 markers, with first-layer hierarchy classification errors on held-out cells of 8.75% (30 markers), 6.44% (40 markers), and 9.74% (one-vs-all).: [evidence span withheld — anchor status: not_found]
- [intro] we developed scGeneFit, a rigorous and efficient approach for marker selection in the context of scRNA-seq data with a given hierarchical partition of labels.: "we developed scGeneFit, a rigorous and efficient approach for marker selection in the context of scRNA-seq data with a given hierarchical partition of labels."
- [intro] our approach jointly recovers the optimal set of genes of a given size that allows robust partitioning of the given labels: "our approach jointly recovers the optimal set of genes of a given size that allows robust partitioning of the given"
- [discussion] A key underlying assumption common to both scGeneFit and existing one-vs-all methods is that cell type classes are linearly separable, which is a reasonable assumption in high dimensions, but may not: "A key underlying assumption common to both scGeneFit and existing one-vs-all methods is that cell type classes are linearly separable, which is a reasonable assumption in high dimensions, but may not"
- [discussion] we envision relaxing the categorical labeling to a manifold constraint that will allow selection of markers to place unlabeled cells at, for example, time points along cell trajectories or locations: "we envision relaxing the categorical labeling to a manifold constraint that will allow selection of markers to place unlabeled cells at, for example, time points along cell trajectories or locations"
