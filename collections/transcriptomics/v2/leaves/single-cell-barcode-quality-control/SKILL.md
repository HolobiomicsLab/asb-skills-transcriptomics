---
name: single-cell-barcode-quality-control
description: Use when starting from a raw, unfiltered droplet-based scRNA-seq count
  matrix (mtx or h5, e.g. neurons_900_raw.h5) and you need to define the P/U barcode
  sets that EmptyNN requires before training its classifier.
when_to_use_negative:
- You need ambient RNA-corrected gene expression profiles — EmptyNN does not estimate
  corrected expression; use SoupX downstream instead
- You need to remove low-quality cells with a high fraction of mitochondrial reads
  — these may pass the initial filtering and must be removed in downstream analysis
- The data are not from a droplet-based scRNA-seq experiment with cell-free droplets
  in the raw matrix
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: EmptyNN
  role: Provides the emptynn() function that consumes the count matrix with threshold
    T and performs the P/U partitioning as the first stage of PU-learning cell calling
  repo: https://github.com/lkmklsmn/empty_nn
- name: Matrix
  role: R package used to compute per-barcode total UMI counts (column sums) on the
    sparse transposed count matrix
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.patter.2021.100311
    title: 'EmptyNN: A neural network based on positive and unlabeled learning to
      remove cell-free droplets and recover lost cells in scRNA-seq data'
  source_content_hash: sha256:3a7cbd1300d76544a096fb651780c860446fff2166a4860493aaeaffea7b7898
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
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# single-cell-barcode-quality-control

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Partition droplet-based scRNA-seq barcodes into a positive set P of bona fide cell-free droplets (total UMI counts below threshold T, default 100) and an unlabeled set U, after excluding barcodes with total counts of 10 or fewer. This positive/unlabeled labeling is the prerequisite input for EmptyNN's PU-learning classifier.

## When to use

Use when starting from a raw, unfiltered droplet-based scRNA-seq count matrix (mtx or h5, e.g. neurons_900_raw.h5) and you need to define the P/U barcode sets that EmptyNN requires before training its classifier.

## When NOT to use

- You need ambient RNA-corrected gene expression profiles — EmptyNN does not estimate corrected expression; use SoupX downstream instead
- You need to remove low-quality cells with a high fraction of mitochondrial reads — these may pass the initial filtering and must be removed in downstream analysis
- The data are not from a droplet-based scRNA-seq experiment with cell-free droplets in the raw matrix

## Inputs

- Raw unfiltered count matrix in mtx or h5 format (e.g. neurons_900_raw.h5)
- User-specified total UMI count threshold T (default: 100)

## Outputs

- Barcode-to-set assignment file pu_barcodes.csv (each retained barcode labeled P or U)
- Retained barcode set with total counts > 10, partitioned at total UMI = T

## How to apply

Load the raw unfiltered count matrix with Read10X_h5 (or mtx) and transpose it so rows are barcodes and columns are genes, computing per-barcode total UMI counts via column sums with the Matrix package. First exclude barcodes with total counts of 10 or fewer — only barcodes with total counts greater than 10 are included in the analysis. Assign retained barcodes with total UMI counts below the user-specified threshold T (default 100) to set P, the bona fide empty droplets; the remaining barcodes form set U, which may contain either cell-containing or cell-free droplets. The rationale is that the distribution of total counts across sorted barcodes separates a low-count region (ambient-RNA-only droplets) from the gray region of higher counts, so P serves as reliable negative labels while U is deliberately left unlabeled. Hyperparameter tuning over T (50–400) showed robustness, but T can be adjusted manually when genuine low-count cells exist. Write the barcode identifiers with their P or U assignment (pu_barcodes.csv) for use by the downstream emptynn() classifier step.

## Related tools

- **EmptyNN** (Provides the emptynn() function that consumes the count matrix with threshold T and performs the P/U partitioning as the first stage of PU-learning cell calling) — https://github.com/lkmklsmn/empty_nn
- **Matrix** (R package used to compute per-barcode total UMI counts (column sums) on the sparse transposed count matrix)

## Examples

```
counts <- t(Read10X_h5('./data/neurons_900_raw.h5')); nn.res <- emptynn(counts, threshold = 100, k = 10, iteration = 10, verbose = TRUE)
```

## Evaluation signals

- Every retained barcode has total UMI counts greater than 10
- Each retained barcode is assigned to exactly one of set P or set U (no overlaps, no orphans)
- The P/U boundary falls exactly at total UMI count = T (default 100)
- Set P consists only of barcodes in the low-count region of the barcode rank plot, matching the blue region of the total-count distribution

## Limitations

- The key assumption that low total count barcodes are cell-free may not hold when genuine cell-containing droplets with very low total counts exist; the user must manually adjust the T threshold to mitigate potential bias
- Parameter selection, such as the number of cross-validation folds and the T threshold, must be specified manually — no automated hyperparameter selection is implemented
- Retained cell-containing droplets may have a high fraction of mitochondrial reads, requiring separate quality filtering in downstream analysis

## Evidence
