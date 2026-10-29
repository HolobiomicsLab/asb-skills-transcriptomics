---
name: classifier-benchmark-metrics-computation
description: Use when a trained cell-type classifier (e.g., CellO IR or CLR models)
  and a labeled single-cell test set must be compared against a baseline by computing
  F1-score, precision, and average precision per Cell Ontology term.
when_to_use_negative:
- The goal is annotating new cells rather than benchmarking — use cello_predict directly
  instead of the metric-comparison workflow.
- The input is raw sequencing reads that have not been quantified or classified; the
  skill requires per-cell probability/binary decision tables plus ground-truth labels.
- Working with non-human species — CellO's models are trained on human primary cell
  bulk RNA-seq data from the SRA, and no curated mouse training set exists yet.
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: CellO
  role: Runs the hierarchical classifiers (IR via isotonic regression, CLR via cascaded
    logistic regression) producing per-cell probability and binary decision tables
    that are benchmarked with F1, precision, and average precision.
  repo: https://github.com/deweylab/CellO
- name: Scanpy
  role: Python library through which CellO's API interfaces for single-cell analysis
    pipelines, per the CellO tutorial notebook.
  repo: https://github.com/deweylab/CellO/blob/package_for_pypi/tutorial/cello_tutorial.ipynb
- name: cell-type-classification-paper
  role: Repository containing all code for performing the paper's experiments, including
    the Figure 4 benchmark reproducing median F1-scores.
  repo: https://github.com/deweylab/cell-type-classification-paper
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.isci.2020.101913
    title: 'CellO: comprehensive and hierarchical cell type classification of human
      cells with the Cell Ontology'
  source_content_hash: sha256:f5906159b6dd95bed6aae2a4973048c6c7b410883db4f6bfdfa20379541b4c3c
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
  - Scanpy
  - cell-type-classification-paper
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# classifier-benchmark-metrics-computation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark CellO's hierarchical classifiers (isotonic regression 'IR' and cascaded logistic regression 'CLR') against a Pearson-correlation 1NN baseline on a single-cell test subset by computing per-cell-type F1-score, precision, and average precision. Provides the quantitative evaluation protocol for comparing classifiers trained on SRA bulk RNA-seq data.

## When to use

Use when a trained cell-type classifier (e.g., CellO IR or CLR models) and a labeled single-cell test set must be compared against a baseline by computing F1-score, precision, and average precision per Cell Ontology term.

## When NOT to use

- The goal is annotating new cells rather than benchmarking — use cello_predict directly instead of the metric-comparison workflow.
- The input is raw sequencing reads that have not been quantified or classified; the skill requires per-cell probability/binary decision tables plus ground-truth labels.
- Working with non-human species — CellO's models are trained on human primary cell bulk RNA-seq data from the SRA, and no curated mouse training set exists yet.

## Inputs

- Processed single-cell test set: 4,936-cell non-droplet-based subset (Zenodo deposit 10.5281/zenodo.4289064)
- SRA bulk RNA-seq training set used for CellO model training and the 1NN baseline
- Cell Ontology term set over which classifiers are evaluated
- Per-cell classification probability tables (<output_prefix>.probability.tsv) and binary decision tables (<output_prefix>.binary.tsv) from cello_predict
- Ground-truth cell type labels for the test cells

## Outputs

- figure4_metrics.csv containing per-cell-type F1-score, precision, and average precision for each method (IR, CLR, 1NN)
- Figure 4-style comparison of metric distributions across cell types and methods
- Median F1-score summaries per method (reported medians: IR 0.81, CLR 0.85, 1NN 0.63)

## How to apply

Download the processed training and evaluation data from the Zenodo deposit (DOI 10.5281/zenodo.4289064) and extract the 4,936-cell non-droplet single-cell test subset along with the SRA bulk RNA-seq training set. Train CellO's per-term binary classifiers against the Cell Ontology using each hierarchical algorithm: isotonic regression ('IR', the default cello_predict algorithm) and cascaded logistic regression ('CLR', selected via the -a/--algo option). Run the Pearson-correlation 1NN baseline, assigning each test cell the type of the most correlation-similar bulk training sample. Classify all 4,936 test cells with each method, then compute per-cell-type F1-score, precision, and average precision over the evaluated Cell Ontology terms. Summarize metric distributions as Figure 4-style comparisons and write all per-cell-type values to figure4_metrics.csv; validation requires reproduced median F1 values to match the reported medians of 0.81 (IR), 0.85 (CLR), and 0.63 (1NN) within a small tolerance.

## Related tools

- **CellO** (Runs the hierarchical classifiers (IR via isotonic regression, CLR via cascaded logistic regression) producing per-cell probability and binary decision tables that are benchmarked with F1, precision, and average precision.) — https://github.com/deweylab/CellO
- **Scanpy** (Python library through which CellO's API interfaces for single-cell analysis pipelines, per the CellO tutorial notebook.) — https://github.com/deweylab/CellO/blob/package_for_pypi/tutorial/cello_tutorial.ipynb
- **cell-type-classification-paper** (Repository containing all code for performing the paper's experiments, including the Figure 4 benchmark reproducing median F1-scores.) — https://github.com/deweylab/cell-type-classification-paper

## Examples

```
cello_predict -a IR -d 10x -u COUNTS -s 3_PRIME example_input/Zheng_PBMC_10x -o test
```

## Evaluation signals

- Reproduced median per-cell-type F1 values match the reported medians of 0.81 (IR), 0.85 (CLR), and 0.63 (1NN baseline) within a small tolerance on the 4,936-cell non-droplet test subset.
- figure4_metrics.csv contains F1-score, precision, and average precision for every evaluated cell type for each of the three methods.
- Classifier probability outputs fall in [0, 1] and binary decisions align with CellO's hierarchical Cell Ontology structure.
- IR and CLR (discriminative hierarchical methods) outperform the 1NN baseline, mirroring the paper's result that IR/CLR medians exceed the 1NN median of 0.63.

## Limitations

- Classifier calibration gap: performance measured with F1-score lags performance measured with average precision; the authors note techniques are needed for closing this gap for binary yes-no decisions.
- Determining the optimal clustering resolution in scRNA-seq data is an open problem affecting CellO's cluster-based accuracy.
- Calibration of the classifiers is expected to improve only as more training data per cell type become available.

## Evidence

- [other] producing median F1-scores of 0.81, 0.85, and 0.63, respectively, across all cell types.: [evidence span withheld — anchor status: not_found]
