---
name: ontology-graph-classifier-training
description: Use when annotating human scRNA-seq or bulk RNA-seq cells against the
  Cell Ontology hierarchy, or when benchmarking hierarchical classifiers (IR, CLR)
  against flat baselines such as a Pearson-correlation 1NN.
when_to_use_negative:
- Input RNA-seq data is from non-human species (e.g., mouse) — CellO's pre-trained
  models were trained only on human primary samples from the SRA
- The input gene set does not match the classifier's training genes and retraining
  on a single small dataset would be unreliable
- The task requires lineage-based or evolutionary relationships between cell types
  that the Cell Ontology does not encode
edam_operation: http://edamontology.org/operation_1777
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0625
tools:
- name: CellO
  role: Python package that trains and applies the per-term IR and CLR hierarchical
    classifiers against the Cell Ontology
  repo: https://github.com/deweylab/CellO
- name: Scanpy
  role: Single-cell library with which CellO's Python API integrates in the analysis
    pipeline
  repo: https://github.com/deweylab/CellO/blob/package_for_pypi/tutorial/cello_tutorial.ipynb
- name: cell-type-classification-paper
  role: Repository with all code for performing the paper's experiments, including
    benchmark reproductions
  repo: https://github.com/deweylab/cell-type-classification-paper
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.isci.2020.101913
    title: 'CellO: comprehensive and hierarchical cell type classification of human
      cells with the Cell Ontology'
  source_content_hash: sha256:e8154c784f1c13c14458991c1e1acc82b46ee44a7f4f2636e3e8074a2bd62f22
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
  edam_operation: http://edamontology.org/operation_1777
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# ontology-graph-classifier-training

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Train per-term binary linear classifiers (isotonic regression 'IR' or cascaded logistic regression 'CLR') against the graph-structured Cell Ontology, using bulk RNA-seq training data, to hierarchically annotate single-cell RNA-seq data.

## When to use

Use when annotating human scRNA-seq or bulk RNA-seq cells against the Cell Ontology hierarchy, or when benchmarking hierarchical classifiers (IR, CLR) against flat baselines such as a Pearson-correlation 1NN.

## When NOT to use

- Input RNA-seq data is from non-human species (e.g., mouse) — CellO's pre-trained models were trained only on human primary samples from the SRA
- The input gene set does not match the classifier's training genes and retraining on a single small dataset would be unreliable
- The task requires lineage-based or evolutionary relationships between cell types that the Cell Ontology does not encode

## Inputs

- Processed single-cell RNA-seq expression matrix (TSV, CSV, HDF5, or 10x-formatted directory with matrix.mtx, genes.tsv, barcodes.tsv)
- SRA human primary-cell bulk RNA-seq training set (from Zenodo deposit 10.5281/zenodo.4289064)
- Cell Ontology graph structure (obofoundry cl.owl)
- Optional pre-computed cluster assignments (TSV of cell IDs and cluster integers)

## Outputs

- <output_prefix>.probability.tsv — NxM per-cell classification probability table against Cell Ontology terms
- <output_prefix>.binary.tsv — NxM binary-decision matrix per cell type
- <output_prefix>.most_specific.tsv — per-cell most-specific predicted cell type
- Per-cell-type F1-score, precision, and average precision metrics (e.g., figure4_metrics.csv)

## How to apply

Download the processed training/evaluation data (Zenodo DOI 10.5281/zenodo.4289064) and the SRA bulk RNA-seq training set. Train a binary classifier for each Cell Ontology term using either isotonic regression ('IR') or cascaded logistic regression ('CLR'), exploiting the ontology's 'is a' relationships so predictions remain hierarchy-consistent. Classify cells (e.g., a 4,936-cell non-droplet test subset), producing per-cell probability and binary decision tables (<output_prefix>.probability.tsv and <output_prefix>.binary.tsv). As a baseline, run a Pearson-correlation 1NN classifier that assigns each test cell the type of the most correlation-similar bulk training sample. Evaluate per-cell-type F1-score, precision, and average precision; reported medians on the non-droplet test set are 0.81 (IR), 0.85 (CLR), and 0.63 (1NN).

## Related tools

- **CellO** (Python package that trains and applies the per-term IR and CLR hierarchical classifiers against the Cell Ontology) — https://github.com/deweylab/CellO
- **Scanpy** (Single-cell library with which CellO's Python API integrates in the analysis pipeline) — https://github.com/deweylab/CellO/blob/package_for_pypi/tutorial/cello_tutorial.ipynb
- **cell-type-classification-paper** (Repository with all code for performing the paper's experiments, including benchmark reproductions) — https://github.com/deweylab/cell-type-classification-paper

## Examples

```
cello_predict -a IR -d 10x -u COUNTS -s 3_PRIME example_input/Zheng_PBMC_10x -o test
```

## Evaluation signals

- Reproduced median per-cell-type F1 values match the reported medians of 0.81 (IR), 0.85 (CLR), and 0.63 (1NN) within a small tolerance on the 4,936-cell non-droplet test subset
- Output CSV contains F1-score, precision, and average precision for every evaluated Cell Ontology term
- Binary decision matrix is logically consistent with the ontology hierarchy (no cell predicted as a descendant without its ancestor)
- Probability and binary TSVs have N rows equal to the number of input cells and M columns equal to the evaluated ontology terms
- Average precision substantially exceeds F1-score, consistent with the paper's noted calibration gap

## Limitations

- Classifier calibration is imperfect: performance with F1-score lags average precision, and calibration is expected to improve only as more training data per cell type become available
- Independent per-term classifiers can be inconsistent when cell types share a parent via 'is a' but have divergent evolutionary origins
- Accuracy depends on clustering resolution, and determining optimal clustering in scRNA-seq data is an unsolved problem
- Cell types known to be resident to input tissues can be filtered via Uberon term IDs, but confounding by sorting/preparation procedures (e.g., FACS) on gene expression vs. cell type remains unclear

## Evidence

- [results] Reproduced benchmark medians: [evidence span withheld — anchor status: not_found]
