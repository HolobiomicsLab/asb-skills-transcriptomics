---
name: euclidean-distance-knn-graph-construction
description: Use when integrating two scRNA-seq datasets (e.g., Muraro CEL-Seq2 and
  Segerstolpe Smart-Seq2 log-count matrices) and you need to build Euclidean-distance
  neighbor sets to match anchors across datasets or quantify neighborhood retention
  after batch correction.
when_to_use_negative:
- The current BATMAN implementation only supports integrating two datasets; do not
  rely on this skill for multi-dataset integration workflows
- Data are raw counts without Seurat V3.0-style log-count pre-processing; Euclidean
  distances in raw count space are not what the method assumes
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: BATMAN
  role: Builds the Euclidean-distance-weighted bipartite anchor graph and matches
    anchors to perform batch correction while preserving local structure
  repo: https://github.com/mandricigor/batman
- name: Scanorama
  role: Comparison method whose post-integration neighbor sets are scored against
    pre-integration Euclidean kNN sets via the k-RNN metric
- name: Harmony
  role: Comparison integration tool cited alongside BATMAN in the batch-correction
    evaluation
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.isci.2020.101185
    title: 'BATMAN: Fast and Accurate Integration of Single-Cell RNA-Seq Datasets
      via Minimum-Weight Matching'
  source_content_hash: sha256:a2b195eea4aa9108ee0044613dc4d716e6c5146e05445363e420eafff2274685
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# euclidean-distance-knn-graph-construction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs k-nearest-neighbor graphs weighted by Euclidean distance over cell expression vectors, both as the anchor-matching substrate in BATMAN batch integration and as the basis for the k-RNN (retained nearest neighbors) local-structure metric.

## When to use

Use when integrating two scRNA-seq datasets (e.g., Muraro CEL-Seq2 and Segerstolpe Smart-Seq2 log-count matrices) and you need to build Euclidean-distance neighbor sets to match anchors across datasets or quantify neighborhood retention after batch correction.

## When NOT to use

- The current BATMAN implementation only supports integrating two datasets; do not rely on this skill for multi-dataset integration workflows
- Data are raw counts without Seurat V3.0-style log-count pre-processing; Euclidean distances in raw count space are not what the method assumes

## Inputs

- Log-count expression matrices for two scRNA-seq datasets (e.g., Muraro CEL-Seq2 pancreas, 2,285 cells; Segerstolpe Smart-Seq2 pancreas, 2,394 cells), pre- and post-integration
- Anchor count parameter (-anchors) and optional anchor correlation filter threshold (-filter-cor) for BATMAN graph construction
- k parameter for neighbor computation (k=50 in the paper's LB scenario)

## Outputs

- Weighted bipartite graph with Euclidean-distance edge weights between anchor cells of the two datasets
- Per-cell nearest-neighbor sets (k=50) in pre- and post-integration expression space
- Mean k-RNN (retained nearest neighbors) score per integration method, e.g., aggregated into krnn_scores.csv

## How to apply

Load the log-count matrices (obtained via Seurat V3.0 pre-processing) for each dataset and treat each cell as a point in N-dimensional gene space. For graph construction in BATMAN, build a weighted bipartite graph whose partitions are the two datasets and whose edges are weighted by the Euclidean distance between cells, then find a minimum-weight one-to-one matching of anchor cells (exact blossom-based algorithm, or greedy approximation with ratio 2 for very large datasets). For k-RNN scoring, compute each cell's top-k nearest neighbor set (k=50 in the LB scenario) with Euclidean distance in the pre-integration high-dimensional gene space, recompute the neighbor sets in the post-integration corrected space, and report the average fraction of pre-integration neighbors retained across all cells. In the LB scenario the expected mean 50-RNN pattern is Original 1.00, BATMAN 0.92, Seurat V3.0 0.58, MNNcorrect 0.51, Scanorama 0.14, which serves as a validation target for correct implementation.

## Related tools

- **BATMAN** (Builds the Euclidean-distance-weighted bipartite anchor graph and matches anchors to perform batch correction while preserving local structure) — https://github.com/mandricigor/batman
- **Scanorama** (Comparison method whose post-integration neighbor sets are scored against pre-integration Euclidean kNN sets via the k-RNN metric)
- **Harmony** (Comparison integration tool cited alongside BATMAN in the batch-correction evaluation)

## Examples

```
from sklearn.neighbors import NearestNeighbors; nn = NearestNeighbors(n_neighbors=50, metric='euclidean').fit(logcounts)
```

## Evaluation signals

- Reproduce the LB-scenario mean 50-RNN pattern: Original 1.00, BATMAN 0.92, Seurat V3.0 0.58, MNNcorrect 0.51, Scanorama 0.14
- Original (unintegrated) data must yield a 50-RNN of exactly 1.00 since pre- and post-integration spaces are identical
- Check that each cell's kNN set contains exactly k=50 neighbors computed with Euclidean distance in the log-count gene space
- Verify the matched anchor pairs form a one-to-one minimum-weight matching (blossom-based exact solution or greedy approximation ratio 2)

## Limitations

- BATMAN's current implementation works only with two datasets; multi-dataset integration is possible only 'in principle'
- The greedy matching heuristic is an approximation with ratio 2, trading exactness of the minimum-weight matching for speed on very large datasets
- Local-structure preservation (k-RNN) and batch mixing (iLISI) can trade off: e.g., Scanorama mixes but retains only 0.14 mean 50-RNN in the LB scenario

## Evidence

- [readme] Building a weighted bipartite graph where each partition corresponds to each of the datasets and the edges are weighted by the Euclidean distance: [evidence span withheld — anchor status: external_doc]
- [other] The k-RNN metric is computed by determining each cell's top-k nearest neighbors in the original dataset and again after integration: [evidence span withheld — anchor status: not_found]
