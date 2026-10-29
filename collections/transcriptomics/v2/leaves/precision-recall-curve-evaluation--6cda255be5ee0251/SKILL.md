---
name: precision-recall-curve-evaluation
description: Use when comparing discriminative vs. hierarchy-corrected cell-type classifiers
  (independent, IR, TPR, BNC) on labeled bulk or single-cell RNA-seq with Cell Ontology
  term labels, and average precision or PR curves are the scoring metrics.
when_to_use_negative:
- When the evaluation target is binary yes-no cell type decisions rather than ranking
  quality — the paper notes F1-score performance lags average precision and calibration
  techniques were still being developed
- When classifying non-human species — CellO's pretrained classifiers were trained
  on human primary bulk RNA-seq from the SRA, and no comprehensive curated mouse training
  set exists yet
- When inputs are single-cell data with unknown cluster resolution — the paper notes
  determining optimal clustering in scRNA-seq is an unsolved problem that affects
  cluster-based accuracy
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: CellO
  role: Trains independent per-term logistic regression classifiers against the Cell
    Ontology and applies hierarchical correction; the default 'IR' (isotonic regression)
    algorithm is one of the compared schemes and produces per-cell-type probability
    outputs
  repo: https://github.com/deweylab/CellO
- name: URSA
  role: Provides the Bayesian network correction (BNC) generative approach used as
    the comparison baseline that significantly degraded performance
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.isci.2020.101913
    title: 'CellO: comprehensive and hierarchical cell type classification of human
      cells with the Cell Ontology'
  source_content_hash: sha256:9e6506ab147c1f919d859bdff032115dd2ea3baf723fdc4499015a0979d2198d
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
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# precision-recall-curve-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluates cell-type classifiers from the CellO hierarchical framework by computing per-cell-type probabilities and scoring them with average precision, overlaying precision-recall curves to compare independent classifiers against hierarchy-correction schemes (IR, TPR, BNC).

## When to use

Use when comparing discriminative vs. hierarchy-corrected cell-type classifiers (independent, IR, TPR, BNC) on labeled bulk or single-cell RNA-seq with Cell Ontology term labels, and average precision or PR curves are the scoring metrics.

## When NOT to use

- When the evaluation target is binary yes-no cell type decisions rather than ranking quality — the paper notes F1-score performance lags average precision and calibration techniques were still being developed
- When classifying non-human species — CellO's pretrained classifiers were trained on human primary bulk RNA-seq from the SRA, and no comprehensive curated mouse training set exists yet
- When inputs are single-cell data with unknown cluster resolution — the paper notes determining optimal clustering in scRNA-seq is an unsolved problem that affects cluster-based accuracy

## Inputs

- Human primary bulk RNA-seq expression matrix (pre-training/validation partition, Zenodo deposit 10.5281/zenodo.4289064)
- Cell type labels mapped to Cell Ontology terms for each sample
- Cell Ontology graph structure (for TPR ancestor propagation)
- Trained CellO per-term logistic regression classifiers (pretrained .model.dill or retrained)

## Outputs

- figure3_pr_curves.png — per-cell-type precision-recall curves overlaying the four classification schemes (independent, IR, TPR, BNC)
- figure3_average_precision.csv — per-cell-type average precision values for each of the four schemes
- Per-cell-type probability tables (e.g., <output_prefix>.probability.tsv with N cells x M cell types)

## How to apply

Partition the human primary bulk RNA-seq data (Zenodo deposit 10.5281/zenodo.4289064) into pre-training and validation sets, mapping cell type labels to Cell Ontology terms. Train independent per-term logistic regression classifiers on the pre-training partition using CellO's framework, then apply the hierarchical correction schemes: isotonic regression (CellO's default 'IR' algorithm, selectable via the -a/--algo option), True Path Rule (TPR) propagation to ontology ancestors, and the Bayesian network correction (BNC) implemented in URSA. Compute per-cell-type probabilities on the validation partition for all four schemes, score each with average precision, and plot per-cell-type precision-recall curves overlaying the schemes. Validate by confirming that IR and TPR average precision values match the independent-classifier baseline within a small tolerance (no degradation), while BNC shows lower average precision, consistent with the paper's finding that discriminative methods outperformed the generative BNC approach.

## Related tools

- **CellO** (Trains independent per-term logistic regression classifiers against the Cell Ontology and applies hierarchical correction; the default 'IR' (isotonic regression) algorithm is one of the compared schemes and produces per-cell-type probability outputs) — https://github.com/deweylab/CellO
- **URSA** (Provides the Bayesian network correction (BNC) generative approach used as the comparison baseline that significantly degraded performance)

## Examples

```
cello_predict -a IR -d 10x -u COUNTS -s 3_PRIME example_input/Zheng_PBMC_10x -o test
```

## Evaluation signals

- IR and TPR average precision values match the independent-classifier baseline within a small tolerance (no degradation on the validation set)
- BNC scheme shows visibly lower average precision and inferior precision-recall curves relative to the discriminative schemes
- Precision-recall curves for IR and TPR resemble the independent classifiers' curves per cell type
- Output probability table is an N cells x M cell types matrix where element (i,j) is CellO's confidence that cell i is of cell type j
- Cell Ontology term IDs in outputs can be verified when running CellO with the -b/--ontology_term_ids option

## Limitations

- The generative BNC correction significantly degrades classification performance relative to independent classifiers, IR, and TPR
- Calibration gap: CellO's F1-score performance lags its average precision, so binary yes-no decisions are less reliable than ranking
- Independent classifiers can be logically inconsistent for cell types sharing a parent via 'is a' but with divergent evolutionary origins
- Classifier calibration is expected to improve only as more training data per cell type become available

## Evidence
