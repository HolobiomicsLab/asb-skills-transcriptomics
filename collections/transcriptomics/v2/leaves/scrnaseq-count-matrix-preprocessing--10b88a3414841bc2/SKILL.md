---
name: scrnaseq-count-matrix-preprocessing
description: Use when a raw unfiltered droplet-based scRNA-seq count matrix (.h5 or
  .mtx) must be shaped for EmptyNN cell-calling or PU-learning based filtering of
  cell-free droplets.
when_to_use_negative:
- The matrix is already filtered by Cell Ranger v.3+ / EmptyDrops — cell calling has
  effectively been done and EmptyNN's raw-input requirement is violated
- The goal is to correct ambient RNA contamination in retained cells' expression estimates
  — EmptyNN does not estimate corrected profiles; use SoupX instead
- The dataset contains many genuine cell-containing droplets with very low total counts
  near threshold T — the low-count-equals-empty assumption may not hold and manual
  T adjustment or a different approach is needed
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: EmptyNN
  role: Downstream cell-calling algorithm that consumes the preprocessed matrix; its
    set P / set U definition drives the threshold T partitioning
  repo: https://github.com/lkmklsmn/empty_nn
- name: Seurat
  role: Used to load and transpose the raw h5 count matrix via Read10X_h5
- name: keras
  role: Backend dependency for the neural network trained on the preprocessed sets
    P and U
provenance:
  source_task_ids:
  - task_001
  - task_005
  source_papers:
  - doi: 10.1016/j.patter.2021.100311
    title: 'EmptyNN: A neural network based on positive and unlabeled learning to
      remove cell-free droplets and recover lost cells in scRNA-seq data'
  source_content_hash: sha256:83d89869b95d331da78ca3344ed5bca3c1dd9faf1ad6f49a0accf810a5cfe5c6
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
  - Seurat
  - keras
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# scrnaseq-count-matrix-preprocessing

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Preprocess raw unfiltered droplet scRNA-seq count matrices (mtx or h5) so they are suitable for EmptyNN cell-calling: transpose barcodes to rows, restrict to barcodes with total counts > 10, and partition barcodes into set P (total UMI counts below threshold T, default 100) and unlabeled set U.

## When to use

Use when a raw unfiltered droplet-based scRNA-seq count matrix (.h5 or .mtx) must be shaped for EmptyNN cell-calling or PU-learning based filtering of cell-free droplets.

## When NOT to use

- The matrix is already filtered by Cell Ranger v.3+ / EmptyDrops — cell calling has effectively been done and EmptyNN's raw-input requirement is violated
- The goal is to correct ambient RNA contamination in retained cells' expression estimates — EmptyNN does not estimate corrected profiles; use SoupX instead
- The dataset contains many genuine cell-containing droplets with very low total counts near threshold T — the low-count-equals-empty assumption may not hold and manual T adjustment or a different approach is needed

## Inputs

- Raw unfiltered count matrix in mtx or h5 format (e.g., neurons_900_raw.h5 from download_data.sh)

## Outputs

- Transposed barcode-by-gene count matrix restricted to barcodes with total counts > 10
- Set P of barcodes with total UMI counts below threshold T (default 100) and unlabeled set U of remaining barcodes
- Feature-restricted matrix limited to the 2,000 most frequent genes among set P barcodes (optional input to emptynn())

## How to apply

Load the raw unfiltered count matrix (e.g., ./data/neurons_900_raw.h5) with Seurat's Read10X_h5 and transpose it so rows are barcodes and columns are genes, keeping only barcodes with total counts greater than 10. Partition the retained barcodes into set P — bona fide empty droplets with total UMI counts below the user-specified threshold T (default: 100) — and set U, the remaining unlabeled barcodes that may contain both cell-free and cell-containing droplets. Optionally restrict the feature space to the 2,000 most frequent genes detected among set P barcodes before handing the matrix to emptynn(). The rationale is that very low-count barcodes are reliable negatives (cell-free) for PU learning, while the ambient RNA in cell-free droplets produces UMI counts that make higher-count empty droplets hard to distinguish without a classifier.

## Related tools

- **EmptyNN** (Downstream cell-calling algorithm that consumes the preprocessed matrix; its set P / set U definition drives the threshold T partitioning) — https://github.com/lkmklsmn/empty_nn
- **Seurat** (Used to load and transpose the raw h5 count matrix via Read10X_h5)
- **keras** (Backend dependency for the neural network trained on the preprocessed sets P and U)

## Examples

```
library(Seurat); counts <- Read10X_h5("./data/neurons_900_raw.h5", use.names=TRUE, unique.features=TRUE); counts <- t(counts); nn.res <- emptynn(counts, threshold = 100, k = 10, iteration = 10, verbose = TRUE)
```

## Evaluation signals

- Every retained barcode row has total UMI counts strictly greater than 10; barcodes with total counts <= 10 are excluded
- The count matrix is oriented rows = barcodes/cells, columns = genes after transposition
- The size of set P matches the count of barcodes with total counts below T = 100 (or the chosen T) in the total-count distribution
- emptynn() returns a boolean cell-free/cell-containing prediction vector and per-barcode probabilities for set U without errors, confirming the matrix shape is compatible

## Limitations

- The assumption that low total count barcodes are cell-free may not hold when cell-containing droplets with very low total counts exist; the user can adjust the T threshold to mitigate potential bias
- Only barcodes with total counts greater than 10 are included in the analysis, so extremely sparse barcodes are discarded before cell calling
- Parameter selection such as the T threshold is manual; no hyperparameter selection approach is implemented

## Evidence
