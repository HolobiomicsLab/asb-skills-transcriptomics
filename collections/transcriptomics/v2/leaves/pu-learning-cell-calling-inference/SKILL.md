---
name: pu-learning-cell-calling-inference
description: Use when you have a raw unfiltered droplet-based scRNA-seq count matrix
  (h5/mtx) and need to distinguish cell-free from cell-containing droplets, especially
  to recover genuine low-count cells below a total-count filtering threshold.
when_to_use_negative:
- You need ambient RNA-corrected gene expression values — EmptyNN does not estimate
  corrected expression profiles; use SoupX in addition
- Your dataset contains many genuine cell-containing droplets with very low total
  counts near the empty-droplet regime — the assumption that barcodes below T are
  cell-free may not hold; adjust T manually
- You already have a filtered cell x gene matrix from a completed cell-calling step
  — re-running EmptyNN cell calling would be redundant
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3474
tools:
- name: EmptyNN
  role: PU-learning neural network that classifies barcodes as cell-free or cell-containing
    droplets from the raw count matrix
  repo: https://github.com/lkmklsmn/empty_nn
- name: Seurat
  role: 'Downstream processing of EmptyNN-retained barcodes: mitochondrial-fraction
    filtering, normalization, HVG detection, PCA, clustering, and t-SNE via runSeurat'
- name: Cell Ranger v.2
  role: Reference total-count cutoff method whose threshold (e.g., 200) defines the
    barcodes EmptyNN recovers; not run by this skill
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.patter.2021.100311
    title: 'EmptyNN: A neural network based on positive and unlabeled learning to
      remove cell-free droplets and recover lost cells in scRNA-seq data'
  source_content_hash: sha256:6e05a2e342284da93f119514b247c773819f2626d900feabdb7087922c009e56
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
  - EmptyNN
  - Seurat
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# pu-learning-cell-calling-inference

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Cell-calling skill that applies EmptyNN, a positive-unlabeled (PU) learning neural network, to a raw unfiltered scRNA-seq count matrix to classify barcodes as cell-free versus cell-containing droplets, removing empty droplets while recovering low-total-count cells lost by total-count cutoffs like Cell Ranger v.2.

## When to use

Use when you have a raw unfiltered droplet-based scRNA-seq count matrix (h5/mtx) and need to distinguish cell-free from cell-containing droplets, especially to recover genuine low-count cells below a total-count filtering threshold.

## When NOT to use

- You need ambient RNA-corrected gene expression values — EmptyNN does not estimate corrected expression profiles; use SoupX in addition
- Your dataset contains many genuine cell-containing droplets with very low total counts near the empty-droplet regime — the assumption that barcodes below T are cell-free may not hold; adjust T manually
- You already have a filtered cell x gene matrix from a completed cell-calling step — re-running EmptyNN cell calling would be redundant

## Inputs

- Raw unfiltered count matrix in h5 or mtx format (e.g., Read10X_h5 output)
- Optional: user-specified threshold T (default 100), number of cross-validation folds k (default 10), number of training iterations N (default 10)

## Outputs

- Boolean vector of cell-free vs cell-containing droplet predictions for each barcode
- Prediction probabilities for each barcode in set U
- Set of recovered barcodes below the external filtering threshold (e.g., 885 barcodes below 200 total counts in the cell hashing dataset)
- Seurat object with t-SNE embedding of retained cells (e.g., tsne_recovered_barcodes.png)

## How to apply

Load the raw unfiltered count matrix (h5 or mtx) and transpose it so rows are barcodes and columns are genes. Run emptynn() with default parameters (threshold T = 100, k = 10 cross-validation folds, N = 10 iterations, batch_size and epochs per package defaults); EmptyNN defines barcodes with total counts below T as the positive set P of bona fide empty droplets, trains a neural network classifier via k-fold cross-validation repeated N times, and averages prediction probabilities over set U to label each barcode. Exclude barcodes with total counts <= 10 from analysis. Extract recovered barcodes — those retained by EmptyNN whose total counts fall below an external cutoff such as the Cell Ranger v.2-style threshold of 200 used in the original cell hashing study — and process the retained matrix downstream in Seurat via runSeurat (resolution = 0.2) with mitochondrial-fraction filtering, normalization, HVG detection, PCA, clustering, and t-SNE. Rationale: ambient RNA produces UMI counts in cell-free droplets, making simple total-count cutoffs discard genuine low-count cells such as platelets (recovered platelets had total counts 51–196, median 93.5).

## Related tools

- **EmptyNN** (PU-learning neural network that classifies barcodes as cell-free or cell-containing droplets from the raw count matrix) — https://github.com/lkmklsmn/empty_nn
- **Seurat** (Downstream processing of EmptyNN-retained barcodes: mitochondrial-fraction filtering, normalization, HVG detection, PCA, clustering, and t-SNE via runSeurat)
- **Cell Ranger v.2** (Reference total-count cutoff method whose threshold (e.g., 200) defines the barcodes EmptyNN recovers; not run by this skill)

## Examples

```
counts <- t(Read10X_h5("./data/neurons_900_raw.h5")); nn.res <- emptynn(counts, threshold = 100, k = 10, iteration = 10); retained <- runSeurat(counts = counts[, nn.res$nn.keep], resolution = 0.2)
```

## Evaluation signals

- In the cell hashing dataset, EmptyNN should classify 20,833 of 39,842 barcodes (52.3%) as cell-free and 19,009 (47.7%) as cell-containing
- Recovering barcodes below the original Cell Ranger v.2 threshold of 200 should yield ~885 recovered barcodes (excluding total counts <= 10)
- t-SNE of recovered low-total-count barcodes should resolve five distinct clusters, with differential expression assigning cell-type identities
- The recovered platelet cluster should correlate with the PBMC 3k platelet reference at Rho = 0.95 (Pearson, p < 2.2e-06)
- Predictions should be stable across reasonable hyperparameter choices (T 50–400, k 5–20, N 5–20 iterations)

## Limitations

- The assumption that low total count barcodes are cell-free may not hold when genuine low-count cells exist; the T threshold must be manually adjusted in such cases
- Parameter selection (cross-validation folds k and threshold T) is manual; no hyperparameter selection approach is implemented, though the authors claim robustness to various choices
- Retained cell-containing droplets may have a high fraction of mitochondrial reads; low-quality cells passing the filter must be removed in downstream analysis
- EmptyNN does not correct ambient RNA contamination in gene expression estimates; apply external tools such as SoupX for correction

## Evidence
