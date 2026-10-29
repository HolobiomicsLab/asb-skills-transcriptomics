---
name: auroc-benchmark-evaluation-of-scrnaseq-barcodes
description: Use when benchmarking a scRNA-seq cell-calling/filtering method against
  ground-truth barcode labels (e.g. HTO-derived singlet/doublet/negative or demuxlet
  genetic labels) via AUROC, especially under a hyperparameter sweep.
when_to_use_negative:
- The dataset has no independent ground-truth labels (e.g. no HTO counts or genetic
  demultiplexing), since AUROC cannot be computed reliably
- You need ambient-RNA-corrected expression profiles or downstream low-quality-cell
  removal — EmptyNN does not correct gene expression (use SoupX) and retained droplets
  may have high mitochondrial fractions
- Input is already a filtered/processed cell-by-gene matrix rather than the raw unfiltered
  count matrix, since set P must be defined from total UMI counts below threshold
  T
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0091
tools:
- name: EmptyNN
  role: Cell-calling algorithm whose hyperparameters (T, k, N) are swept; produces
    boolean keep/cell-free predictions and set-U probabilities scored by AUROC
  repo: https://github.com/lkmklsmn/empty_nn
- name: keras
  role: R dependency backing EmptyNN's neural network classifier during the cross-validated
    PU-learning training iterations
- name: Matrix
  role: R dependency for handling sparse raw count matrices loaded as mtx or h5
- name: demuxlet
  role: Infers per-barcode genetic labels used as ground truth for AUROC evaluation
    on the multiplexed PBMC dataset
  repo: https://github.com/yelabucsf/demuxlet_paper_code
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.patter.2021.100311
    title: 'EmptyNN: A neural network based on positive and unlabeled learning to
      remove cell-free droplets and recover lost cells in scRNA-seq data'
  source_content_hash: sha256:cb1ffe44cfb0d2926c11bb4bf50763713a7cee5cd628c0ae3828a8b527af7f60
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
  - demuxlet
  - keras
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# auroc-benchmark-evaluation-of-scrnaseq-barcodes

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A sensitivity-analysis skill that sweeps EmptyNN's PU-learning hyperparameters (count threshold T, cross-validation folds k, training iterations N) and scores each setting's cell-calling predictions against ground-truth barcode labels using AUROC.

## When to use

Use when benchmarking a scRNA-seq cell-calling/filtering method against ground-truth barcode labels (e.g. HTO-derived singlet/doublet/negative or demuxlet genetic labels) via AUROC, especially under a hyperparameter sweep.

## When NOT to use

- The dataset has no independent ground-truth labels (e.g. no HTO counts or genetic demultiplexing), since AUROC cannot be computed reliably
- You need ambient-RNA-corrected expression profiles or downstream low-quality-cell removal — EmptyNN does not correct gene expression (use SoupX) and retained droplets may have high mitochondrial fractions
- Input is already a filtered/processed cell-by-gene matrix rather than the raw unfiltered count matrix, since set P must be defined from total UMI counts below threshold T

## Inputs

- Raw unfiltered count matrix in mtx or h5 format (e.g. neurons_900_raw.h5), transposed so rows are barcodes and columns are genes
- HTO-derived singlet/doublet/negative ground-truth labels for the cell hashing dataset (Stoeckius et al.)
- Demuxlet genetic-variation labels for the multiplexed PBMC dataset (Kang et al., SRR5398237)
- Per-combination EmptyNN predictions (boolean cell-free/cell-containing vector) and set-U probabilities

## Outputs

- AUROC value for each (T, k, N) parameter combination per dataset
- Retained/removed barcode counts per parameter combination
- Aggregated sensitivity table written to parameter_sensitivity_auroc.csv

## How to apply

