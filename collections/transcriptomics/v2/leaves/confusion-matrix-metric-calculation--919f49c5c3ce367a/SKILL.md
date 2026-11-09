---
name: confusion-matrix-metric-calculation
description: Use when an inferred GRN must be quantitatively scored against an undirected
  typed gold standard (e.g., DREAM 4) using edge-level confusion-matrix metrics.
when_to_use_negative:
- The network to evaluate is a directed, signed Boolean model rather than an undirected
  edge list compared against an undirected typed gold standard — the required simplification
  (dropping interaction types, making edges positive) is specific to edge-level GRN
  scoring.
- The goal is ranking metrics such as AUROC or AUPR on a scored edge list rather than
  thresholded confusion-matrix proportions (sensitivity/specificity/accuracy).
- No gold standard network exists for the organism or dataset, so true positives and
  true negatives cannot be defined.
edam_operation: http://edamontology.org/operation_3799
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0602
tools:
- name: Augusta
  role: Produces the inferred initial GRNs (via mutual information with MSD edge orientation)
    that are scored against the gold standard using confusion-matrix metrics
  repo: https://github.com/JanaMus/Augusta
- name: Python
  role: Implements the confusion-matrix computation and writes sensitivity/specificity/accuracy
    rows to the metrics CSV
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.csbj.2024.01.013
    title: 'Augusta: From RNA‐Seq to gene regulatory networks and Boolean models'
  source_content_hash: sha256:bd22e891fceeae4d58c6aaf5355bace4dea4b4fc843b8f3a5b887017b57d5fec
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/WanluLiuLab/CRISPRTE
  repo_url: https://github.com/JanaMus/Augusta
  tools:
  - Augusta
  - Python
  edam_operation: http://edamontology.org/operation_3799
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.75
n_claims: 4
---

# confusion-matrix-metric-calculation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Evaluates inferred gene regulatory networks by building a confusion matrix of predicted versus true (gold standard) edges and deriving sensitivity, specificity, and accuracy, as done for Augusta's DREAM 4 normalization benchmark.

## When to use

Use when an inferred GRN must be quantitatively scored against an undirected typed gold standard (e.g., DREAM 4) using edge-level confusion-matrix metrics.

## When NOT to use

- The network to evaluate is a directed, signed Boolean model rather than an undirected edge list compared against an undirected typed gold standard — the required simplification (dropping interaction types, making edges positive) is specific to edge-level GRN scoring.
- The goal is ranking metrics such as AUROC or AUPR on a scored edge list rather than thresholded confusion-matrix proportions (sensitivity/specificity/accuracy).
- No gold standard network exists for the organism or dataset, so true positives and true negatives cannot be defined.

## Inputs

- Inferred GRN edge list (e.g., CSV of gene-gene interactions from Augusta MI-based inference)
- Gold standard network with undirected typed edges (e.g., DREAM 4 In Silico Network Challenge reference network, 100 genes)
- Variant labels for comparison rows (e.g., normalization methods: none, CPM, RPKM, TPM)

## Outputs

- Confusion matrix of predicted versus true edges
- Metrics CSV (e.g., normalization_benchmark_metrics.csv) containing sensitivity, specificity, and accuracy per variant

## How to apply

First simplify the inferred network to match the gold standard format: drop interaction types and convert all negative edges to positive ones so both networks are undirected and untyped. Then compare the predicted edge set against the true edge set (e.g., DREAM 4 In Silico Network Challenge, 100 genes) to build a confusion matrix over edge predictions. From this matrix compute sensitivity as the proportion of true positives, specificity as the proportion of true negatives, and accuracy as the overall proportion of correct predictions. Write the metrics per variant (e.g., per normalization method: none, CPM, RPKM, TPM) to a CSV file for benchmark comparison. Reference values of the same order as the paper's DREAM 4 results (with refinement: sensitivity 0.06, specificity 0.91, accuracy 0.89; no refinement/MI only: 0.38/0.54/0.54) indicate the computation is consistent with the published pipeline.

## Related tools

- **Augusta** (Produces the inferred initial GRNs (via mutual information with MSD edge orientation) that are scored against the gold standard using confusion-matrix metrics) — https://github.com/JanaMus/Augusta
- **Python** (Implements the confusion-matrix computation and writes sensitivity/specificity/accuracy rows to the metrics CSV)

## Examples

```
import pandas as pd; tp, fp, tn, fn = 60, 2240, 3800, 900; metrics = pd.DataFrame({'sensitivity': [tp/(tp+fn)], 'specificity': [tn/(tn+fp)], 'accuracy': [(tp+tn)/(tp+fp+tn+fn)]}); metrics.to_csv('normalization_benchmark_metrics.csv')
```

## Evaluation signals

- The script runs to completion on all four normalization variants (none, CPM, RPKM, TPM) without errors.
- The output metrics CSV contains a sensitivity, specificity, and accuracy row for each of the four normalization methods.
- Metric values are of the same order as the paper's reported DREAM 4 metrics (e.g., sensitivity 0.06, specificity 0.91, accuracy 0.89 with refinement).
- The inferred network has been simplified to the gold standard format: interaction types dropped and negative edges converted to positive before comparison.
- Sensitivity, specificity, and accuracy each fall within [0, 1] and are internally consistent with the edge counts in the confusion matrix.

## Limitations

- Inferred networks should be considered approximations, not perfectly accurate representations of biological systems' regulation.
- Comparing against a typed gold standard requires preprocessing steps (dropping interaction types, converting negative edges to positive) that discard sign and mechanism information.
- Inferring autoregulations remains a significant challenge, which can affect edge-level scoring.

## Evidence

- [results] On the DREAM 4 dataset, Augusta (with refinement) achieved sensitivity 0.06, specificity 0.91, accuracy 0.89 in 714.18 s: [evidence span withheld — anchor status: not_found]
