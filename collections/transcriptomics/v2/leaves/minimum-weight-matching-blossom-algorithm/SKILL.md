---
name: minimum-weight-matching-blossom-algorithm
description: 'Use when integrating two scRNA-seq batches via BATMAN: anchors have
  been identified per dataset and a globally minimal one-to-one cell-to-cell correspondence
  (not mutual-nearest-neighbor) is needed to derive local shifting vectors for batch
  correction.'
when_to_use_negative:
- The task requires integrating more than two datasets — the current BATMAN implementation
  only works with two datasets (multi-dataset integration is possible only 'in principle')
- The goal is a many-to-many or mutual-nearest-neighbor alignment rather than a strict
  one-to-one parsimonious matching of representative cells
- The input is a single dataset or the batches have no shared biological cell types,
  so meaningful cross-dataset anchor pairs cannot be identified
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: BATMAN
  role: 'Implements the full matching-based integration: anchor identification, weighted
    bipartite graph construction, exact (blossom) or greedy minimum-weight matching,
    shifting-vector computation, and batch correction'
  repo: https://github.com/mandricigor/batman
- name: Seurat
  role: V3.0 pre-processing steps used to obtain log-counts for each dataset before
    anchor selection and matching
- name: Splatter
  role: Simulation of LB-scenario scRNA-seq datasets (100 datasets per scenario, 1,000
    genes) used to validate the matching-based integration
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.isci.2020.101185
    title: 'BATMAN: Fast and Accurate Integration of Single-Cell RNA-Seq Datasets
      via Minimum-Weight Matching'
  source_content_hash: sha256:4fc7a2ab3fda556f8fa35917e50719d540ebdf45825f153789820bc122d9622e
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
  - Seurat
  - Splatter
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# minimum-weight-matching-blossom-algorithm

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Computes a parsimonious one-to-one minimum-weight matching between representative anchor cells of two scRNA-seq datasets on a weighted bipartite graph, as the core of the BATMAN batch integration method. Edge weights are Euclidean distances between cells in N-dimensional gene (log-count) space, and the exact blossom-based algorithm can be swapped for a greedy 2-approximation.

## When to use

Use when integrating two scRNA-seq batches via BATMAN: anchors have been identified per dataset and a globally minimal one-to-one cell-to-cell correspondence (not mutual-nearest-neighbor) is needed to derive local shifting vectors for batch correction.

## When NOT to use

- The task requires integrating more than two datasets — the current BATMAN implementation only works with two datasets (multi-dataset integration is possible only 'in principle')
- The goal is a many-to-many or mutual-nearest-neighbor alignment rather than a strict one-to-one parsimonious matching of representative cells
- The input is a single dataset or the batches have no shared biological cell types, so meaningful cross-dataset anchor pairs cannot be identified

## Inputs

- Two scRNA-seq count matrices (one query, one reference) pre-processed to log-counts with Seurat V3.0-style steps
- Number of anchor points to consider in each dataset (-anchors parameter)
- Optional filtering threshold on Pearson correlation between anchors across datasets (-filter-cor)

## Outputs

- A one-to-one matching of anchors across the two datasets (matched anchor pairs, e.g. matching_edges.csv in the workflow)
- Local shifting vectors per matched anchor pair used for batch correction
- An integrated (batch-corrected) query dataset with corrected expression values

## How to apply

First identify a set of anchor cells in each dataset (count controlled by the -anchors parameter), after Seurat V3.0-style pre-processing to log-counts. Build a weighted bipartite graph with one partition per dataset and edges weighted by Euclidean distance between anchor cells in N-dimensional gene space. Run the minimum-weight matching either with the exact blossom-based algorithm or with the greedy approximation (approximation ratio 2, recommended for extra large datasets via the -greedy flag); optionally filter anchors by a Pearson correlation threshold (-filter-cor). For every matched anchor pair, determine the local shifting vector pointing from the query cell to its matched reference cell and apply these vectors as batch correction, which yields the minimal total correction necessary to remove discrepancies between the datasets.

## Related tools

- **BATMAN** (Implements the full matching-based integration: anchor identification, weighted bipartite graph construction, exact (blossom) or greedy minimum-weight matching, shifting-vector computation, and batch correction) — https://github.com/mandricigor/batman
- **Seurat** (V3.0 pre-processing steps used to obtain log-counts for each dataset before anchor selection and matching)
- **Splatter** (Simulation of LB-scenario scRNA-seq datasets (100 datasets per scenario, 1,000 genes) used to validate the matching-based integration)

## Examples

```
python3.7 batman.py -data query_counts.csv,ref_counts.csv -output integrated.csv -anchors 100 -filter-cor 0.7
```

## Evaluation signals

- The matching is one-to-one: each query anchor is matched to exactly one reference anchor
- The exact blossom-based matching's total Euclidean edge weight does not exceed the greedy matching's total weight (greedy has approximation ratio 2)
- The integrated dataset contains all cells from the query dataset with corrected expression values
- In the LB simulation scenario, mean iLISI ≈ 1.84 (vs 1.00 for Seurat V3.0 and MNNcorrect) and mean 50-RNN ≈ 0.92 (vs 0.58/0.51/0.14), indicating mixing without loss of local structure
- Matched anchor pairs are written out (e.g. matching_edges.csv) so the correspondence can be inspected

## Limitations

- The current implementation of BATMAN works only with two datasets; multi-dataset integration is not implemented
- The greedy matching is a 2-approximation and may produce a matching with up to twice the total weight of the exact blossom-based solution
- The number of anchors per dataset is a user-specified parameter (-anchors) and a Pearson correlation threshold (-filter-cor) must be chosen for anchor filtering

## Evidence

- [readme] Building a weighted bipartite graph where each partition corresponds to each of the datasets and the edges are weighted by the Euclidean distance between the points (cells in N-dimensional space) corresponding to their endpoints: [evidence span withheld — anchor status: external_doc]