Download the benchmark raw unfiltered count matrices (e.g. neurons_900_raw.h5 and the cell hashing dataset) with the package's download_data.sh script, then transpose each matrix so rows are barcodes and columns are genes. Run emptynn() over a parameter grid: threshold T (default 100; 50–400), k-fold CV folds k (default 10; 5–20), and training iterations N (default 10; 5–20), recording the boolean cell-free/cell-containing predictions and set-U probabilities for each combination. Evaluate each setting's AUROC against ground-truth labels: HTO-derived singlet/doublet/negative labels for the cell hashing dataset and demuxlet genetic labels for the multiplexed PBMC dataset. Aggregate AUROC values plus retained/removed barcode counts per combination into a single sensitivity table (parameter_sensitivity_auroc.csv) and check stability against the reported benchmarks of 94.73% (cell hashing) and 96.30% (genetic variation). Rationale: EmptyNN's PU-learning scheme labels barcodes below T as set P (cell-free) and trains a neural network on set U, so AUROC against independent labels reveals how sensitive the cell-calling boundary is to T, k, and N.

## Related tools

- **EmptyNN** (Cell-calling algorithm whose hyperparameters (T, k, N) are swept; produces boolean keep/cell-free predictions and set-U probabilities scored by AUROC) — https://github.com/lkmklsmn/empty_nn
- **keras** (R dependency backing EmptyNN's neural network classifier during the cross-validated PU-learning training iterations)
- **Matrix** (R dependency for handling sparse raw count matrices loaded as mtx or h5)
- **demuxlet** (Infers per-barcode genetic labels used as ground truth for AUROC evaluation on the multiplexed PBMC dataset) — https://github.com/yelabucsf/demuxlet_paper_code

## Examples

```
nn.res <- emptynn(counts, threshold = 100, k = 10, iteration = 10, verbose = TRUE)
```

## Evaluation signals

- Cell hashing AUROC should be ~94.73% and genetic-variation AUROC ~96.30% near default settings (T=100, k=10, N=10)
- AUROC should remain stable across k in 5–20 and N in 5–20, indicating robustness to folds and iterations
- Highest AUROC should occur for T values between 100 and 300; AUROC degrades for T outside this range
- Increasing T should monotonically decrease the number of predicted cell-containing droplets
- parameter_sensitivity_auroc.csv must contain one row per swept (T, k, N) combination with AUROC and retained-barcode counts populated

## Limitations

- Benchmarking ground truth is limited to cell hashing or genetic-variation labels; methods' classifications may overlap highly, limiting biological interpretation of differences
- The assumption that low-total-count barcodes are cell-free may not hold when genuine cell-containing droplets have very low counts; the user must manually adjust T to mitigate bias
- Parameter selection (k and T) is manual with no automated hyperparameter-selection approach implemented; robustness to k and T is claimed but selection is deferred to future work

## Evidence

- [results] We assessed different threshold values T (default: 100, from 50 to 400), numbers of cross-validation folds k (default: 10, from 5 to 20), and numbers of training iterations N (default: 10, from: "We assessed different threshold values T (default: 100, from 50 to 400), numbers of cross-validation folds k (default: 10, from 5 to 20), and numbers of training iterations N (default: 10, from"
- [results] we integrated information derived from the HTO counts. As described above, the HTO counts provide singlet, doublet, and negative labels for each barcode.: "we integrated information derived from the HTO counts. As described above, the HTO counts provide singlet, doublet, and negative labels for each barcode."
- [discussion] we applied demuxlet to infer the label of each barcode and evaluated the performance of the cell-calling methods.: "we applied demuxlet to infer the label of each barcode and evaluated the performance of the cell-calling methods."
- [abstract] achieved an area under the receiver operating characteristics of 94.73% and 96.30%, respectively: "achieved an area under the receiver operating characteristics of 94.73% and 96.30%, respectively"
- [readme] nn.res <- emptynn(counts, threshold = 100, k = 10, iteration = 10, verbose = TRUE): "nn.res <- emptynn(counts, threshold = 100, k = 10, iteration = 10, verbose = TRUE)"
