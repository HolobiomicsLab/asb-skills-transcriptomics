---
name: cross-method-benchmark-evaluation
description: Use when comparing a new cell-type annotation tool against ready-to-run
  baselines on a shared single-cell test set, especially when tools use different
  reference sets and label vocabularies that must be reconciled via an ontology.
when_to_use_negative:
- When the goal is to evaluate a single method's absolute accuracy rather than a controlled
  head-to-head comparison across methods with heterogeneous reference sets
- When the input data are droplet-based single-cell test sets, which fall outside
  the non-droplet benchmark scope of this comparison
edam_operation: http://edamontology.org/operation_3802
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: CellO
  role: Hierarchical annotation method under evaluation; run with algo='IR' and its
    pre-trained model to produce per-cell classification probabilities against the
    Cell Ontology
  repo: https://github.com/deweylab/CellO
- name: scMatch
  role: Baseline ready-to-run annotation method evaluated with the FANTOM5 reference;
    flat labels projected onto Cell Ontology term IDs
- name: SingleR
  role: Baseline ready-to-run annotation method evaluated with Blueprint+ENCODE and
    HPCA references; flat labels projected onto Cell Ontology term IDs
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.isci.2020.101913
    title: 'CellO: comprehensive and hierarchical cell type classification of human
      cells with the Cell Ontology'
  source_content_hash: sha256:b0cb57bd610ab355b07c5a98fe7bd1adf786de75284d288ff1e29571f6b25b4a
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
  - SingleR
  - scMatch
  edam_operation: http://edamontology.org/operation_3802
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# cross-method-benchmark-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A fair-comparison benchmarking procedure for cell type annotation methods: classify the same test set with multiple prepackaged tools (CellO, scMatch, SingleR), project flat labels onto the Cell Ontology, and score each method only over cell types present in its own training set using per-cell-type F1-score and precision.

## When to use

Use when comparing a new cell-type annotation tool against ready-to-run baselines on a shared single-cell test set, especially when tools use different reference sets and label vocabularies that must be reconciled via an ontology.

## When NOT to use

- When the goal is to evaluate a single method's absolute accuracy rather than a controlled head-to-head comparison across methods with heterogeneous reference sets
- When the input data are droplet-based single-cell test sets, which fall outside the non-droplet benchmark scope of this comparison

## Inputs

- Non-droplet single-cell RNA-seq test set from the Zenodo deposit 10.5281/zenodo.4289064
- CellO pre-trained model with hierarchical isotonic regression classifiers
- scMatch with FANTOM5 reference set
- SingleR with Blueprint+ENCODE and HPCA reference sets
- Ontology-grounded ground-truth cell type labels for the test cells

## Outputs

- Per-method per-cell-type F1-score and precision metrics table (TSV)
- Figure 5-style summary plots of F1-score and precision distributions across cell types for CellO, scMatch, and SingleR
- Ontology-projected cell-type labels for scMatch and SingleR (Cell Ontology term IDs)

## How to apply

Download the benchmark data package from the Zenodo deposit (10.5281/zenodo.4289064) containing the non-droplet single-cell test set. Classify the test set with CellO using the default hierarchical isotonic regression algorithm (algo='IR') and its pre-trained model, capturing the per-cell classification probabilities from the .probability.tsv output. Classify the same test set with scMatch using the FANTOM5 reference and with SingleR using the Blueprint+ENCODE and HPCA references. Project the flat scMatch and SingleR cell-type labels onto Cell Ontology term IDs so all three methods are scored against the same ontology-grounded ground truth. Critically, for each method restrict evaluation to cell types present in that method's own training set — this avoids penalizing methods for cell types they were never designed to recognize — then compute per-cell-type F1-score and precision and render the distributions as Figure 5-style summary plots.

## Related tools

- **CellO** (Hierarchical annotation method under evaluation; run with algo='IR' and its pre-trained model to produce per-cell classification probabilities against the Cell Ontology) — https://github.com/deweylab/CellO
- **scMatch** (Baseline ready-to-run annotation method evaluated with the FANTOM5 reference; flat labels projected onto Cell Ontology term IDs)
- **SingleR** (Baseline ready-to-run annotation method evaluated with Blueprint+ENCODE and HPCA references; flat labels projected onto Cell Ontology term IDs)

## Examples

```
cello_predict -d TSV -u LOG1_TPM -s FULL_LENGTH -a IR -o cello_output non_droplet_test_set.tsv
```

## Evaluation signals

- The per-cell-type metrics TSV is populated for all three methods (CellO, scMatch, SingleR) over their respective restricted cell-type sets
- F1-score and precision distributions qualitatively match Figure 5B of the paper: CellO competitive with or superior to scMatch and SingleR on the non-droplet test set
- scMatch and SingleR labels are successfully projected to valid Cell Ontology term IDs shared with the ground-truth annotation
- Each method's evaluation set contains only cell types present in that method's own prepackaged reference/training set, ensuring no method is scored on unfamiliar cell types

## Limitations

- Fair comparison requires restricting evaluation to each method's own training cell types, so metrics are not computed on a single common cell-type universe across methods
- CellO's F1-score lags its average precision; calibration for binary yes-no decisions needs improvement, which affects thresholded F1 comparisons
- Results depend on the quality of the projection of flat scMatch/SingleR labels onto Cell Ontology term IDs

## Evidence

- [results] CellO outperformed the existing ready-to-run methods scMatch and SingleR on the non-droplet-based single-cell test set (Figure 5B): [evidence span withheld — anchor status: not_found]
