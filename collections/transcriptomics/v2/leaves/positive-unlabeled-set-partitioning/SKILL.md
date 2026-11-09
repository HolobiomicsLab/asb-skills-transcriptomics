---
name: positive-unlabeled-set-partitioning
description: Use when starting from a raw unfiltered count matrix (mtx or h5) and
  you must assign each retained barcode to a positive (cell-free) or unlabeled (mixed
  cell-containing/cell-free) set before PU-learning-based cell calling.
when_to_use_negative:
- The dataset is expected to contain genuine cell-containing droplets with very low
  total counts comparable to ambient-RNA barcodes; the low-count-implies-empty assumption
  of set P may not hold and the user must manually adjust T
- The input is already a filtered/processed feature table rather than a raw unfiltered
  count matrix; partitioning assumes the full barcode space including cell-free droplets
- The goal is ambient RNA correction of expression profiles rather than barcode partitioning;
  EmptyNN does not estimate corrected profiles and tools such as SoupX are needed
edam_operation: http://edamontology.org/operation_3695
tools:
- name: EmptyNN
  role: Implements the P/U partitioning as the first stage of its PU-learning cell-calling
    workflow (emptynn() with threshold = T)
  repo: https://github.com/lkmklsmn/empty_nn
- name: Matrix
  role: R package used to compute per-barcode total UMI counts (column sums) on the
    sparse raw count matrix for P/U assignment
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.patter.2021.100311
    title: 'EmptyNN: A neural network based on positive and unlabeled learning to
      remove cell-free droplets and recover lost cells in scRNA-seq data'
  source_content_hash: sha256:bcff8bb8ad1c550e3a419b050c5cf64ce8e93d768ccd1bd87d928bb94b0e3091
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/lkmklsmn/empty_nn
  repo_url: https://github.com/lkmklsmn/empty_nn
  tools:
  - EmptyNN
  - Matrix
  edam_operation: http://edamontology.org/operation_3695
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 5
---

# positive-unlabeled-set-partitioning

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Partitions raw droplet-based scRNA-seq barcodes into a positively labeled set P of bona fide cell-free droplets (total UMI counts below threshold T, default 100) and an unlabeled set U of remaining barcodes, the prerequisite step for EmptyNN's positive-unlabeled (PU) learning classifier.

## When to use

Use when starting from a raw unfiltered count matrix (mtx or h5) and you must assign each retained barcode to a positive (cell-free) or unlabeled (mixed cell-containing/cell-free) set before PU-learning-based cell calling.

## When NOT to use

- The dataset is expected to contain genuine cell-containing droplets with very low total counts comparable to ambient-RNA barcodes; the low-count-implies-empty assumption of set P may not hold and the user must manually adjust T
- The input is already a filtered/processed feature table rather than a raw unfiltered count matrix; partitioning assumes the full barcode space including cell-free droplets
- The goal is ambient RNA correction of expression profiles rather than barcode partitioning; EmptyNN does not estimate corrected profiles and tools such as SoupX are needed

## Inputs

- Raw unfiltered count matrix in mtx or h5 format (e.g. neurons_900_raw.h5)
- User-specified threshold T (default 100)

## Outputs

- Set P: list of barcodes with total UMI counts below T (bona fide cell-free droplets)
- Set U: list of retained barcodes with total UMI counts at or above T (unlabeled, mixed cell-containing/cell-free)
- Barcode-to-set assignment table (e.g. pu_barcodes.csv)

## How to apply

Load the raw unfiltered count matrix (e.g. neurons_900_raw.h5 via Read10X_h5 or mtx format) and transpose it so rows are barcodes and columns are genes. Compute per-barcode total UMI counts (column sums, e.g. with the Matrix R package) and exclude barcodes with total counts of 10 or fewer from the analysis. Assign retained barcodes with total counts below the user-specified threshold T (default: 100) to set P, the bona fide empty droplets, and assign all remaining barcodes to set U, which may contain either cell-containing or cell-free droplets. The rationale is that very-low-count barcodes are almost certainly cell-free, providing reliable positive labels, while higher-count barcodes are ambiguous and must remain unlabeled for the PU classifier. Write the barcode-to-set assignments (P or U) to a file such as pu_barcodes.csv; the boundary between P and U must fall exactly at total UMI = T.

## Related tools

- **EmptyNN** (Implements the P/U partitioning as the first stage of its PU-learning cell-calling workflow (emptynn() with threshold = T)) — https://github.com/lkmklsmn/empty_nn
- **Matrix** (R package used to compute per-barcode total UMI counts (column sums) on the sparse raw count matrix for P/U assignment)

## Examples

```
library(EmptyNN); counts <- t(Read10X_h5("./data/neurons_900_raw.h5")); nn.res <- emptynn(counts, threshold = 100, k = 10, iteration = 10, verbose = TRUE)
```

## Evaluation signals

- Every retained barcode has total UMI counts greater than 10 (the minimum-inclusion cutoff)
- Each retained barcode is assigned to exactly one of P or U (disjoint, exhaustive partition)
- The P/U boundary falls exactly at total UMI count = T (e.g. barcodes with counts < 100 in P, counts >= 100 in U for default T = 100)
- Set P is non-empty and consists exclusively of the lowest-total-count barcodes; set U contains all higher-count barcodes
- The transposed count matrix has barcodes as rows and genes as columns before computing totals

## Limitations

- The assumption that low total count barcodes are cell-free may not hold when cell-containing droplets with very low total counts exist; in such cases the user can adjust the T threshold to mitigate potential bias
- Threshold T must be specified manually; no hyperparameter selection approach is implemented, although the authors claim the algorithm is robust to various choices of k and T
- Only barcodes with total counts greater than 10 are included in the analysis, so extremely low-count barcodes are excluded entirely

## Evidence
