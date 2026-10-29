---
name: greedy-approximate-matching-on-weighted-graphs
description: Use when a minimum-weight one-to-one matching is needed on a large weighted
  bipartite graph (e.g. many anchor cells across two scRNA-seq batches) and the exact
  blossom algorithm is too slow; accepts up to 2x suboptimality.
when_to_use_negative:
- When the exact minimum total weight is required and dataset size permits the blossom-based
  algorithm — the greedy result can be up to twice the optimal total edge weight
- When integrating more than two datasets — the current BATMAN implementation only
  supports two datasets
- When inputs are not log-count matrices with identified anchor cells (e.g. raw counts
  or already-integrated embeddings)
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: BATMAN
  role: Implements the greedy approximation (approximation ratio 2) for minimum-weight
    matching via the -greedy flag, used as a faster alternative to the exact blossom-based
    algorithm on extra-large datasets
  repo: https://github.com/mandricigor/batman
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.isci.2020.101185
    title: 'BATMAN: Fast and Accurate Integration of Single-Cell RNA-Seq Datasets
      via Minimum-Weight Matching'
  source_content_hash: sha256:c0b13404bb9a970adae57ce3bcea93e832f238767a3c0c0f739c3a922e41173e
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
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7777777777777778
n_claims: 9
---

# greedy-approximate-matching-on-weighted-graphs

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

An approximate alternative to the exact blossom-based minimum-weight matching: edges of a weighted bipartite graph (here, anchor cells across two scRNA-seq datasets, weighted by Euclidean distance in N-dimensional gene space) are matched greedily, guaranteeing a total weight within a factor of 2 of optimal. BATMAN exposes this via the -greedy flag and recommends it for extra-large datasets.

## When to use

Use when a minimum-weight one-to-one matching is needed on a large weighted bipartite graph (e.g. many anchor cells across two scRNA-seq batches) and the exact blossom algorithm is too slow; accepts up to 2x suboptimality.

## When NOT to use

- When the exact minimum total weight is required and dataset size permits the blossom-based algorithm — the greedy result can be up to twice the optimal total edge weight
- When integrating more than two datasets — the current BATMAN implementation only supports two datasets
- When inputs are not log-count matrices with identified anchor cells (e.g. raw counts or already-integrated embeddings)

## Inputs

- Two log-count scRNA-seq matrices pre-processed with Seurat V3.0 steps (query and query dataset)
- Weighted bipartite graph with partitions corresponding to each dataset and edge weights equal to Euclidean distance between anchor cells in N-dimensional gene space
- Number of anchor points per dataset (BATMAN -anchors parameter)

## Outputs

- One-to-one matching of anchor cells across the two datasets (matched anchor pairs)
- Matching edge list written to matching_edges.csv
- Total matching weight within a factor of 2 of the exact (blossom-based) minimum

## How to apply

First identify representative anchor cells in each dataset partition (BATMAN parameter -anchors sets the number per dataset) and build the weighted bipartite graph whose edges are Euclidean distances between cells in N-dimensional gene space. Instead of running the exact blossom-based algorithm, run the greedy approximation, which repeatedly selects the lowest-weight available edge and is guaranteed an approximation ratio of 2 versus the optimal total matching weight. In BATMAN this is enabled with the -greedy flag, which the README recommends 'for extra large datasets'. Optionally filter anchor pairs with -filter-cor (threshold on Pearson correlation between anchors across datasets) before or after matching. The resulting matched pairs feed directly into local shifting-vector computation and batch correction, exactly as the exact matching's pairs would.

## Related tools

- **BATMAN** (Implements the greedy approximation (approximation ratio 2) for minimum-weight matching via the -greedy flag, used as a faster alternative to the exact blossom-based algorithm on extra-large datasets) — https://github.com/mandricigor/batman

## Examples

```
python3.7 batman.py -data query.csv,reference.csv -output integrated.csv -anchors 100 -filter-cor 0.7 -greedy
```

## Evaluation signals

- Matching is one-to-one: each query anchor is matched to exactly one reference anchor
- Total Euclidean edge weight of the greedy matching does not exceed 2x the total weight of the exact blossom-based matching run on the same graph
- Greedy matching runs faster than the exact algorithm on the same anchor graph (motivating its use for extra-large datasets)
- Downstream integration quality: mean iLISI close to the original-data value (e.g. 1.84 vs 1.82 in the LB scenario) and mean 50-RNN retention (0.92 in the LB scenario)
- Every matched anchor pair passes the -filter-cor Pearson correlation threshold across datasets

## Limitations

- Approximation ratio is 2, so the greedy total weight can be up to twice the optimal blossom-based minimum
- Current BATMAN implementation only supports integration of two datasets; multi-dataset integration is not implemented
- Number of anchors must be chosen a priori via -anchors, which affects graph size and matching quality

## Evidence

- [readme] Building a weighted bipartite graph where each partition corresponds to each of the datasets: [evidence span withheld — anchor status: external_doc]
