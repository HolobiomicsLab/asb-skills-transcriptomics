---
name: barcode-total-umi-count-filtering
description: Use when a raw unfiltered droplet-based scRNA-seq count matrix (mtx or
  h5) must be split into known-empty barcodes and unlabeled barcodes prior to PU-learning-based
  cell calling, or when a simple total-UMI cutoff defines empty droplets.
when_to_use_negative:
- When cell-containing droplets with very low total counts are expected — the assumption
  that low-count barcodes are cell-free may not hold, and the T threshold must be
  manually adjusted to mitigate bias
- When an alternative cell-calling model is appropriate, e.g., EmptyDrops' Dirichlet-multinomial
  profile-deviation test or Cell Ranger v.2's total-count-distribution cutoff (m/10
  rule), rather than a fixed total-UMI threshold
- When the input is already a filtered cell-by-gene matrix rather than the raw unfiltered
  droplet matrix — the partition requires all barcodes including empty ones
edam_operation: http://edamontology.org/operation_3218
edam_topics:
- http://edamontology.org/topic_3518
tools:
- name: EmptyNN
  role: Cell-calling algorithm whose first stage defines set P (bona fide empty droplets)
    and set U by total UMI count threshold T before PU-learning classifier training;
    its emptynn() function accepts the threshold = 100 parameter
  repo: https://github.com/lkmklsmn/empty_nn
- name: Matrix
  role: R package used to load the sparse count matrix and compute per-barcode column
    sums for the total UMI counts that define the P/U boundary
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.patter.2021.100311
    title: 'EmptyNN: A neural network based on positive and unlabeled learning to
      remove cell-free droplets and recover lost cells in scRNA-seq data'
  source_content_hash: sha256:8ad055bec72203df1c012187227a9a084ef631ce1e20eadd488e74890a78f698
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
  edam_operation: http://edamontology.org/operation_3218
  edam_topics:
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# barcode-total-umi-count-filtering

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Partitions barcodes from a raw unfiltered scRNA-seq count matrix into a positive set P (bona fide cell-free droplets, total UMI counts below a user-specified threshold T, default 100) and an unlabeled set U (possibly cell-containing or cell-free), providing the PU-learning partition EmptyNN requires before training its classifier.

## When to use

Use when a raw unfiltered droplet-based scRNA-seq count matrix (mtx or h5) must be split into known-empty barcodes and unlabeled barcodes prior to PU-learning-based cell calling, or when a simple total-UMI cutoff defines empty droplets.

## When NOT to use

- When cell-containing droplets with very low total counts are expected — the assumption that low-count barcodes are cell-free may not hold, and the T threshold must be manually adjusted to mitigate bias
- When an alternative cell-calling model is appropriate, e.g., EmptyDrops' Dirichlet-multinomial profile-deviation test or Cell Ranger v.2's total-count-distribution cutoff (m/10 rule), rather than a fixed total-UMI threshold
- When the input is already a filtered cell-by-gene matrix rather than the raw unfiltered droplet matrix — the partition requires all barcodes including empty ones

## Inputs

- Raw unfiltered count matrix in mtx or h5 format (e.g., neurons_900_raw.h5), transposed so rows are barcodes and columns are genes
- User-specified threshold T for total UMI counts (default: 100)
- Per-barcode total UMI counts computed via column sums (Matrix R package)

## Outputs

- Barcode identifiers with P/U set assignment (e.g., pu_barcodes.csv)
- Set P: list of barcodes with total UMI counts below T (bona fide cell-free droplets)
- Set U: list of retained barcodes with total UMI counts at or above T (unlabeled: cell-containing or cell-free)

## How to apply

Load the raw unfiltered count matrix (e.g., neurons_900_raw.h5 via Read10X_h5) and transpose it so rows are barcodes and columns are genes. Compute per-barcode total UMI counts with column sums using the Matrix R package. First exclude barcodes with total counts of 10 or fewer from the analysis entirely. Then assign every retained barcode with total UMI counts below the user-specified threshold T (default 100) to set P — these low-count barcodes are treated as bona fide cell-free droplets — and assign all remaining retained barcodes to set U, which may contain either cell-containing or cell-free droplets. The P/U boundary must fall exactly at total UMI = T, and each retained barcode belongs to exactly one set; the labels are then written out (e.g., pu_barcodes.csv) as input for EmptyNN's positive-unlabeled classifier training.

## Related tools

- **EmptyNN** (Cell-calling algorithm whose first stage defines set P (bona fide empty droplets) and set U by total UMI count threshold T before PU-learning classifier training; its emptynn() function accepts the threshold = 100 parameter) — https://github.com/lkmklsmn/empty_nn
- **Matrix** (R package used to load the sparse count matrix and compute per-barcode column sums for the total UMI counts that define the P/U boundary)

## Examples

```
counts <- t(Read10X_h5("./data/neurons_900_raw.h5")); totals <- colSums(counts); p <- colnames(counts)[totals < 100 & totals > 10]; u <- colnames(counts)[totals >= 100]
```

## Evaluation signals

- Every retained barcode has total counts strictly greater than 10 (barcodes with total counts <= 10 are excluded before partitioning)
- Each retained barcode is assigned to exactly one of P or U — the sets are disjoint and jointly exhaustive over retained barcodes
- The P/U boundary falls exactly at total UMI = T (default 100): max total count in P < T and min total count in U >= T
- Set P size is consistent with the low-count tail of the barcode rank plot (the blue region of the total-count distribution)
- Output file (e.g., pu_barcodes.csv) contains one row per retained barcode with a valid P or U label

## Limitations

- The key assumption — that low total count barcodes are cell-free — may not hold when genuine cell-containing droplets with very low total counts exist; the user must manually adjust the T threshold to mitigate potential bias
- Threshold T selection is manual with no hyperparameter selection approach implemented (although the algorithm is claimed robust to various choices of k and T)
- Retained cell-containing droplets may have a high fraction of mitochondrial reads; low-quality cells passing this filter require separate removal in downstream analysis
- This filter does not correct ambient RNA contamination in gene expression estimates of retained droplets; tools such as SoupX must be applied separately

## Evidence
