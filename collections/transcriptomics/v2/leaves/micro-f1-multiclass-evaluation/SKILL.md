---
name: micro-f1-multiclass-evaluation
description: Use when a per-cell assignment (e.g., inferred spatial layer L1-L6 for
  15413 scRNA-seq cells) must be scored against ground-truth multiclass labels and
  compared to a chance/majority baseline.
when_to_use_negative:
- Input predictions have no per-cell ground-truth labels — micro F1 cannot be computed
  without paired reference labels
- The evaluation target is continuous (e.g., reconstructed spatial gene expression
  values) rather than categorical class membership
- Only 2 of 1020 STARmap genes or unaligned cell sets are available, so predicted
  and truth label vectors have different lengths/indices
edam_operation: http://edamontology.org/operation_0282
tools:
- name: scikit-learn
  role: Computes the multiclass micro F1 score of the L1-L6 spatial-origin assignment
- name: SpaOTsc
  role: Produces the per-cell layer predictions being evaluated, via the transport_plan
    mapping between 15413 scRNA-seq cells and 1549 STARmap positions
  repo: https://github.com/zcang/SpaOTsc
- name: pandas
  role: Loads the Allen Brain Atlas mouse_VISp_2018-06-14_exon-matrix.csv scRNA-seq
    matrix and writes the per-cell prediction table
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-020-15968-5
    title: Inferring spatial and signaling relationships between cells from single
      cell transcriptomic data
  source_content_hash: sha256:dba2ab8fb7806dd34aa42bae8652d0464325bb8b9cc01a1a4d696fc9379a7cc2
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/zcang/SpaOTsc
  tools:
  - Seurat
  - SpaOTsc
  - numpy
  - pandas
  - scikit-learn
  - scipy
  edam_operation: http://edamontology.org/operation_0282
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# micro-f1-multiclass-evaluation

## Summary

Evaluates per-cell label assignments (e.g., SpaOTsc's L1-L6 spatial-origin assignment of mouse visual cortex scRNA-seq cells) as a multiclass classification problem using scikit-learn's micro-averaged F1 score, benchmarked against a majority-population baseline.

## When to use

Use when a per-cell assignment (e.g., inferred spatial layer L1-L6 for 15413 scRNA-seq cells) must be scored against ground-truth multiclass labels and compared to a chance/majority baseline.

## When NOT to use

- Input predictions have no per-cell ground-truth labels — micro F1 cannot be computed without paired reference labels
- The evaluation target is continuous (e.g., reconstructed spatial gene expression values) rather than categorical class membership
- Only 2 of 1020 STARmap genes or unaligned cell sets are available, so predicted and truth label vectors have different lengths/indices

## Inputs

- Vector of per-cell predicted labels (e.g., L1-L6 layer assignments for 15413 scRNA-seq cells from spatial_origin_assignment.csv)
- Vector of ground-truth multiclass labels for the same cells (original spatial region L1-L6)
- Optional baseline prediction vector (constant majority population label)

## Outputs

- Multiclass micro F1 score (expected ≈ 0.48 for SpaOTsc vs 0.09 for the majority baseline)
- Per-cell layer prediction table (spatial_origin_assignment.csv)

## How to apply

Collect the per-cell predicted labels (here, cortical layers L1-L6 assigned to each of the 15413 Allen Brain Atlas mouse visual cortex scRNA-seq cells from the weighted SpaOTsc transport mapping onto layer-annotated STARmap positions) and the corresponding ground-truth labels. Compute the multiclass micro F1 score with scikit-learn (f1_score with average='micro'), which aggregates per-class contributions so that frequent classes dominate. Critically, compute the same metric for a baseline model that always predicts the majority population label, because class imbalance makes raw accuracy misleading. In this task the SpaOTsc assignment reached micro F1 ≈ 0.48 versus 0.09 for the majority baseline, the value reported for this classification (Fig 2e,f). Report both the score and the per-cell predictions so the result is reproducible.

## Related tools

- **scikit-learn** (Computes the multiclass micro F1 score of the L1-L6 spatial-origin assignment)
- **SpaOTsc** (Produces the per-cell layer predictions being evaluated, via the transport_plan mapping between 15413 scRNA-seq cells and 1549 STARmap positions) — https://github.com/zcang/SpaOTsc
- **pandas** (Loads the Allen Brain Atlas mouse_VISp_2018-06-14_exon-matrix.csv scRNA-seq matrix and writes the per-cell prediction table)

## Examples

```
from sklearn.metrics import f1_score; micro_f1 = f1_score(y_true_layers, y_pred_layers, average='micro')
```

## Evaluation signals

- Micro F1 of the SpaOTsc L1-L6 assignment is approximately 0.48, matching the value reported for this classification (Fig 2e,f)
- Majority-population baseline yields micro F1 ≈ 0.09, confirming the model score substantially exceeds class-imbalance level
- Predicted and truth label vectors have equal length (15413 cells) and aligned cell indices
- All predicted labels fall within the L1-L6 class set of the ground-truth annotations
- Per-cell predictions and the score are persisted to spatial_origin_assignment.csv for reproducibility

## Limitations

- Micro F1 is dominated by frequent classes, so a majority-label baseline (0.09 here) must be reported alongside it
- Assignment quality depends on the upstream SpaOTsc transport mapping and the low-dimensional common space built with Seurat 'cca', since direct correlation between 1020-gene STARmap data and scRNA-seq is noisy

## Evidence

- [other] micro F1 finding: [evidence span withheld — anchor status: not_found]
- [methods] mapping construction: "SpaOTsc constructs a mapping between the n cells in scRNA-seq data and the m positions in spatial data by solving an optimal transport problem"
- [discussion] spatial origin assignment motivation: "along with spatial origin assignments to the scRNA-seq data of mouse visual cortex"
- [readme] initialization API: "spsc = SpaOTsc.spatial_sc(sc_data=df_sc, is_dmat=is_dmat, sc_dmat=sc_dmat)"
