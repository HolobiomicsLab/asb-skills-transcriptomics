---
name: positive-unlabeled-learning-classification
description: Use when you have a raw unfiltered count matrix (mtx or h5) from droplet-based
  scRNA-seq/snRNA-seq and need to distinguish cell-free from cell-containing droplets,
  including recovering genuine cells with total counts below a fixed cutoff.
when_to_use_negative:
- When cell-containing droplets with very low total counts may be abundant and the
  assumption that low-count barcodes are cell-free does not hold without manually
  adjusting the T threshold
- When the goal is to correct ambient RNA contamination in gene expression estimates
  — EmptyNN does not estimate corrected expression profiles; use a tool such as SoupX
  instead
- When the input is already a filtered cell-by-gene matrix or a feature table, since
  EmptyNN requires the raw unfiltered droplet matrix to construct sets P and U
edam_operation: http://edamontology.org/operation_3919
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: EmptyNN
  role: Implements the PU-learning neural network classifier that labels barcodes
    as cell-free or cell-containing and returns per-barcode probabilities
  repo: https://github.com/lkmklsmn/empty_nn
- name: CellBender
  role: Baseline neural-network-based cell caller compared against EmptyNN in benchmarking
- name: EmptyDrops
  role: Baseline Dirichlet-multinomial cell-calling method compared against EmptyNN
- name: Cell Ranger v.2
  role: Baseline total-count-cutoff cell caller (v.2 or lower) used for comparison
- name: Cell Ranger v.3
  role: Baseline cell caller that integrates EmptyDrops, used for comparison
provenance:
  source_task_ids:
  - task_003
  - task_006
  source_papers:
  - doi: 10.1016/j.patter.2021.100311
    title: 'EmptyNN: A neural network based on positive and unlabeled learning to
      remove cell-free droplets and recover lost cells in scRNA-seq data'
  source_content_hash: sha256:b7a80fdfa64945926c6649d6696bf516d7b88703f3600d7307d72d5f16439610
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
  - Cell Ranger v.2
  - Cell Ranger v.3
  - CellBender
  - EmptyDrops
  - EmptyNN
  edam_operation: http://edamontology.org/operation_3919
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 1.0
n_claims: 12
---

# positive-unlabeled-learning-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Train a neural network with positive-unlabeled (PU) learning to classify droplets as cell-free or cell-containing from a raw unfiltered scRNA-seq count matrix, using confidently cell-free barcodes (set P) as positives for the negative class and the ambiguous remainder (set U) as unlabeled data.

## When to use

Use when you have a raw unfiltered count matrix (mtx or h5) from droplet-based scRNA-seq/snRNA-seq and need to distinguish cell-free from cell-containing droplets, including recovering genuine cells with total counts below a fixed cutoff.

## When NOT to use

- When cell-containing droplets with very low total counts may be abundant and the assumption that low-count barcodes are cell-free does not hold without manually adjusting the T threshold
- When the goal is to correct ambient RNA contamination in gene expression estimates — EmptyNN does not estimate corrected expression profiles; use a tool such as SoupX instead
- When the input is already a filtered cell-by-gene matrix or a feature table, since EmptyNN requires the raw unfiltered droplet matrix to construct sets P and U

## Inputs

- Raw unfiltered count matrix in mtx or h5 format (e.g., Read10X_h5 input)
- User-specified threshold T (default 100) defining bona fide empty droplets
- Number of cross-validation folds k (default 10) and training iterations N (default 10)

## Outputs

- Boolean vector of predictions classifying each barcode as cell-free or cell-containing (nn.keep)
- Average prediction probabilities for each barcode in set U

## How to apply

Load the raw unfiltered count matrix (e.g., with Read10X_h5) and transpose it so rows are barcodes/droplets and columns are genes; exclude barcodes with total counts <= 10 from analysis. Define set P as barcodes with total UMI counts below threshold T (default 100) — these are bona fide cell-free droplets used as the reliable negative class — and set U as the remaining higher-count barcodes. Run repeated k-fold cross-validation (default k = 10 folds) for N iterations (default 10), training a neural network (keras) where barcodes from P are labeled cell-free and a fraction of U is labeled cell-containing; each barcode in U is predicted k-1 times per fold and the average prediction probability over N iterations defines the final classification. The rationale is that ambient RNA produces UMI counts in cell-free droplets, so low-count barcodes provide trusted labels for the cell-free class while PU learning resolves the unlabeled set; tuning studies assessed T in 50–400, k in 5–20, and N in 5–20, and EmptyNN was found robust to these choices.

## Related tools

- **EmptyNN** (Implements the PU-learning neural network classifier that labels barcodes as cell-free or cell-containing and returns per-barcode probabilities) — https://github.com/lkmklsmn/empty_nn
- **CellBender** (Baseline neural-network-based cell caller compared against EmptyNN in benchmarking)
- **EmptyDrops** (Baseline Dirichlet-multinomial cell-calling method compared against EmptyNN)
- **Cell Ranger v.2** (Baseline total-count-cutoff cell caller (v.2 or lower) used for comparison)
- **Cell Ranger v.3** (Baseline cell caller that integrates EmptyDrops, used for comparison)

## Examples

```
counts <- t(Read10X_h5("./data/neurons_900_raw.h5")); nn.res <- emptynn(counts, threshold = 100, k = 10, iteration = 10, verbose = TRUE)
```

## Evaluation signals

- On the cell hashing dataset benchmarked against HTO-derived singlet/doublet labels, EmptyNN should reach an AUROC of approximately 94.73% (pROC), exceeding all baselines (79.87%–90.16%)
- On the multiplexed PBMC dataset benchmarked against demuxlet genetic-variation labels, EmptyNN should reach approximately 96.30% AUROC
- Of the 39,842 barcodes in the cell hashing dataset, EmptyNN should classify roughly 20,833 (52.3%) as cell-free and 19,009 (47.7%) as cell-containing
- On the cell hashing dataset, EmptyNN should recover ~885 barcodes with total counts below the original study's 200-count filtering threshold, including platelets with total counts 51–196 (median 93.5)
- Output schema check: the result must contain a boolean cell-free/cell-containing vector plus probabilities for every barcode in set U, and no barcodes with total counts <= 10 should be analyzed

## Limitations

- The key assumption that barcodes below threshold T are cell-free may fail when genuine low-total-count cells exist; the T threshold must be adjusted manually
- Parameter selection (cross-validation folds k and threshold T) is manual; no hyperparameter selection approach is implemented, though the authors report robustness to choices of k and T
- Retained cell-containing droplets may still have a high fraction of mitochondrial reads; low-quality cells must be removed separately in downstream analysis
- EmptyNN does not estimate corrected gene expression profiles; ambient RNA correction requires an external tool such as SoupX

## Evidence
