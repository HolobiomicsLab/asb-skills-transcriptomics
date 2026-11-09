---
name: cross-validation-neural-network-training
description: Use when you need to train or robustness-test EmptyNN's PU-learning classifier
  over a grid of threshold T (50-400), cross-validation folds k (5-20), and iterations
  N (5-20) on raw droplet scRNA-seq count matrices.
when_to_use_negative:
- The data are already filtered cell barcodes rather than a raw unfiltered count matrix
  — EmptyNN's set P/U logic requires the full droplet distribution
- You need ambient RNA correction of gene expression estimates — EmptyNN does not
  correct expression; an external tool such as SoupX is required
- Cell-containing droplets with very low total counts are expected (the low-count-equals-empty
  assumption breaks); the T threshold must be manually adjusted and this sweep alone
  will not resolve the bias
tools:
- name: EmptyNN
  role: R package implementing the PU-learning neural network trained via k-fold cross-validation;
    its emptynn() function exposes threshold, k, and iteration parameters for the
    sweep
  repo: https://github.com/lkmklsmn/empty_nn
- name: keras
  role: R dependency backing the neural network training in each cross-validation
    fold
- name: Matrix
  role: R dependency for handling sparse count matrices used as training input
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.patter.2021.100311
    title: 'EmptyNN: A neural network based on positive and unlabeled learning to
      remove cell-free droplets and recover lost cells in scRNA-seq data'
  source_content_hash: sha256:a39a35a28042faec9cb387b920a7ea95ce7f7492e3a7415685c43cb8e2bf7d1d
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
  - keras
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# cross-validation-neural-network-training

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Trains EmptyNN's PU-learning neural network (R/keras) over k-fold cross-validation with N repeated iterations to classify cell-free vs. cell-containing droplets, and supports robustness sweeps of threshold T, folds k, and iterations N against ground-truth AUROC benchmarks.

## When to use

Use when you need to train or robustness-test EmptyNN's PU-learning classifier over a grid of threshold T (50-400), cross-validation folds k (5-20), and iterations N (5-20) on raw droplet scRNA-seq count matrices.

## When NOT to use

- The data are already filtered cell barcodes rather than a raw unfiltered count matrix — EmptyNN's set P/U logic requires the full droplet distribution
- You need ambient RNA correction of gene expression estimates — EmptyNN does not correct expression; an external tool such as SoupX is required
- Cell-containing droplets with very low total counts are expected (the low-count-equals-empty assumption breaks); the T threshold must be manually adjusted and this sweep alone will not resolve the bias

## Inputs

- Raw unfiltered count matrix in mtx or h5 format (e.g. neurons_900_raw.h5), transposed so rows are barcodes and columns are genes
- Ground-truth labels for AUROC evaluation (HTO-derived singlet/doublet/negative labels for cell hashing; demuxlet genetic labels for multiplexed PBMC data)

## Outputs

- Boolean vector of cell-free vs. cell-containing droplet predictions per barcode
- Prediction probabilities for each barcode in set U
- Sensitivity table of AUROC values and retained/removed barcode counts per (T, k, N) parameter combination, e.g. parameter_sensitivity_auroc.csv

## How to apply

Load the raw unfiltered count matrix (mtx or h5, e.g. neurons_900_raw.h5) and transpose it so rows are barcodes and columns are genes; only barcodes with total counts > 10 enter the analysis. Define set P as barcodes with total UMI counts below threshold T (default 100) — these are labeled bona fide cell-free droplets — and set U as the remaining barcodes. Train the keras neural network with k-fold cross-validation (default k = 10), so each barcode in U is predicted k-1 times per fold, repeating the whole process for N iterations (default 10); the averaged prediction probability per barcode in U defines the final cell-free/cell-containing call. For sensitivity analysis, sweep T from 50 to 400, k from 5 to 20, and N from 5 to 20, recording boolean predictions and set-U probabilities for each combination. Judge correctness by comparing AUROC against the published benchmarks (94.73% for HTO-based cell hashing ground truth, 96.30% for demuxlet genetic-variation labels); results should remain stable across k and N, with best performance for T between 100 and 300.

## Related tools

- **EmptyNN** (R package implementing the PU-learning neural network trained via k-fold cross-validation; its emptynn() function exposes threshold, k, and iteration parameters for the sweep) — https://github.com/lkmklsmn/empty_nn
- **keras** (R dependency backing the neural network training in each cross-validation fold)
- **Matrix** (R dependency for handling sparse count matrices used as training input)

## Examples

```
nn.res <- emptynn(counts, threshold = 100, k = 10, iteration = 10, verbose = TRUE)
```

## Evaluation signals

- AUROC against HTO-derived labels for the cell hashing dataset should reproduce the reported benchmark of 94.73%
- AUROC against demuxlet genetic labels for the multiplexed PBMC dataset should reproduce the reported benchmark of 96.30%
- Performance should be invariant to the tested ranges of k (5-20) and N (5-20), per Figures S9B, S9C, S9F, and S9G
- AUROC should peak for threshold T values between 100 and 300, and larger T should monotonically decrease the number of predicted cell-containing droplets
- Only barcodes with total counts greater than 10 should appear in the training/analysis set

## Limitations

- Parameter selection (k folds and T threshold) must be specified manually; the paper defers automated hyperparameter selection to future work
- The assumption that low total-count barcodes are cell-free may not hold when genuine low-count cells exist, biasing classification
- Retained cell-containing droplets may carry a high mitochondrial read fraction and require separate low-quality-cell removal downstream

## Evidence

- [readme] averaged prediction probability defines the call: [evidence span withheld — anchor status: external_doc]
