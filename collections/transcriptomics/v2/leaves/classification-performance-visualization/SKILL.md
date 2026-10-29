---
name: classification-performance-visualization
description: Use when you have trained classifier outputs (per-cell probabilities
  and binary decisions) plus ground-truth cell-type labels and need to compare classification
  algorithms via per-type F1, precision, and average precision.
when_to_use_negative:
- Input is non-human data (e.g., mouse) — CellO's training set covers human primary
  SRA samples only
- You need cell annotations for new samples rather than a benchmark/comparison of
  classification algorithms against gold-standard labels
- Input is raw reads that still need quantification — first quantify with Kallisto
  via cello_quantify_sample and provide LOG1_TPM units
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: CellO
  role: Trains per-term binary classifiers against the Cell Ontology with the 'IR'
    (isotonic regression) and 'CLR' (cascaded logistic regression) algorithms via
    cello_predict, emitting probability and binary decision tables used to compute
    metrics
  repo: https://github.com/deweylab/CellO
- name: Scanpy
  role: Python library whose API CellO interfaces with for single-cell preprocessing
    and classification within the analysis pipeline
  repo: https://github.com/deweylab/CellO
- name: cell-type-classification-paper
  role: Repository holding all code for performing the paper's experiments, including
    the Figure 4 benchmark reproduction
  repo: https://github.com/deweylab/cell-type-classification-paper
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.isci.2020.101913
    title: 'CellO: comprehensive and hierarchical cell type classification of human
      cells with the Cell Ontology'
  source_content_hash: sha256:7ebbd0a594ce5435133ba4851268440bde55d458d698a5a91b61c788b20c0846
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
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# classification-performance-visualization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark CellO's hierarchical classifiers (IR and CLR) against a Pearson-correlation 1NN baseline on a single-cell test set by computing per-cell-type F1-score, precision, and average precision, then summarizing the metric distributions into a Figure 4-style comparison (figure4_metrics.csv).

## When to use

Use when you have trained classifier outputs (per-cell probabilities and binary decisions) plus ground-truth cell-type labels and need to compare classification algorithms via per-type F1, precision, and average precision.

## When NOT to use

- Input is non-human data (e.g., mouse) — CellO's training set covers human primary SRA samples only
- You need cell annotations for new samples rather than a benchmark/comparison of classification algorithms against gold-standard labels
- Input is raw reads that still need quantification — first quantify with Kallisto via cello_quantify_sample and provide LOG1_TPM units

## Inputs

- Processed single-cell test set: 4,936-cell non-droplet-based subset with ground-truth cell-type labels (from Zenodo DOI 10.5281/zenodo.4289064)
- SRA bulk RNA-seq training set of human primary cells (for training CellO models and for the 1NN Pearson-correlation baseline)
- Cell Ontology term set over which classifiers are trained and evaluated

## Outputs

- Per-cell-type F1-score, precision, and average precision for IR, CLR, and 1NN (figure4_metrics.csv)
- Per-cell probability and binary decision tables for each method (<output_prefix>.probability.tsv, <output_prefix>.binary.tsv)
- Figure 4-style metric distribution comparison across methods

## How to apply

Download the processed training and evaluation data from the Zenodo deposit (DOI 10.5281/zenodo.4289064) and extract the 4,936-cell non-droplet single-cell test subset together with the SRA bulk RNA-seq training set. Train CellO's per-term binary classifiers against the Cell Ontology using the two hierarchical algorithms: isotonic regression ('IR', the default '-a' value) and cascaded logistic regression ('CLR'). Run the 1NN baseline by computing Pearson correlation between each test cell and bulk training samples, assigning each test cell the type of the most correlation-similar sample. Classify all 4,936 test cells with each method, producing per-cell probability tables (<output_prefix>.probability.tsv) and binary decision tables (<output_prefix>.binary.tsv). Compute per-cell-type F1-score, precision, and average precision for each method over the evaluated Cell Ontology terms, summarize the metric distributions as Figure 4-style comparisons, and write all per-cell-type values to figure4_metrics.csv.

## Related tools

- **CellO** (Trains per-term binary classifiers against the Cell Ontology with the 'IR' (isotonic regression) and 'CLR' (cascaded logistic regression) algorithms via cello_predict, emitting probability and binary decision tables used to compute metrics) — https://github.com/deweylab/CellO
- **Scanpy** (Python library whose API CellO interfaces with for single-cell preprocessing and classification within the analysis pipeline) — https://github.com/deweylab/CellO
- **cell-type-classification-paper** (Repository holding all code for performing the paper's experiments, including the Figure 4 benchmark reproduction) — https://github.com/deweylab/cell-type-classification-paper

## Examples

```
cello_predict -a CLR -d TSV -u LOG1_TPM -s FULL_LENGTH expression.tsv -o cello_out
```

## Evaluation signals

- Reproduced median per-cell-type F1 values match the reported medians of 0.81 (IR), 0.85 (CLR), and 0.63 (1NN baseline) within a small tolerance
- figure4_metrics.csv contains F1-score, precision, and average precision rows for every evaluated Cell Ontology term and all three methods
- IR and CLR medians exceed the 1NN baseline median F1, consistent with the reported ordering
- CellO outputs are well-formed: probability table is NxM (N cells x M cell types) and binary table entries are 0/1 for each cell-type pair

## Limitations

- Calibration gap: CellO's F1-score performance lags its average precision, so binary yes-no decisions are weaker than ranking performance; classifiers are expected to improve only as more per-type training data become available
- CellO is trained on human primary cell bulk RNA-seq from the SRA; comprehensive training sets for other species (e.g., mouse) do not yet exist
- Optimal clustering resolution for scRNA-seq annotation is an unsolved problem that affects cluster-based accuracy

## Evidence
