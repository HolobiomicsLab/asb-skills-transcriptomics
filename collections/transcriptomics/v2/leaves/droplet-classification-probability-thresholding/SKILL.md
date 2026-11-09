---
name: droplet-classification-probability-thresholding
description: Use when a raw unfiltered count matrix (mtx or h5) from droplet-based
  scRNA-seq must be split into cell-free vs cell-containing droplets and ambient RNA
  causes low-count droplets that simple total-count cutoffs misclassify.
when_to_use_negative:
- When gene expression estimates need ambient RNA correction — EmptyNN does not estimate
  corrected expression profiles; use a tool like SoupX afterward
- When the dataset contains genuine cell-containing droplets with very low total counts,
  since the assumption that low-count barcodes are cell-free may not hold and requires
  manual adjustment of T
- When barcodes have total counts <= 10 — these are excluded from EmptyNN analysis
  and cannot be classified
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: EmptyNN
  role: Implements the PU-bagging neural network that produces averaged set-U probabilities
    and the boolean cell-free/cell-containing prediction vector
  repo: https://github.com/lkmklsmn/empty_nn
- name: Seurat
  role: Reads the raw h5 count matrix (Read10X_h5) and transposes it so rows are barcodes
    and columns are genes prior to emptynn()
- name: keras
  role: Provides the three-layer neural network (128/64 hidden neurons, 2-neuron output)
    trained with RMSprop and binary cross-entropy inside the PU-bagging loop
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.patter.2021.100311
    title: 'EmptyNN: A neural network based on positive and unlabeled learning to
      remove cell-free droplets and recover lost cells in scRNA-seq data'
  source_content_hash: sha256:755febd5029f06b707f556be2fda6fc514be06f0a62988394ce53460b1f5bb05
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
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# droplet-classification-probability-thresholding

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Classifies barcodes from droplet-based scRNA-seq as cell-free or cell-containing by averaging PU-learning neural network predicted probabilities over N iterations and applying a 0.5 probability cutoff to set-U barcodes, using total-count threshold T (default 100) to define bona fide empty droplets.

## When to use

Use when a raw unfiltered count matrix (mtx or h5) from droplet-based scRNA-seq must be split into cell-free vs cell-containing droplets and ambient RNA causes low-count droplets that simple total-count cutoffs misclassify.

## When NOT to use

- When gene expression estimates need ambient RNA correction — EmptyNN does not estimate corrected expression profiles; use a tool like SoupX afterward
- When the dataset contains genuine cell-containing droplets with very low total counts, since the assumption that low-count barcodes are cell-free may not hold and requires manual adjustment of T
- When barcodes have total counts <= 10 — these are excluded from EmptyNN analysis and cannot be classified

## Inputs

- Raw unfiltered scRNA-seq count matrix in mtx or h5 format (barcodes x genes after transposition)
- User-specified total-count threshold T (default 100), fold number k (default 10), and iteration count N (default 10)

## Outputs

- Boolean vector of predictions labeling each barcode as cell-free or cell-containing (saved as emptynn_predictions.csv)
- Per-barcode prediction probabilities for each barcode in set U

## How to apply

Load the raw unfiltered count matrix (e.g., Read10X_h5 on ./data/neurons_900_raw.h5), transpose so rows are barcodes, and keep only barcodes with total counts greater than 10. Define set P as barcodes with total UMI counts below user-specified threshold T (default 100) — bona fide cell-free droplets — and the rest as unlabeled set U; restrict features to the 2,000 most frequent genes among set P. Run the emptynn() PU-bagging loop: N=10 iterations of k=10-fold cross-validation training a three-layer keras network (128 and 64 hidden neurons, 2-neuron output, RMSprop optimizer, binary cross-entropy loss) with P as the positive cell-free class and U as the unlabeled class, collecting predicted probabilities for held-out set-U barcodes. Average the N=10 probabilities per set-U barcode and threshold at > 0.5 to call cell-containing droplets; all other barcodes, including set P, are labeled cell-free. Hyperparameters T (50–400), k (5–20), and N (5–20) are selected manually; the authors report the algorithm is robust across these ranges.

## Related tools

- **EmptyNN** (Implements the PU-bagging neural network that produces averaged set-U probabilities and the boolean cell-free/cell-containing prediction vector) — https://github.com/lkmklsmn/empty_nn
- **Seurat** (Reads the raw h5 count matrix (Read10X_h5) and transposes it so rows are barcodes and columns are genes prior to emptynn())
- **keras** (Provides the three-layer neural network (128/64 hidden neurons, 2-neuron output) trained with RMSprop and binary cross-entropy inside the PU-bagging loop)

## Examples

```
library(EmptyNN); library(Seurat); counts <- t(Read10X_h5('./data/neurons_900_raw.h5')); nn.res <- emptynn(counts, threshold = 100, k = 10, iteration = 10, verbose = TRUE)
```

## Evaluation signals

- Averaged set-U probabilities should be bimodal, with barcodes > 0.5 called cell-containing and the remainder, plus all set-P barcodes, called cell-free
- On benchmark datasets with HTO-derived or demuxlet-derived ground truth, expect AUROC near 94.73% (cell hashing) and 96.30% (genetic variation)
- Verify that only barcodes with total counts > 10 are analyzed and that set P consists of barcodes with total UMI counts below T=100
- Check output files contain both the boolean prediction vector and the set-U probabilities, matching the emptynn_predictions.csv schema
- In the cell hashing dataset, expect roughly half of the 39,842 evaluated barcodes classified as cell-free (20,833, 52.3%) versus cell-containing (19,009, 47.7%)

## Limitations

- The key assumption that low-total-count barcodes are cell-free may not hold when genuine low-count cells exist; the user must manually adjust the T threshold to mitigate bias
- Parameter selection (cross-validation folds k and threshold T) is manual — no automated hyperparameter selection is implemented, though the authors claim robustness to various k and T
- Retained cell-containing droplets may carry a high fraction of mitochondrial reads; low-quality cells passing the filter must be removed in downstream analysis
- EmptyNN does not correct ambient RNA contamination in gene expression estimates; apply SoupX for corrected profiles

## Evidence

- [readme] README workflow description: [evidence span withheld — anchor status: external_doc]
