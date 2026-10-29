---
name: frequent-gene-feature-selection
description: Use when building an EmptyNN-style classifier on a raw unfiltered scRNA-seq
  count matrix and you need a data-driven, ambient-informed gene subset (top frequent
  genes of empty droplets) instead of the full gene space.
when_to_use_negative:
- The input is already a filtered or feature-selected matrix rather than the raw unfiltered
  count matrix — selection of frequent genes requires the full droplet space including
  cell-free barcodes
- The goal is correcting ambient RNA contamination in expression estimates rather
  than cell calling — EmptyNN does not estimate corrected gene expression profiles;
  tools such as SoupX are required
- Barcodes have total counts ≤ 10, which are excluded from EmptyNN analysis and provide
  too little signal for frequency-based gene selection
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: EmptyNN
  role: R package whose emptynn() function internally performs the frequent-gene feature
    restriction on set P barcodes before PU-learning
  repo: https://github.com/lkmklsmn/empty_nn
- name: Seurat
  role: Used to load and transpose the raw unfiltered count matrix (Read10X_h5) prior
    to gene selection and cell calling
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.patter.2021.100311
    title: 'EmptyNN: A neural network based on positive and unlabeled learning to
      remove cell-free droplets and recover lost cells in scRNA-seq data'
  source_content_hash: sha256:a73ed58fa9a18c4323f9684132e76297042a21662e9c1b7729e3f29f921b6e6b
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# Frequent-gene feature selection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A preprocessing step within the EmptyNN cell-calling pipeline that restricts the classifier's feature space to the 2,000 most frequently detected genes among set P barcodes (bona fide cell-free droplets with total UMI counts below threshold T = 100), reducing noise and dimensionality before PU-learning.

## When to use

Use when building an EmptyNN-style classifier on a raw unfiltered scRNA-seq count matrix and you need a data-driven, ambient-informed gene subset (top frequent genes of empty droplets) instead of the full gene space.

## When NOT to use

- The input is already a filtered or feature-selected matrix rather than the raw unfiltered count matrix — selection of frequent genes requires the full droplet space including cell-free barcodes
- The goal is correcting ambient RNA contamination in expression estimates rather than cell calling — EmptyNN does not estimate corrected gene expression profiles; tools such as SoupX are required
- Barcodes have total counts ≤ 10, which are excluded from EmptyNN analysis and provide too little signal for frequency-based gene selection

## Inputs

- Raw unfiltered scRNA-seq count matrix in mtx or h5 format (e.g., ./data/neurons_900_raw.h5)
- Set P barcodes: droplets with total UMI counts below threshold T (default 100) and above the minimum count cutoff (> 10)

## Outputs

- A reduced gene-by-cell feature matrix restricted to the 2,000 most frequent genes among set P barcodes
- The gene index/subset used downstream by the emptynn() PU-bagging neural network classifier

## How to apply

First load the raw unfiltered count matrix (e.g., an .h5 file via Seurat's Read10X_h5) and transpose it so rows are barcodes and columns are genes, keeping only barcodes with total counts greater than 10. Partition barcodes into set P (total UMI counts below threshold T, default 100; bona fide cell-free droplets) and set U (remaining unlabeled barcodes). Then restrict the feature space to the 2,000 most frequent genes detected among set P barcodes — the rationale is that genes recurring across empty droplets characterize the ambient RNA background the classifier must learn to discriminate from cell-containing droplets. This reduced matrix feeds the PU-bagging loop of emptynn() (k = 10 folds, N = 10 iterations), where set P is the positive cell-free class and set U the unlabeled class.

## Related tools

- **EmptyNN** (R package whose emptynn() function internally performs the frequent-gene feature restriction on set P barcodes before PU-learning) — https://github.com/lkmklsmn/empty_nn
- **Seurat** (Used to load and transpose the raw unfiltered count matrix (Read10X_h5) prior to gene selection and cell calling)

## Examples

```
counts <- t(Read10X_h5("./data/neurons_900_raw.h5", use.names = TRUE, unique.features = TRUE)); nn.res <- emptynn(counts, threshold = 100, k = 10, iteration = 10, verbose = TRUE)
```

## Evaluation signals

- The feature matrix contains exactly 2,000 genes, all drawn from genes detected among set P (low-count, T = 100) barcodes
- Barcodes with total counts ≤ 10 are absent from the matrix before selection
- Set P is defined strictly as barcodes below threshold T = 100, and set U retains the remaining barcodes
- Downstream emptynn() run completes and returns a boolean cell-free/cell-containing prediction vector plus per-barcode probabilities for set U
- After selection, all retained barcodes are scored against the ambient-RNA-defined feature space, so predicted cell-free droplets should show high average probability similarity to set P

## Limitations

- Selecting genes by frequency in set P inherits the key assumption that low-total-count barcodes are cell-free; this may not hold when genuine low-count cells exist, requiring manual adjustment of the T threshold
- Frequent-gene selection targets ambient RNA characterization and does not correct ambient contamination in expression values
- The number of selected genes (2,000) and other parameters (k, T) must be specified manually; no hyperparameter selection approach is implemented

## Evidence
