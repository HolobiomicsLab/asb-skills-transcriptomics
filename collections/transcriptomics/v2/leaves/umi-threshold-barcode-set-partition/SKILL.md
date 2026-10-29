---
name: umi-threshold-barcode-set-partition
description: Use when starting from a raw unfiltered count matrix (mtx or h5) and
  you need to define PU-learning training sets for distinguishing cell-free from cell-containing
  droplets by total UMI count.
when_to_use_negative:
- Input is an already-filtered count matrix — EmptyNN requires the raw unfiltered
  matrix to retain low-count barcodes for set P
- A correction for ambient RNA contamination in gene expression estimates is needed
  — EmptyNN does not estimate corrected expression profiles (use SoupX instead)
- The dataset contains genuine cell-containing droplets with very low total counts
  near the threshold — the low-count-equals-empty assumption may not hold and T must
  be manually adjusted
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: EmptyNN
  role: Implements the emptynn() function that consumes the T-threshold partition
    of P and U for PU-bagging classification
  repo: https://github.com/lkmklsmn/empty_nn
- name: Seurat
  role: Loads the raw unfiltered h5 count matrix and transposes it so rows are barcodes
    and columns are genes
- name: keras
  role: R package dependency backing the three-layer neural network trained on the
    P/U partition
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.patter.2021.100311
    title: 'EmptyNN: A neural network based on positive and unlabeled learning to
      remove cell-free droplets and recover lost cells in scRNA-seq data'
  source_content_hash: sha256:f6c61ec6d817f48f09cc602bb61b1ba87c0a5c46020ef181673711f62d3e7daf
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
  - Seurat
  - keras
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# umi-threshold-barcode-set-partition

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Partition raw droplet-based scRNA-seq barcodes into a positive set P (cell-free droplets with total UMI counts below threshold T, default 100) and an unlabeled set U (remaining barcodes) to enable positive-unlabeled learning for cell calling in EmptyNN.

## When to use

Use when starting from a raw unfiltered count matrix (mtx or h5) and you need to define PU-learning training sets for distinguishing cell-free from cell-containing droplets by total UMI count.

## When NOT to use

- Input is an already-filtered count matrix — EmptyNN requires the raw unfiltered matrix to retain low-count barcodes for set P
- A correction for ambient RNA contamination in gene expression estimates is needed — EmptyNN does not estimate corrected expression profiles (use SoupX instead)
- The dataset contains genuine cell-containing droplets with very low total counts near the threshold — the low-count-equals-empty assumption may not hold and T must be manually adjusted

## Inputs

- Raw unfiltered count matrix in mtx or h5 format (e.g., neurons_900_raw.h5)
- User-specified total-count threshold T (default: 100)
- Minimum total-count inclusion cutoff of 10 barcodes

## Outputs

- Set P: barcodes with total UMI counts below T, labeled bona fide cell-free droplets
- Set U: remaining unlabeled barcodes, candidate cell-free or cell-containing droplets
- Feature-restricted count matrix (2,000 most frequent genes among set P barcodes) used for network training

## How to apply

Load the raw unfiltered count matrix (e.g., ./data/neurons_900_raw.h5 via Seurat's Read10X_h5), transpose it so rows are barcodes and columns are genes, and keep only barcodes with total counts greater than 10. Define set P as the bona fide empty droplets with total UMI counts less than threshold T (default: 100); all remaining barcodes form the unlabeled set U, which may contain both cell-free and cell-containing droplets. Pass these sets, together with k = 10 folds and N = 10 iterations, into the emptynn() PU-bagging routine, which further restricts features to the 2,000 most frequent genes detected among set P barcodes. The key decision point is the choice of T: the default 100 works for most datasets, but if cell-containing droplets with very low total counts exist the user can adjust T to mitigate bias, since low total count is assumed to indicate cell-free droplets.

## Related tools

- **EmptyNN** (Implements the emptynn() function that consumes the T-threshold partition of P and U for PU-bagging classification) — https://github.com/lkmklsmn/empty_nn
- **Seurat** (Loads the raw unfiltered h5 count matrix and transposes it so rows are barcodes and columns are genes)
- **keras** (R package dependency backing the three-layer neural network trained on the P/U partition)

## Examples

```
library(EmptyNN); library(Seurat); counts <- t(Read10X_h5("./data/neurons_900_raw.h5")); nn.res <- emptynn(counts, threshold = 100, k = 10, iteration = 10, verbose = TRUE)
```

## Evaluation signals

- Verify only barcodes with total counts greater than 10 are included in the analysis
- Check that all barcodes in set P have total UMI counts strictly below threshold T (default 100) and that set P and set U together cover all retained barcodes
- Confirm the output includes both a boolean cell-free/cell-containing prediction vector and per-barcode probabilities for set U
- Confirm the training feature space is restricted to the 2,000 most frequent genes detected among set P barcodes
- Sanity-check the partition against the barcode rank plot total-count distribution: set P should sit in the low-count region of the sorted distribution

## Limitations

- The assumption that low total count barcodes are cell-free may not hold when genuine low-count cells exist; the user must manually adjust the T threshold to mitigate potential bias
- Parameter selection (threshold T and number of cross-validation folds k) is manual; no hyperparameter selection approach is implemented
- Retained cell-containing droplets may have a high fraction of mitochondrial reads; low-quality cells require separate removal in downstream analysis
- EmptyNN does not correct ambient RNA contamination in gene expression estimates; external tools such as SoupX are required
