---
name: ontology-term-projection
description: Use when comparing a hierarchical ontology-based classifier against flat
  annotation tools whose outputs are free-text cell-type labels, and you need a shared
  Cell Ontology term-ID space for hierarchical F1/precision scoring.
when_to_use_negative:
- When all methods being compared already output Cell Ontology term IDs natively (no
  projection step is needed)
- When the analysis goal is flat, non-hierarchical annotation accuracy without an
  ontology-grounded ground truth
- When the input is non-human RNA-seq data (CellO is trained only on human primary
  cell bulk RNA-seq from the SRA)
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CellO
  role: Hierarchical classifier whose labels are natively grounded in Cell Ontology
    term IDs; used with algo='IR' (isotonic regression) to produce the reference ontology-space
    predictions
  repo: https://github.com/deweylab/CellO
- name: CellO Viewer
  role: Web application for exploring CellO's models and cell type expression signals
    across the Cell Ontology graph
  repo: https://github.com/uwgraphics/CellOViewer
- name: cell-type-classification-paper
  role: Repository holding all code for reproducing the paper's experiments, including
    the scMatch/SingleR benchmark against CellO
  repo: https://github.com/deweylab/cell-type-classification-paper
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.isci.2020.101913
    title: 'CellO: comprehensive and hierarchical cell type classification of human
      cells with the Cell Ontology'
  source_content_hash: sha256:8f6f1b1b008312829bdce41ea9f205d353d13c29aed869d4d43f23e324f23f05
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
  - cell-type-classification-paper
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# ontology-term-projection

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Project flat cell-type annotation labels (e.g., from scMatch or SingleR) onto Cell Ontology term IDs so that ontology-based classifiers like CellO and flat ready-to-run methods can be evaluated against a common ontology-grounded ground truth using per-cell-type F1-score and precision.

## When to use

Use when comparing a hierarchical ontology-based classifier against flat annotation tools whose outputs are free-text cell-type labels, and you need a shared Cell Ontology term-ID space for hierarchical F1/precision scoring.

## When NOT to use

- When all methods being compared already output Cell Ontology term IDs natively (no projection step is needed)
- When the analysis goal is flat, non-hierarchical annotation accuracy without an ontology-grounded ground truth
- When the input is non-human RNA-seq data (CellO is trained only on human primary cell bulk RNA-seq from the SRA)

## Inputs

- Non-droplet single-cell RNA-seq test set from the Zenodo deposit 10.5281/zenodo.4289064
- CellO per-cell classification probability table (<output_prefix>.probability.tsv)
- scMatch output labels (FANTOM5 reference)
- SingleR output labels (Blueprint+ENCODE and HPCA references)
- Cell Ontology term-ID mappings for the ground-truth and predicted labels

## Outputs

- Per-cell-type F1-score and precision TSV table populated for CellO, scMatch, and SingleR
- Figure 5-style summary plots of F1-score and precision distributions across cell types
- Ontology-projected label sets in Cell Ontology term-ID space (e.g., via cello_predict -b output)

## How to apply

Download the benchmark data package from the Zenodo deposit (10.5281/zenodo.4289064), which contains the non-droplet single-cell test set. Run CellO with the hierarchical isotonic regression algorithm (algo='IR', the default) to obtain per-cell classification probabilities, then run scMatch with the FANTOM5 reference and SingleR with the Blueprint+ENCODE and HPCA references on the same test set. Map the flat scMatch and SingleR labels onto Cell Ontology term IDs so all three methods are scored against the same ontology-grounded ground truth. For each method, restrict evaluation to cell types present in that method's own prepackaged reference/training set, compute per-cell-type F1-score and precision, save the metrics as a TSV table, and render F1 and precision distributions across cell types as the Figure 5-style summary. The ontology projection is the key decision point: without mapping flat labels into the Cell Ontology term-ID space, hierarchical evaluation and method comparison are invalid.

## Related tools

- **CellO** (Hierarchical classifier whose labels are natively grounded in Cell Ontology term IDs; used with algo='IR' (isotonic regression) to produce the reference ontology-space predictions) — https://github.com/deweylab/CellO
- **CellO Viewer** (Web application for exploring CellO's models and cell type expression signals across the Cell Ontology graph) — https://github.com/uwgraphics/CellOViewer
- **cell-type-classification-paper** (Repository holding all code for reproducing the paper's experiments, including the scMatch/SingleR benchmark against CellO) — https://github.com/deweylab/cell-type-classification-paper

## Examples

```
cello_predict -d 10x -u COUNTS -s 3_PRIME -a IR -b example_input/Zheng_PBMC_10x -o test
```

## Evaluation signals

- The metric table is populated with per-cell-type F1-scores and precision for all three methods (CellO, scMatch, SingleR)
- The F1 and precision distributions qualitatively match Figure 5B of the paper: CellO outperforms scMatch and SingleR when evaluation is restricted to each method's own reference cell types
- All projected labels resolve to valid Cell Ontology term IDs (verifiable by running CellO with the -b/--ontology_term_ids flag for rigorous ID-based output)
- Evaluation is restricted per-method to cell types present in that method's training/reference set, avoiding penalizing tools for missing cell types
- The same ontology-grounded ground truth is used to score all three methods, ensuring the comparison is apples-to-apples

## Limitations

- CellO's training data cover human primary cell bulk RNA-seq from the SRA; no curated mouse training set exists yet, and mouse metadata standardization in MetaSRA is still pending
- CellO's F1-score performance lags its average-precision performance, so calibration for binary yes/no decisions is imperfect
- Evaluation restricted to each method's own reference cell types means comparisons are conditional on the available cell-type sets, and the optimal clustering resolution for scRNA-seq annotation remains an open problem

## Evidence
