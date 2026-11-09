---
name: auroc-estimation-with-proc
description: Use when comparing droplet cell-calling predictions on a raw scRNA-seq
  count matrix against HTO-derived singlet/doublet/negative labels and you need a
  threshold-independent AUROC per method.
when_to_use_negative:
- Labels come from genetic demultiplexing (demuxlet on the multiplexed PBMC dataset
  SRR5398237) rather than HTO hashing — a parallel but distinct benchmarking pathway
- The raw count matrix is already filtered (e.g., Cell Ranger filtered_feature_bc_matrix),
  since AUROC must be computed over all 39,842 non-zero barcodes including low-count
  ones
- No ground-truth labels (singlet/doublet/negative) exist for the dataset, making
  ROC analysis uninterpretable
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_0203
tools:
- name: pROC
  role: R package used to compute the AUROC against HTO-derived labels
- name: EmptyNN
  role: Method under evaluation; produces cell-containing probabilities for barcodes
    in set U
  repo: https://github.com/lkmklsmn/empty_nn
- name: Cell Ranger v.2
  role: Baseline caller using total-count cutoff (threshold 200) to be scored in the
    same AUROC comparison
- name: Cell Ranger v.3
  role: Baseline caller with integrated EmptyDrops cell calling
- name: EmptyDrops
  role: Baseline caller using a Dirichlet-multinomial model to infer the cell-free
    profile
- name: CellBender
  role: Neural-network baseline caller; expected number of cells chosen from the barcode
    rank plot via total-droplets-included
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.patter.2021.100311
    title: 'EmptyNN: A neural network based on positive and unlabeled learning to
      remove cell-free droplets and recover lost cells in scRNA-seq data'
  source_content_hash: sha256:6b75e44e81a44a202300fc860b37e02ec6048dd994c074ce30ca29bb80a7d725
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
  - pROC
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8461538461538461
n_claims: 13
---

# auroc-estimation-with-proc

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes area under the receiver operating characteristic (AUROC) with the pROC procedure to benchmark cell-calling methods (EmptyNN vs Cell Ranger v.2, EmptyDrops, Cell Ranger v.3, CellBender) against ground-truth labels on the Stoeckius cell hashing dataset.

## When to use

Use when comparing droplet cell-calling predictions on a raw scRNA-seq count matrix against HTO-derived singlet/doublet/negative labels and you need a threshold-independent AUROC per method.

## When NOT to use

- Labels come from genetic demultiplexing (demuxlet on the multiplexed PBMC dataset SRR5398237) rather than HTO hashing — a parallel but distinct benchmarking pathway
- The raw count matrix is already filtered (e.g., Cell Ranger filtered_feature_bc_matrix), since AUROC must be computed over all 39,842 non-zero barcodes including low-count ones
- No ground-truth labels (singlet/doublet/negative) exist for the dataset, making ROC analysis uninterpretable

## Inputs

- Raw unfiltered count matrix of the Stoeckius cell hashing dataset (mtx or h5 format, loaded via Read10X_h5)
- HTO-derived ground-truth labels (singlet, doublet, negative) per barcode
- Per-method cell-calling predictions/probabilities (EmptyNN, Cell Ranger v.2, Cell Ranger v.3, EmptyDrops, CellBender)

## Outputs

- Per-method AUROC values (EmptyNN expected 94.73%; baselines 79.87%-90.16%)
- auroc_comparison.csv summarizing method ranking

## How to apply

First obtain ground-truth labels by integrating HTO (oligo-tag) counts from the cell hashing dataset, assigning each of the 39,842 barcodes a singlet, doublet, or negative call. For each method (EmptyNN run with threshold=100, k=10, iteration=10; Cell Ranger v.2 total-count cutoff of 200; Cell Ranger v.3; EmptyDrops; CellBender), use the predicted cell-containing probability or classification score as the ranking variable. Feed the binary ground truth (cell-containing vs cell-free) and the method scores to pROC's ROC/AUC computation per method. Write per-method AUROC values to auroc_comparison.csv and compare: EmptyNN should reach ~94.73% while baselines fall in the 79.87%-90.16% range, confirming EmptyNN's superiority.

## Related tools

- **pROC** (R package used to compute the AUROC against HTO-derived labels)
- **EmptyNN** (Method under evaluation; produces cell-containing probabilities for barcodes in set U) — https://github.com/lkmklsmn/empty_nn
- **Cell Ranger v.2** (Baseline caller using total-count cutoff (threshold 200) to be scored in the same AUROC comparison)
- **Cell Ranger v.3** (Baseline caller with integrated EmptyDrops cell calling)
- **EmptyDrops** (Baseline caller using a Dirichlet-multinomial model to infer the cell-free profile)
- **CellBender** (Neural-network baseline caller; expected number of cells chosen from the barcode rank plot via total-droplets-included)

## Examples

```
roc_obj <- pROC::roc(response = hto_labels, predictor = nn.res$probabilities); auc(roc_obj)
```

## Evaluation signals

- EmptyNN's computed AUROC on the cell hashing dataset is approximately 94.73%
- Each baseline method's AUROC falls within the reported 79.87%-90.16% range
- The computed ranking reproduces the paper's ordering, with EmptyNN superior to all baselines (Cell Ranger v.2, EmptyDrops, Cell Ranger v.3, CellBender)
- Every barcode receives a valid ground-truth label (singlet, doublet, or negative) from HTO counts before ROC computation
- auroc_comparison.csv contains exactly one AUROC value per evaluated method

## Limitations

- Ground truth is limited to cell hashing (or genetic variation) labels; high overlap in cell-calling classifications across methods may limit biological interpretation of method differences
- A method's binary keep/drop output must be converted to a score (e.g., EmptyNN's average prediction probability) to compute a meaningful ROC; pure cutoff callers are evaluated at a single operating point unless scores are available

## Evidence

- [abstract] achieved an area under the receiver operating characteristics of 94.73% and 96.30%, respectively: "achieved an area under the receiver operating characteristics of 94.73% and 96.30%, respectively"
- [results] HTO labels for benchmarking: "we integrated information derived from the HTO counts. As described above, the HTO counts provide singlet, doublet, and negative labels for each barcode."
