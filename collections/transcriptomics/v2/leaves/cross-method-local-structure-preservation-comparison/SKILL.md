---
name: cross-method-local-structure-preservation-comparison
description: Use when comparing batch-correction/integration methods on scRNA-seq
  data and you need a quantitative measure of local structure preservation (e.g.,
  before adopting an integration method for a two-dataset comparison).
when_to_use_negative:
- When the goal is to measure batch mixing/cell-type mixing rather than local structure
  preservation — use a metric such as mean iLISI instead
- 'When more than two datasets must be integrated: the current BATMAN implementation
  (whose corrected outputs feed this comparison) works only with two datasets'
edam_operation: http://edamontology.org/operation_2938
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: BATMAN
  role: Integration method whose corrected output is evaluated for local structure
    preservation (expected to retain the most pre-integration neighbors, mean 50-RNN
    = 0.92)
  repo: https://github.com/mandricigor/batman
- name: Scanorama
  role: MNN-based integration method serving as a comparison baseline in the k-RNN
    evaluation (lowest mean 50-RNN, 0.14)
- name: Seurat V3.0
  role: Provides pre-processing steps to obtain log-counts for each dataset and serves
    as a comparison integration method (mean 50-RNN = 0.58)
- name: Harmony
  role: Integration tool cited alongside the compared methods in the batch-effect
    correction context
- name: BERMUDA
  role: Deep transfer learning batch-correction tool cited as a state-of-the-art integration
    baseline
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.isci.2020.101185
    title: 'BATMAN: Fast and Accurate Integration of Single-Cell RNA-Seq Datasets
      via Minimum-Weight Matching'
  source_content_hash: sha256:ab7103912db5654bc4e6183ec9ead9fc58560538c8af7c569eb1a2e784e3d5d5
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/mandricigor/batman
  repo_url: https://github.com/mandricigor/batman
  tools:
  - BATMAN
  - BERMUDA
  - Harmony
  - Scanorama
  - Seurat V3.0
  edam_operation: http://edamontology.org/operation_2938
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# cross-method-local-structure-preservation-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Quantifies how well scRNA-seq integration methods (BATMAN, Seurat V3.0, MNNcorrect, Scanorama) preserve each cell's local neighborhood by computing the k-RNN (retained nearest neighbors) score — the fraction of each cell's top-k pre-integration nearest neighbors retained after batch correction.

## When to use

Use when comparing batch-correction/integration methods on scRNA-seq data and you need a quantitative measure of local structure preservation (e.g., before adopting an integration method for a two-dataset comparison).

## When NOT to use

- When the goal is to measure batch mixing/cell-type mixing rather than local structure preservation — use a metric such as mean iLISI instead
- When more than two datasets must be integrated: the current BATMAN implementation (whose corrected outputs feed this comparison) works only with two datasets

## Inputs

- Pre-integration log-count gene expression matrix for each dataset (e.g., Muraro CEL-Seq2, 2,285 cells; Segerstolpe Smart-Seq2, 2,394 cells)
- Post-integration corrected expression matrices for each integration method being compared (BATMAN, Seurat V3.0, MNNcorrect, Scanorama)
- Neighborhood size parameter k (k=50 in the paper)

## Outputs

- Per-method mean k-RNN (e.g., mean 50-RNN) scores
- Comparison table saved as krnn_scores.csv with columns/methods: Original, BATMAN, Seurat V3.0, MNNcorrect, Scanorama

## How to apply

Load log-count matrices for each dataset both pre- and post-integration (in the BATMAN paper, Muraro CEL-Seq2 with 2,285 cells and Segerstolpe Smart-Seq2 with 2,394 cells, preprocessed with Seurat V3.0 to obtain log-counts). For every cell, compute its top-k (k=50) nearest-neighbor set in the high-dimensional gene space using Euclidean distance on the original data, then recompute the post-integration nearest-neighbor sets in each method's corrected expression space. The k-RNN score per method is the fraction of pre-integration neighbors retained after integration, averaged over all cells in the union of the two datasets. Aggregate the per-method mean 50-RNN values into a comparison table (krnn_scores.csv) covering Original (1.00 by definition), BATMAN, Seurat V3.0, MNNcorrect, and Scanorama. Rationale: methods based on the mutual nearest neighbor paradigm can distort local structure while correcting batch effects, so a low k-RNN alongside good mixing (mean iLISI) signals over-correction.

## Related tools

- **BATMAN** (Integration method whose corrected output is evaluated for local structure preservation (expected to retain the most pre-integration neighbors, mean 50-RNN = 0.92)) — https://github.com/mandricigor/batman
- **Scanorama** (MNN-based integration method serving as a comparison baseline in the k-RNN evaluation (lowest mean 50-RNN, 0.14))
- **Seurat V3.0** (Provides pre-processing steps to obtain log-counts for each dataset and serves as a comparison integration method (mean 50-RNN = 0.58))
- **Harmony** (Integration tool cited alongside the compared methods in the batch-effect correction context)
- **BERMUDA** (Deep transfer learning batch-correction tool cited as a state-of-the-art integration baseline)

## Examples

```
from scipy.spatial.distance import cdist; D = cdist(X, X, metric='euclidean'); knn_pre = np.argsort(D, axis=1)[:, 1:k+1]  # k=50; krnn = np.mean([len(set(pre) & set(post))/k for pre, post in zip(knn_pre, knn_post)]); pd.DataFrame(scores).to_csv('krnn_scores.csv')
```

## Evaluation signals

- The Original (uncorrected) data must yield a mean 50-RNN of 1.00 by construction (identical neighbor sets pre and post)
- Computed mean 50-RNN values should reproduce the reported pattern: BATMAN 0.92, Seurat V3.0 0.58, MNNcorrect 0.51, Scanorama 0.14
- Per-cell retained-neighbor fraction must lie in [0, 1]; the aggregated mean should also fall within this range
- A method with strong iLISI mixing but low k-RNN (e.g., Scanorama: iLISI 1.01, 50-RNN 0.14) indicates over-correction, confirming the metric discriminates structure preservation from mixing
- Output table krnn_scores.csv must contain one row/value per evaluated method covering Original, BATMAN, Seurat V3.0, MNNcorrect, and Scanorama

## Limitations

- The current BATMAN implementation works only with two datasets; multi-dataset integration is not implemented (only possible 'in principle')
- The metric is computed within each cell's original dataset after integration, so it measures within-batch structure retention and does not by itself quantify cross-batch mixing — pair it with iLISI for a balanced assessment
- Results are demonstrated on pancreatic CEL-Seq2/Smart-Seq2 pairs with k=50 and Euclidean distance in high-dimensional gene space; other k values or distance metrics are not validated in this context

## Evidence
