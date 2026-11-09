---
name: hierarchical-cell-type-classification
description: Use when annotating human single-cell or bulk RNA-seq cells against the
  Cell Ontology, especially when flat classifiers would be inconsistent with the ontology
  hierarchy or when cells should be placed at an internal (ancestral) ontology node
  rather than a leaf.
when_to_use_negative:
- 'Input is non-human RNA-seq (e.g., mouse): CellO is pre-trained only on human primary
  bulk RNA-seq from the SRA and no curated mouse training set exists yet'
- 'Raw FASTQ reads that have not been quantified: quantify first with cello_quantify_sample/Kallisto
  before classification'
- The task requires flat classification with user-supplied marker genes or a user-provided
  labeled training set rather than ontology-based annotation
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CellO
  role: Trains per-ontology-term binary classifiers and applies hierarchical correction
    (IR or CLR) via cello_predict to annotate cells against the Cell Ontology
  repo: https://github.com/deweylab/CellO
- name: CellO Viewer
  role: Web application for exploring gene-wise model coefficients (cell type expression
    signatures) across the Cell Ontology
  repo: https://github.com/uwgraphics/CellOViewer
- name: Scanpy
  role: Python library CellO integrates with (Leiden clustering, Scanpy API tutorial)
    for single-cell pipelines
  repo: https://github.com/deweylab/CellO
- name: URSA
  role: Bayesian network correction (BNC) baseline used to compare hierarchy-correction
    schemes against CellO's discriminative IR/TPR approaches
provenance:
  source_task_ids:
  - task_003
  - task_004
  - task_008
  - task_007
  - task_006
  - task_002
  source_papers:
  - doi: 10.1016/j.isci.2020.101913
    title: 'CellO: comprehensive and hierarchical cell type classification of human
      cells with the Cell Ontology'
  source_content_hash: sha256:933e6bba34f6283e09c89d9aae459a94f2e337b10bbef640f4f1e8c29a430db3
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
  - CellO Viewer
  - Scanpy
  - URSA
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 15
---

# hierarchical-cell-type-classification

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Hierarchical cell type classification of human RNA-seq data against the graph-structured Cell Ontology using CellO, which trains per-term binary classifiers on bulk RNA-seq from the SRA and applies hierarchy-aware correction (isotonic regression or cascaded logistic regression) to produce logically consistent, ontology-placed annotations.

## When to use

Use when annotating human single-cell or bulk RNA-seq cells against the Cell Ontology, especially when flat classifiers would be inconsistent with the ontology hierarchy or when cells should be placed at an internal (ancestral) ontology node rather than a leaf.

## When NOT to use

- Input is non-human RNA-seq (e.g., mouse): CellO is pre-trained only on human primary bulk RNA-seq from the SRA and no curated mouse training set exists yet
- Raw FASTQ reads that have not been quantified: quantify first with cello_quantify_sample/Kallisto before classification
- The task requires flat classification with user-supplied marker genes or a user-provided labeled training set rather than ontology-based annotation

## Inputs

- Human gene expression matrix in 10x format (matrix.mtx, genes.tsv, barcodes.tsv), TSV, CSV, or HDF5
- Expression units: COUNTS, CPM, LOG1_CPM, TPM, or LOG1_TPM
- Assay type: 3_PRIME or FULL_LENGTH
- Optional pre-clustering TSV mapping cell IDs to cluster integers (for -p)
- Optional pretrained model file (.dill) via -m

## Outputs

- <output_prefix>.probability.tsv — N x M per-cell, per-ontology-term probability table
- <output_prefix>.binary.tsv — N x M binary decision matrix
- <output_prefix>.most_specific.tsv — most-specific predicted cell type per cell
- <output_prefix>.log/ directory with execution logs, genes_absent_from_training_set.tsv, and clustering.tsv
- Per-cell-type F1-score, precision, and average precision when benchmarked against reference labels

## How to apply

Run cello_predict on a gene expression matrix (10x directory, TSV, CSV, or HDF5), specifying the data type (-d), expression units (-u: COUNTS, CPM, LOG1_CPM, TPM, LOG1_TPM), and assay type (-s: 3_PRIME or FULL_LENGTH); the hierarchical algorithm is chosen with -a, defaulting to isotonic regression ('IR') with cascaded logistic regression ('CLR') as the alternative. CellO clusters the input cells (Leiden by default) unless a pre-clustering TSV is supplied via -p; if the input gene set does not match the pre-trained classifier's genes, use -t to retrain a model on the present genes, saved as <output_prefix>.model.dill. The tool writes three tables: <output_prefix>.probability.tsv (per-cell per-ontology-term probabilities), <output_prefix>.binary.tsv (binary decisions), and <output_prefix>.most_specific.tsv (most-specific predicted type per cell). Evaluate per-cell-type F1-score, precision, and average precision against reference labels; on the non-droplet single-cell test set, expect median F1s near 0.81 (IR) and 0.85 (CLR), and on FAC-sorted Zheng PBMCs near 0.97 (IR) and 0.96 (CLR).

## Related tools

- **CellO** (Trains per-ontology-term binary classifiers and applies hierarchical correction (IR or CLR) via cello_predict to annotate cells against the Cell Ontology) — https://github.com/deweylab/CellO
- **CellO Viewer** (Web application for exploring gene-wise model coefficients (cell type expression signatures) across the Cell Ontology) — https://github.com/uwgraphics/CellOViewer
- **Scanpy** (Python library CellO integrates with (Leiden clustering, Scanpy API tutorial) for single-cell pipelines) — https://github.com/deweylab/CellO
- **URSA** (Bayesian network correction (BNC) baseline used to compare hierarchy-correction schemes against CellO's discriminative IR/TPR approaches)

## Examples

```
cello_predict -d 10x -u COUNTS -s 3_PRIME example_input/Zheng_PBMC_10x -o test
```

## Evaluation signals

- All three output tables (<prefix>.probability.tsv, .binary.tsv, .most_specific.tsv) are produced and the probability table is a valid N x M matrix over Cell Ontology terms
- On the 4,936-cell non-droplet single-cell test subset, reproduced median per-cell-type F1-scores match ~0.81 (IR), 0.85 (CLR), and 0.63 (1NN) within tolerance
- On the FAC-sorted Zheng PBMC 10x dataset, median F1-scores reproduce ~0.97 (IR), 0.96 (CLR), and 0.95 (1NN) with IR/CLR outperforming 1NN
- Hierarchy correction with IR and TPR yields average precision similar to independent classifiers (no degradation), whereas BNC shows lower average precision
- F1-scores are stable (small variance) across Leiden clustering resolutions (e.g., five settings on Zheng PBMC), confirming robustness to clustering granularity

## Limitations

- F1-score performance lags average precision; classifier calibration for binary yes-no decisions is imperfect and expected to improve only as more training data per cell type become available
- The input gene set must match the pre-trained classifier's genes, otherwise the model must be retrained with -t, and absent genes are logged in genes_absent_from_training_set.tsv
- Determining optimal Leiden clustering resolution is an open problem that affects cluster-based annotation accuracy
- Training set covers only human, healthy, untreated primary bulk RNA-seq samples from the SRA; cell types absent from training may only be assignable to ancestral ontology terms (e.g., acinar cells as glandular epithelial cells)

## Evidence
