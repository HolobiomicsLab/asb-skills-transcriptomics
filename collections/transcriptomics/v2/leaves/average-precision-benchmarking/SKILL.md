---
name: average-precision-benchmarking
description: Use when comparing alternative hierarchical correction algorithms for
  Cell Ontology-based classification and you need a threshold-free metric (average
  precision) plus per-cell-type PR curves to establish whether a correction degrades
  or preserves the baseline.
when_to_use_negative:
- Evaluating calibration or F1-score for binary yes/no cell-type decisions — the paper
  notes CellO's F1 performance lags average precision, so average precision alone
  understates the calibration gap
- Non-human or non-primary-cell RNA-seq data outside CellO's SRA-trained human primary
  bulk RNA-seq training scope
- Comparing against marker-gene-based flat classifiers rather than hierarchical correction
  schemes — that requires a different benchmarking design
tools:
- name: CellO
  role: Hierarchical classification framework providing the independent logistic regression
    classifiers and the default 'IR' (isotonic regression) correction algorithm whose
    outputs are scored by average precision
  repo: https://github.com/deweylab/CellO
- name: URSA
  role: Source of the Bayesian network correction (BNC) generative approach used as
    the degraded-performance comparison arm in the benchmark
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.isci.2020.101913
    title: 'CellO: comprehensive and hierarchical cell type classification of human
      cells with the Cell Ontology'
  source_content_hash: sha256:c7ea6d9b2b5104233693dfac125ff879e68bb0c63eaae64f2218c8afbee2dfce
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/deweylab/CellO
  repo_url: https://github.com/deweylab/CellO
  tools:
  - CellO
  - URSA
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# average-precision-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmarks hierarchical cell-type classification schemes (independent classifiers, isotonic regression/IR, True Path Rule/TPR, Bayesian network correction/BNC) by scoring per-cell-type probabilities with average precision and comparing precision-recall curves on a held-out validation partition of bulk RNA-seq data.

## When to use

Use when comparing alternative hierarchical correction algorithms for Cell Ontology-based classification and you need a threshold-free metric (average precision) plus per-cell-type PR curves to establish whether a correction degrades or preserves the baseline.

## When NOT to use

- Evaluating calibration or F1-score for binary yes/no cell-type decisions — the paper notes CellO's F1 performance lags average precision, so average precision alone understates the calibration gap
- Non-human or non-primary-cell RNA-seq data outside CellO's SRA-trained human primary bulk RNA-seq training scope
- Comparing against marker-gene-based flat classifiers rather than hierarchical correction schemes — that requires a different benchmarking design

## Inputs

- Pre-training/validation partitioned human primary bulk RNA-seq expression data (Zenodo 10.5281/zenodo.4289064)
- Cell type labels mapped to Cell Ontology terms
- Per-cell-type probability matrices from four schemes: independent classifiers, IR, TPR, BNC

## Outputs

- Per-cell-type average precision values CSV (figure3_average_precision.csv)
- Per-cell-type precision-recall curves figure overlaying the four schemes (figure3_pr_curves.png)

## How to apply

Partition the human primary bulk RNA-seq training data (Zenodo deposit 10.5281/zenodo.4289064, Cell Ontology-mapped labels) into pre-training and validation sets, then train independent per-term logistic regression classifiers on the pre-training partition within the CellO framework. Generate cell-type probabilities under four schemes: the independent-classifier baseline, isotonic regression (CellO's default 'IR' algorithm via the -a/--algo option), TPR propagation of probabilities to ontology ancestors, and BNC as implemented in URSA. Score each scheme on the validation partition using average precision per cell type, overlay per-cell-type precision-recall curves across all four schemes, and write the per-cell-type average precision values to a CSV. Validate by confirming IR and TPR average precision match the independent-classifier baseline within a small tolerance (no degradation), while BNC shows lower average precision, consistent with the paper's finding that discriminative methods outperformed the generative BNC approach.

## Related tools

- **CellO** (Hierarchical classification framework providing the independent logistic regression classifiers and the default 'IR' (isotonic regression) correction algorithm whose outputs are scored by average precision) — https://github.com/deweylab/CellO
- **URSA** (Source of the Bayesian network correction (BNC) generative approach used as the degraded-performance comparison arm in the benchmark)

## Examples

```
cello_predict -d TSV -u LOG1_TPM -s FULL_LENGTH -a IR -o validation_ir validation_partition.tsv
```

## Evaluation signals

- IR and TPR per-cell-type average precision values match the independent-classifier baseline within a small tolerance (no degradation expected)
- BNC average precision is lower than the independent-classifier baseline, reproducing the paper's discriminative-vs-generative finding
- Precision-recall curves for IR and TPR are visually similar to the independent-classifier curves on the validation partition
- Output files figure3_pr_curves.png and figure3_average_precision.csv exist and cover the same per-cell-type set

## Limitations

- BNC (URSA's generative Bayesian network correction) is not a drop-in improvement — it significantly degraded performance in the paper's validation comparison
- CellO's hierarchical algorithm option is limited to 'IR' (isotonic regression) and 'CLR' (cascaded logistic regression); TPR and BNC require external implementation
- Average precision masks calibration issues: the paper reports a gap between average-precision and F1-score performance, so AP-based benchmarking alone cannot certify binary-decision quality

## Evidence

- [discussion] Discriminative methods outperformed the generative BNC approach implemented in URSA: "we found that discriminative methods outperformed the generative BNC approach implemented in URSA (Lee et al., 2013)"
- [readme] Hierarchical classification algorithm choice is a CellO CLI parameter, with isotonic regression the default: "-a ALGO, --algo=ALGO  Hierarchical classification algorithm to apply (default='IR'). Must be one of: 'IR' - Isotonic regression, 'CLR' - cascaded logistic regression"
- [abstract] CellO achieves competitive performance relative to state-of-the-art methods, motivating AP-based benchmarking against alternatives: "CellO's comprehensive training set enables it to run out of the box on diverse cell types and achieves competitive or even superior performance when compared to existing state-of-the-art methods"
