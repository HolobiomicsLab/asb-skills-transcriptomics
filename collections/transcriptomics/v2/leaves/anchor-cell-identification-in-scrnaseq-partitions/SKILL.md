---
name: anchor-cell-identification-in-scrnaseq-partitions
description: Use when integrating two scRNA-seq datasets affected by batch effects
  and you need a parsimonious set of representative anchor cells per dataset before
  building a weighted bipartite graph for one-to-one matching.
when_to_use_negative:
- You need to integrate more than two datasets — the current BATMAN implementation
  works only with two datasets
- Input is raw UMI counts without Seurat V3.0-style log-count pre-processing — anchors
  are identified in log-count space
- A non-matching-based integration approach is required (e.g. MNN-based tools like
  MNNcorrect, Scanorama, or Harmony are the alternatives)
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: BATMAN
  role: Tool whose pipeline identifies anchor cells per dataset partition (-anchors
    parameter) and uses them as graph nodes for minimum-weight matching
  repo: https://github.com/mandricigor/batman
- name: Splatter
  role: Used to simulate LB-scenario scRNA-seq datasets on which anchor identification
    and matching were validated
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.isci.2020.101185
    title: 'BATMAN: Fast and Accurate Integration of Single-Cell RNA-Seq Datasets
      via Minimum-Weight Matching'
  source_content_hash: sha256:e4aa927d2f220c32b00656a1e9e1c3e6614e70924bbd3662d6171341c9ccc8c3
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
  - Splatter
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# anchor-cell-identification-in-scrnaseq-partitions

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Identify representative anchor cells within each partition of two scRNA-seq datasets (after Seurat V3.0-style log-count pre-processing) to serve as graph nodes for BATMAN's minimum-weight bipartite matching and downstream batch correction.

## When to use

Use when integrating two scRNA-seq datasets affected by batch effects and you need a parsimonious set of representative anchor cells per dataset before building a weighted bipartite graph for one-to-one matching.

## When NOT to use

- You need to integrate more than two datasets — the current BATMAN implementation works only with two datasets
- Input is raw UMI counts without Seurat V3.0-style log-count pre-processing — anchors are identified in log-count space
- A non-matching-based integration approach is required (e.g. MNN-based tools like MNNcorrect, Scanorama, or Harmony are the alternatives)

## Inputs

- Two scRNA-seq count matrices (e.g. CSV/TSV), one per batch, treated as two dataset partitions
- Seurat V3.0-style log-count matrices after pre-processing
- Number of anchors per dataset (-anchors parameter)

## Outputs

- Sets of representative anchor cells, one set per dataset partition
- Anchor nodes consumed by the weighted bipartite graph construction
- matched anchor pairs written to matching_edges.csv (after the matching step)

## How to apply

First pre-process each dataset with Seurat V3.0-style pre-processing to obtain log-count matrices, treating each dataset as one partition of a combined graph. Then identify the set of anchors in each dataset; the number of anchors is controlled by the -anchors parameter of batman.py, and anchors can be filtered with a Pearson correlation threshold across datasets via -filter-cor. These anchors become the nodes of the weighted bipartite graph, where each partition corresponds to one dataset and edges are weighted by Euclidean distance between cells in N-dimensional gene space. The choice of anchor count trades matching granularity against runtime; for extra-large datasets the greedy approximation (ratio 2) is recommended downstream. The identified anchors are the sole cells used to compute local shifting vectors for batch correction, so poor anchor selection directly degrades local structure preservation measured by metrics such as mean 50-RNN.

## Related tools

- **BATMAN** (Tool whose pipeline identifies anchor cells per dataset partition (-anchors parameter) and uses them as graph nodes for minimum-weight matching) — https://github.com/mandricigor/batman
- **Splatter** (Used to simulate LB-scenario scRNA-seq datasets on which anchor identification and matching were validated)

## Examples

```
python3.7 batman.py --data query.csv,reference.csv --output integrated.csv --anchors 100 --filter-cor 0.7
```

## Evaluation signals

- Each query anchor must be matched to exactly one reference anchor (one-to-one matching invariant)
- Anchor count per partition must equal the value supplied via -anchors
- The exact (blossom-based) matching's total Euclidean edge weight must not exceed the greedy matching's total weight
- The integrated dataset must contain all cells from the query dataset with corrected expression values
- Local structure retention can be assessed with mean 50-RNN (BATMAN: 0.92 vs Seurat V3.0: 0.58 in the LB scenario)

## Limitations

- The current implementation of BATMAN works only with two datasets; multi-dataset integration is not implemented
- Anchor number (-anchors) and the Pearson correlation filter threshold (-filter-cor) must be chosen per dataset; no universal defaults are specified in the article
- Performance depends on the quality of log-count pre-processing (Seurat V3.0 steps); noisy or poorly normalized inputs degrade anchor representativeness

## Evidence

- [readme] Identifying the set of anchors in each of the datasets (their number is specified with parameter -anchors): "1. Identifying the set of anchors in each of the datasets (their number is specified with parameter -anchors)"
- [methods] each partition corresponds to each of the datasets and the edges are weighted by the Euclidean distance between the points (cells in N-dimensional space): "each partition corresponds to each of the datasets and the edges are weighted by the Euclidean distance between the points (cells in N-dimensional space)"
- [results] we used Seurat V3.0 pre-processing steps to obtain log-counts for each dataset: "In all the experiments, we used Seurat V3.0 pre-processing steps to obtain log-counts for each dataset."
- [intro] based on a parsimonious one-to-one matching of representative cells across: "we present a method for integration of single-cell datasets called BATMAN (BATch integration via minimum weight MAtchiNg), based on a parsimonious one-to-one matching of representative cells across"
