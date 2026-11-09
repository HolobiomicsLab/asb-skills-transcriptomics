---
name: positive-unlabeled-neural-network-cell-calling
description: Use when given a raw unfiltered gene-by-barcode count matrix (mtx or
  h5) from droplet-based scRNA-seq and you need to classify barcodes as cell-free
  vs cell-containing, or recover cell-containing droplets below common count cutoffs.
when_to_use_negative:
- The input is already a filtered cell-by-gene matrix or a processed Seurat/AnnData
  object — EmptyNN requires the raw unfiltered count matrix including empty droplets
- The goal is correcting ambient RNA contamination in retained cells' expression profiles
  — EmptyNN does not estimate corrected gene expression; use SoupX downstream
- The dataset contains many genuine cell-containing droplets with very low total counts
  near T — the low-count-means-cell-free assumption may mislabel them and T must be
  manually adjusted
edam_operation: http://edamontology.org/operation_3803
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: EmptyNN
  role: PU-learning neural network that classifies barcodes as cell-free or cell-containing
    using threshold T, k folds, and N iterations
  repo: https://github.com/lkmklsmn/empty_nn
- name: demuxlet
  role: Infers per-barcode genotype labels and singlet probabilities from the merged
    VCF to provide ground truth for AUROC benchmarking on the multiplexed PBMC dataset
  repo: https://github.com/yelabucsf/demuxlet_paper_code/
- name: EmptyDrops
  role: Baseline comparator using a Dirichlet-multinomial model of the cell-free profile
    (86.43% AUROC on the multiplexed PBMC benchmark)
- name: CellBender
  role: Neural-network-based baseline comparator (58.02% AUROC); expected-number-of-cells
    and total-droplets-included set via barcode rank plot inspection
- name: keras
  role: R dependency used to train EmptyNN's neural network classifier
provenance:
  source_task_ids:
  - task_004
  - task_005
  source_papers:
  - doi: 10.1016/j.patter.2021.100311
    title: 'EmptyNN: A neural network based on positive and unlabeled learning to
      remove cell-free droplets and recover lost cells in scRNA-seq data'
  source_content_hash: sha256:c8313819f0b93856d25400ffc22cc7b239a2736d736b140ab7314944aa572d8e
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
  - demuxlet
  - keras
  edam_operation: http://edamontology.org/operation_3803
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# positive-unlabeled-neural-network-cell-calling

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Cell-calling skill based on EmptyNN, which trains a neural network with positive-unlabeled (PU) learning to distinguish cell-free from cell-containing droplets in raw unfiltered scRNA-seq count matrices, removing empty droplets while recovering low-count cells lost by threshold-based methods.

## When to use

Use when given a raw unfiltered gene-by-barcode count matrix (mtx or h5) from droplet-based scRNA-seq and you need to classify barcodes as cell-free vs cell-containing, or recover cell-containing droplets below common count cutoffs.

## When NOT to use

- The input is already a filtered cell-by-gene matrix or a processed Seurat/AnnData object — EmptyNN requires the raw unfiltered count matrix including empty droplets
- The goal is correcting ambient RNA contamination in retained cells' expression profiles — EmptyNN does not estimate corrected gene expression; use SoupX downstream
- The dataset contains many genuine cell-containing droplets with very low total counts near T — the low-count-means-cell-free assumption may mislabel them and T must be manually adjusted

## Inputs

- Raw unfiltered gene-by-barcode count matrix in mtx or h5 format (e.g., neurons_900_raw.h5)
- Optional ground-truth barcode labels: HTO-derived singlet/doublet/negative labels (cell hashing dataset) or demuxlet-inferred genotype labels with singlet probabilities (multiplexed PBMC dataset, SRR5398237)

## Outputs

- Boolean vector of predictions labeling each barcode as cell-free or cell-containing (nn.keep)
- Set-U probability for each barcode (average prediction probability across N iterations)
- Optional AUROC benchmark table against ground-truth labels (e.g., auroc_results.csv, parameter_sensitivity_auroc.csv)

## How to apply

Load the raw unfiltered count matrix (mtx or h5), transpose it so rows are barcodes and columns are genes, and exclude barcodes with total counts <= 10. Define set P as barcodes with total UMI counts below the threshold T (default 100) — these are bona fide cell-free droplets — and set U as the remaining barcodes; run emptynn() with k = 10 cross-validation folds and N = 10 PU-bagging iterations, during which each barcode in U is predicted k-1 times per fold and the averaged probability labels it cell-free or cell-containing. Validate on ground-truth labels where available: HTO-derived singlet/doublet/negative labels for the cell hashing dataset (expected AUROC 94.73%) or demuxlet genetic labels for the multiplexed PBMC dataset (expected AUROC 96.30%, retaining 6,025 cell-containing droplets). Hyperparameter sweeps show robustness across k (5–20) and N (5–20), with best performance for T between 100 and 300. For comparison benchmarks run Cell Ranger v.2 (82.94% AUROC), EmptyDrops (86.43%), Cell Ranger v.3 (83.03%), and CellBender (58.02%) on the same raw matrix.

## Related tools

- **EmptyNN** (PU-learning neural network that classifies barcodes as cell-free or cell-containing using threshold T, k folds, and N iterations) — https://github.com/lkmklsmn/empty_nn
- **demuxlet** (Infers per-barcode genotype labels and singlet probabilities from the merged VCF to provide ground truth for AUROC benchmarking on the multiplexed PBMC dataset) — https://github.com/yelabucsf/demuxlet_paper_code/
- **EmptyDrops** (Baseline comparator using a Dirichlet-multinomial model of the cell-free profile (86.43% AUROC on the multiplexed PBMC benchmark))
- **CellBender** (Neural-network-based baseline comparator (58.02% AUROC); expected-number-of-cells and total-droplets-included set via barcode rank plot inspection)
- **keras** (R dependency used to train EmptyNN's neural network classifier)

## Examples

```
counts <- t(Read10X_h5("./data/neurons_900_raw.h5")); nn.res <- emptynn(counts, threshold = 100, k = 10, iteration = 10, verbose = TRUE)
```

## Evaluation signals

- AUROC against demuxlet ground truth reproduces 96.30% (EmptyNN) vs 82.94% (Cell Ranger v.2), 86.43% (EmptyDrops), 83.03% (Cell Ranger v.3), 58.02% (CellBender) on the multiplexed PBMC dataset within reasonable tolerance
- AUROC against HTO singlet/doublet/negative labels on the cell hashing dataset is ~94.73%
- On the cell hashing dataset, of 39,842 barcodes EmptyNN calls 20,833 (52.3%) cell-free and 19,009 (47.7%) cell-containing; it recovers 885 barcodes below the original 200-total-count threshold
- Parameter sensitivity table shows stable AUROC across k = 5–20 and N = 5–20, with best performance for T in 100–300 and fewer retained cells at higher T
- Output schema check: run returns a boolean keep vector plus probabilities defined only for barcodes in set U

## Limitations

- The assumption that low-total-count barcodes are cell-free may fail when genuine low-count cells exist; the T threshold must be manually adjusted in such cases
- Parameter selection (cross-validation folds k and threshold T) is manual; no automated hyperparameter selection is implemented
- Retained cell-containing droplets may have high mitochondrial read fractions and require separate low-quality cell filtering downstream
- EmptyNN does not correct ambient RNA contamination in expression estimates; apply SoupX or similar downstream
- Benchmarking ground truth is limited to cell hashing or genetic variation labels, and high overlap between methods may limit biological interpretation of differences

## Evidence
