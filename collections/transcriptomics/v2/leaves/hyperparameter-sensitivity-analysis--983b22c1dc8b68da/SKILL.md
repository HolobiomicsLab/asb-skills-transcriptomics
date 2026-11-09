---
name: hyperparameter-sensitivity-analysis
description: Use when benchmarking a cell-calling or PU-learning classifier whose
  defaults (e.g., EmptyNN T=100, k=10, N=10) need sensitivity testing against ground-truth
  labels before deployment on new data.
when_to_use_negative:
- The dataset contains genuine cell-containing droplets with very low total counts,
  where the low-count-implies-empty assumption behind threshold T breaks down and
  T must be manually adjusted rather than swept
- Ground-truth labels (HTO or genetic demuxlet labels) are unavailable, so per-setting
  AUROC cannot be computed
- The goal is ambient RNA correction rather than cell calling — EmptyNN does not estimate
  corrected expression profiles and tools like SoupX are needed instead
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: EmptyNN
  role: PU-learning cell-calling algorithm whose threshold T, cross-validation folds
    k, and iteration count N are swept via emptynn()
  repo: https://github.com/lkmklsmn/empty_nn
- name: keras
  role: R dependency backing the neural network trained within each emptynn() run
    of the sweep
- name: Matrix
  role: R dependency for handling the sparse transposed count matrices fed to emptynn()
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.patter.2021.100311
    title: 'EmptyNN: A neural network based on positive and unlabeled learning to
      remove cell-free droplets and recover lost cells in scRNA-seq data'
  source_content_hash: sha256:5dfa469777351858ca071a57ce30f999e9f995c01699c631fa1ee002fc9074a8
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
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# hyperparameter-sensitivity-analysis

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Reproduce ablation sweeps of EmptyNN's PU-bagging hyperparameters — count threshold T, cross-validation folds k, and training iterations N — to quantify how robust cell-calling AUROC is to each setting.

## When to use

Use when benchmarking a cell-calling or PU-learning classifier whose defaults (e.g., EmptyNN T=100, k=10, N=10) need sensitivity testing against ground-truth labels before deployment on new data.

## When NOT to use

- The dataset contains genuine cell-containing droplets with very low total counts, where the low-count-implies-empty assumption behind threshold T breaks down and T must be manually adjusted rather than swept
- Ground-truth labels (HTO or genetic demuxlet labels) are unavailable, so per-setting AUROC cannot be computed
- The goal is ambient RNA correction rather than cell calling — EmptyNN does not estimate corrected expression profiles and tools like SoupX are needed instead

## Inputs

- Raw unfiltered scRNA-seq count matrix in mtx or h5 format (e.g., neurons_900_raw.h5, cell hashing dataset)
- Ground-truth barcode labels: HTO-derived singlet/doublet/negative labels (cell hashing dataset) or demuxlet genetic labels (multiplexed PBMC dataset)
- Parameter grid specification: T in 50–400, k in 5–20, N in 5–20

## Outputs

- Boolean cell-free/cell-containing predictions per barcode for each parameter combination
- Set-U probabilities for each barcode per run
- Sensitivity table of AUROC values and retained/removed barcode counts per combination, written to parameter_sensitivity_auroc.csv

## How to apply

Download the raw unfiltered count matrices (e.g., neurons_900_raw.h5 and the cell hashing dataset) via the repo's download_data.sh script, transpose so rows are barcodes and columns are genes, and run emptynn() over a parameter grid: T from 50 to 400 (default 100), k from 5 to 20 (default 10), and iterations N from 5 to 20 (default 10). Record the boolean cell-free/cell-containing predictions and set-U probabilities for every combination. Evaluate each setting's AUROC against ground truth — HTO-derived singlet/doublet/negative labels for the cell hashing dataset and demuxlet genetic labels for the multiplexed PBMC dataset (SRR5398237). Aggregate AUROC and retained/removed barcode counts per combination into a sensitivity table, verifying stability relative to the reported benchmarks of 94.73% (cell hashing) and 96.30% (genetic variation), and write it to parameter_sensitivity_auroc.csv. The article reports performance stays robust across wide k and N ranges, with peak performance for T between 100 and 300, while larger T reduces the number of predicted cell-containing droplets.

## Related tools

- **EmptyNN** (PU-learning cell-calling algorithm whose threshold T, cross-validation folds k, and iteration count N are swept via emptynn()) — https://github.com/lkmklsmn/empty_nn
- **keras** (R dependency backing the neural network trained within each emptynn() run of the sweep)
- **Matrix** (R dependency for handling the sparse transposed count matrices fed to emptynn())

## Examples

```
library(EmptyNN); counts <- t(Read10X_h5('./data/neurons_900_raw.h5', use.names=TRUE, unique.features=TRUE)); nn.res <- emptynn(counts, threshold=100, k=10, iteration=10, verbose=TRUE)
```

## Evaluation signals

- AUROC for default settings (T=100, k=10, N=10) matches the reported benchmarks of 94.73% (cell hashing) and 96.30% (genetic variation)
- AUROC varies only marginally across k in 5–20 and N in 5–20, confirming the paper's robustness claim
- Highest AUROC is observed for T values between 100 and 300
- Increasing T monotonically decreases the number of barcodes predicted as cell-containing droplets
- parameter_sensitivity_auroc.csv contains one aggregated AUROC and retained/removed barcode count per grid combination

## Limitations

- Parameter selection (k and T) is manual; EmptyNN implements no hyperparameter selection approach
- The low-total-count-implies-cell-free assumption may fail when genuine low-count cells exist, biasing results unless T is manually adjusted
- Benchmarking ground truth is limited to cell hashing or genetic variation labels; high overlap in classifications across methods can limit biological interpretation

## Evidence

- [supplementary] EmptyNN's performance remained very robust with respect to a large range of k (Figures S9B and S9F) and N (Figures S9C and S9G) values, with highest performance for T values between 100 and 300: [evidence span withheld — anchor status: not_found]
