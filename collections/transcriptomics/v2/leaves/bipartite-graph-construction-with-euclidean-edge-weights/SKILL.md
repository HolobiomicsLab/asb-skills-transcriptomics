---
name: bipartite-graph-construction-with-euclidean-edge-weights
description: 'Use when integrating two scRNA-seq datasets via minimum-weight matching:
  after Seurat V3.0-style log-count pre-processing and anchor identification, a cross-dataset
  weighted graph must be built before matching.'
when_to_use_negative:
- More than two datasets must be integrated — the current BATMAN implementation works
  only with two datasets
- Expression values have not been normalized to log-counts (e.g. raw counts or non-Seurat
  V3.0 pre-processed matrices), since edge weights are computed in the pre-processed
  gene space
- Data are not single-cell expression matrices where cell-to-cell Euclidean distances
  in gene space are meaningful
edam_operation: http://edamontology.org/operation_3925
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: BATMAN
  role: Implements anchor identification, bipartite graph construction with Euclidean
    edge weights, and minimum-weight matching for batch correction of two scRNA-seq
    datasets
  repo: https://github.com/mandricigor/batman
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.isci.2020.101185
    title: 'BATMAN: Fast and Accurate Integration of Single-Cell RNA-Seq Datasets
      via Minimum-Weight Matching'
  source_content_hash: sha256:f8d72c9e2258d0b0ba9e2f0cb846c6eba7659db5e52a03e493716d3fdb179ccd
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
  edam_operation: http://edamontology.org/operation_3925
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# Bipartite graph construction with Euclidean edge weights

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs the weighted bipartite anchor graph at the heart of the BATMAN scRNA-seq integration method: one partition per dataset, nodes are representative anchor cells, and edges between cross-dataset anchors are weighted by Euclidean distance in N-dimensional gene (log-count) space.

## When to use

Use when integrating two scRNA-seq datasets via minimum-weight matching: after Seurat V3.0-style log-count pre-processing and anchor identification, a cross-dataset weighted graph must be built before matching.

## When NOT to use

- More than two datasets must be integrated — the current BATMAN implementation works only with two datasets
- Expression values have not been normalized to log-counts (e.g. raw counts or non-Seurat V3.0 pre-processed matrices), since edge weights are computed in the pre-processed gene space
- Data are not single-cell expression matrices where cell-to-cell Euclidean distances in gene space are meaningful

## Inputs

- Two scRNA-seq datasets as log-count matrices (Seurat V3.0-style pre-processing)
- Set of anchor cells per dataset (count specified via -anchors parameter)
- Optional Pearson correlation filtering threshold between anchors across datasets (-filter-cor)

## Outputs

- Weighted bipartite graph with one partition per dataset and cross-partition edges weighted by Euclidean distance in N-dimensional gene space
- Matched anchor pairs (written to matching_edges.csv by the downstream matching step)

## How to apply

After pre-processing each dataset with Seurat V3.0-style steps to obtain log-counts, identify a set of representative anchor cells within each dataset (the number set with BATMAN's -anchors parameter). Treat each dataset as one partition of a combined graph, so the graph is bipartite with anchors of dataset A on one side and anchors of dataset B on the other. Add edges connecting every anchor pair across partitions, weighting each edge by the Euclidean distance between the two cells in N-dimensional gene space; an optional -filter-cor threshold prunes anchors by Pearson correlation across datasets. Rationale: weighting by Euclidean distance makes the subsequent minimum-weight matching (exact blossom-based or greedy approximation with ratio 2) favor matched anchor pairs that require minimal total correction, preserving local structure.

## Related tools

- **BATMAN** (Implements anchor identification, bipartite graph construction with Euclidean edge weights, and minimum-weight matching for batch correction of two scRNA-seq datasets) — https://github.com/mandricigor/batman

## Examples

```
python3.7 batman.py -data datasetA.csv,datasetB.csv -output integrated/ -anchors 100 -filter-cor 0.5
```

## Evaluation signals

- Each query anchor is matched to exactly one reference anchor (one-to-one matching on the bipartite graph)
- The exact (blossom-based) matching's total Euclidean edge weight must not exceed the greedy approximation's total weight (greedy has approximation ratio 2)
- The integrated dataset contains all cells from the query dataset with corrected expression values
- Edge weights correspond to Euclidean distances between anchor cells in the N-dimensional log-count gene space

## Limitations

- The current BATMAN implementation works only with two datasets; multi-dataset integration is not implemented (only possible 'in principle')
- For extra large datasets the greedy heuristic is recommended over the exact blossom-based algorithm, trading optimality (approximation ratio 2) for speed
- Number of anchors (-anchors) and the Pearson correlation filtering threshold (-filter-cor) must be chosen, affecting graph size and matching quality

## Evidence

- [readme] Building a weighted bipartite graph where each partition corresponds to each of the datasets and the edges are weighted by the Euclidean distance between the points (cells in N-dimensional space) corresponding to their endpoints: [evidence span withheld — anchor status: external_doc]
- [methods] each partition corresponds to each of the datasets and the edges are weighted by the Euclidean distance between the points (cells in N-dimensional space): "each partition corresponds to each of the datasets and the edges are weighted by the Euclidean distance between the points (cells in N-dimensional space)"
