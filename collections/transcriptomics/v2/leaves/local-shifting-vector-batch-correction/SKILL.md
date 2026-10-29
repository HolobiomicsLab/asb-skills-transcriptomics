---
name: local-shifting-vector-batch-correction
description: Use when integrating two scRNA-seq datasets (e.g., Seurat V3.0 log-count
  matrices) where batch correction should move query cells toward matched reference
  anchors using per-anchor local shifting vectors rather than global or MNN-based
  correction.
when_to_use_negative:
- Integrating more than two datasets — 'The current implementation of BATMAN works
  only with two datasets.'
- Input data are raw counts or already normalized by another pipeline — BATMAN expects
  Seurat V3.0-style log-counts pre-processing.
- The analysis goal requires methods with different structural assumptions (e.g.,
  MNN-based tools) — the paper shows MNN-based methods fail to both correct batch
  effects and maintain local structure, but BATMAN's one-to-one anchor matching may
  be too sparse if fine-grained per-cell correction is needed.
edam_operation: http://edamontology.org/operation_2935
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: BATMAN
  role: Performs anchor identification, minimum-weight matching, local shifting vector
    determination, and batch correction of two scRNA-seq datasets
  repo: https://github.com/mandricigor/batman
- name: Splatter
  role: Simulates LB-scenario scRNA-seq datasets (100 datasets per scenario, 1,000
    genes) used to benchmark BATMAN's batch correction
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.isci.2020.101185
    title: 'BATMAN: Fast and Accurate Integration of Single-Cell RNA-Seq Datasets
      via Minimum-Weight Matching'
  source_content_hash: sha256:e6e9545d86d71e2c2ec818f7815b451ccc1573599b3ad31144540cc57d25960c
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
  edam_operation: http://edamontology.org/operation_2935
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# local-shifting-vector-batch-correction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

BATMAN's batch-correction step: for every matched anchor pair found by minimum-weight matching on the weighted bipartite anchor graph, determine a local shifting vector from the query cell to its matched reference cell in N-dimensional gene space, then apply these vectors to the query dataset to produce the integrated expression matrix.

## When to use

Use when integrating two scRNA-seq datasets (e.g., Seurat V3.0 log-count matrices) where batch correction should move query cells toward matched reference anchors using per-anchor local shifting vectors rather than global or MNN-based correction.

## When NOT to use

- Integrating more than two datasets — 'The current implementation of BATMAN works only with two datasets.'
- Input data are raw counts or already normalized by another pipeline — BATMAN expects Seurat V3.0-style log-counts pre-processing.
- The analysis goal requires methods with different structural assumptions (e.g., MNN-based tools) — the paper shows MNN-based methods fail to both correct batch effects and maintain local structure, but BATMAN's one-to-one anchor matching may be too sparse if fine-grained per-cell correction is needed.

## Inputs

- Two scRNA-seq datasets as log-count matrices (Seurat V3.0-style pre-processing), passed as a comma-separated list of file paths
- Number of anchor points per dataset (optional -anchors parameter)
- Pearson correlation filtering threshold between anchors across datasets (optional -filter-cor parameter)

## Outputs

- Integrated (batch-corrected) dataset containing all cells from the query dataset with corrected expression values
- Matched anchor pairs (one-to-one matching between query and reference anchors)

## How to apply

First run BATMAN's matching pipeline: identify anchor cells in each dataset partition (-anchors parameter controls their number), build the weighted bipartite graph with edges weighted by Euclidean distance in N-dimensional gene space, and solve the minimum-weight one-to-one matching (exact blossom-based algorithm, or the greedy approximation with ratio 2 for extra-large datasets via -greedy). For every matched anchor pair, compute the local shifting vector pointing from the query anchor to its matched reference anchor. Apply these shifting vectors as batch correction to the query dataset so that it is aligned to the reference, keeping only the minimal total correction necessary. Optionally filter anchors by a Pearson correlation threshold (-filter-cor) before matching. The current implementation supports exactly two datasets; only the query dataset is modified during alignment.

## Related tools

- **BATMAN** (Performs anchor identification, minimum-weight matching, local shifting vector determination, and batch correction of two scRNA-seq datasets) — https://github.com/mandricigor/batman
- **Splatter** (Simulates LB-scenario scRNA-seq datasets (100 datasets per scenario, 1,000 genes) used to benchmark BATMAN's batch correction)

## Examples

```
python3.7 batman.py -data query.csv,reference.csv -output integrated.csv -anchors 100 -filter-cor 0.5
```

## Evaluation signals

- The matching must be one-to-one: each query anchor is matched to exactly one reference anchor.
- The exact (blossom-based) matching's total Euclidean edge weight must not exceed the greedy approximation's total weight (greedy has approximation ratio 2).
- The integrated dataset must contain all cells from the query dataset with corrected expression values.
- On the LB simulated scenario, expect mean iLISI ≈ 1.84 (vs 1.00 for Seurat V3.0 and MNNcorrect, 1.01 for Scanorama, 1.82 for original data) and mean 50-RNN ≈ 0.92 (vs 1.00 original, 0.58 Seurat V3.0, 0.51 MNNcorrect, 0.14 Scanorama).
- Shifting vectors must be local per matched anchor pair, pointing from the query cell to its matched reference cell in N-dimensional gene space.

## Limitations

- The current implementation of BATMAN works only with two datasets; multi-dataset integration is not implemented (only possible 'in principle').
- Greedy matching is an approximation with ratio 2, recommended only for extra large datasets; it may yield higher total correction weight than the exact blossom-based algorithm.
- The number of anchors per dataset must be specified manually via the -anchors parameter, and -filter-cor applies a Pearson correlation threshold on anchors across datasets that can exclude anchor pairs.

## Evidence

- [readme] Weighted bipartite graph with Euclidean edge weights: [evidence span withheld — anchor status: external_doc]
