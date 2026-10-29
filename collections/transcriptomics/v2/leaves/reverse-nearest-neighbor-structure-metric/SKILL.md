---
name: reverse-nearest-neighbor-structure-metric
description: Use when benchmarking scRNA-seq batch-effect correction methods (e.g.
  BATMAN vs Seurat V3.0, MNNcorrect, Scanorama) and you need to quantify retention
  of local structure alongside batch mixing (iLISI).
when_to_use_negative:
- When the task only requires measuring batch mixing and local-structure retention
  is irrelevant — iLISI alone would suffice, though the paper argues this is insufficient
- 'When comparing more than two datasets with BATMAN itself: the current implementation
  only supports two-dataset integration, so per-pair RNN scoring must be redesigned'
- When the input is raw counts or a clustering/label table rather than a cell-by-gene
  log-count matrix from a common pre-processing step (Seurat V3.0 log-counts in the
  paper)
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: BATMAN
  role: Correction method whose output is evaluated with 50-RNN; achieved the highest
    structure retention (0.92) among corrected methods in the LB scenario
  repo: https://github.com/mandricigor/batman
- name: Scanorama
  role: Comparison method evaluated with 50-RNN; scored lowest structure retention
    (0.14) in the LB scenario
- name: Splatter
  role: Generates the simulated LB-scenario dataset pairs (100 datasets, 1,000 genes)
    on which 50-RNN is benchmarked
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.isci.2020.101185
    title: 'BATMAN: Fast and Accurate Integration of Single-Cell RNA-Seq Datasets
      via Minimum-Weight Matching'
  source_content_hash: sha256:65864d52d937d776bef098a090ae07402ae2d2a5303a27cab655b9c87c33e7cc
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
  - MNNcorrect
  - Scanorama
  - Splatter
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# Reverse-nearest-neighbor structure metric (50-RNN)

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

A local-structure-preservation metric used in the BATMAN paper (iScience 2020) to score how well a batch-correction method retains cell-to-cell relationships: it measures the fraction of cells whose 50 reverse-nearest-neighbors in the corrected data match those in the original (uncorrected) data. It complements batch-mixing metrics like iLISI, which alone can be inflated by over-correction.

## When to use

Use when benchmarking scRNA-seq batch-effect correction methods (e.g. BATMAN vs Seurat V3.0, MNNcorrect, Scanorama) and you need to quantify retention of local structure alongside batch mixing (iLISI).

## When NOT to use

- When the task only requires measuring batch mixing and local-structure retention is irrelevant — iLISI alone would suffice, though the paper argues this is insufficient
- When comparing more than two datasets with BATMAN itself: the current implementation only supports two-dataset integration, so per-pair RNN scoring must be redesigned
- When the input is raw counts or a clustering/label table rather than a cell-by-gene log-count matrix from a common pre-processing step (Seurat V3.0 log-counts in the paper)

## Inputs

- Corrected (batch-corrected) expression matrix of log-counts, e.g. from batman.py, Seurat V3.0, MNNcorrect, or Scanorama
- Original (no-correction) concatenated log-count matrix as the reference for neighborhood comparison
- Neighborhood size parameter k=50 (the 50-RNN variant used in the paper)

## Outputs

- Mean 50-RNN score per correction method (value in [0,1]; Original baseline = 1.00)
- Method-comparison table entry, e.g. lb_integration_metrics.csv with mean iLISI and mean 50-RNN per method

## How to apply

After running each correction method on the same preprocessed input pair (Seurat V3.0 pre-processing to log-counts is used in the paper), compute the 50-RNN on the corrected expression matrix against the Original (no-correction concatenation) baseline: for each cell, take its 50 reverse nearest neighbors and score agreement between corrected and original neighborhoods; average over cells to get the mean 50-RNN. Values near 1.00 indicate the correction preserved local cell-to-cell relationships (Original by definition scores 1.00); lower values indicate distorted local structure. In the paper's large-batch-effects (LB) simulated scenario (100 Splatter-simulated dataset pairs, 1,000 genes each), mean 50-RNN was 1.00 (Original), 0.92 (BATMAN), 0.58 (Seurat V3.0), 0.51 (MNNcorrect), and 0.14 (Scanorama), so corrected methods can be ranked by structure retention. Report it jointly with mean iLISI, since a method can maximize mixing (iLISI) while destroying neighborhoods.

## Related tools

- **BATMAN** (Correction method whose output is evaluated with 50-RNN; achieved the highest structure retention (0.92) among corrected methods in the LB scenario) — https://github.com/mandricigor/batman
- **Scanorama** (Comparison method evaluated with 50-RNN; scored lowest structure retention (0.14) in the LB scenario)
- **Splatter** (Generates the simulated LB-scenario dataset pairs (100 datasets, 1,000 genes) on which 50-RNN is benchmarked)

## Evaluation signals

- Original (no-correction) baseline must score mean 50-RNN = 1.00 by construction
- In the LB simulation scenario, corrected methods should fall in the reported range: BATMAN ≈ 0.92, Seurat V3.0 ≈ 0.58, MNNcorrect ≈ 0.51, Scanorama ≈ 0.14 (within simulation variance)
- BATMAN should preserve the highest 50-RNN among corrected methods while also achieving the best mean iLISI (1.84 vs 1.82 original)
- All methods must be scored on the same Seurat V3.0 pre-processed log-count matrices so neighborhood comparisons are like-for-like
- Metric must be computed at k=50 reverse nearest neighbors to match the paper's 50-RNN definition

## Limitations

- Demonstrated on the paper's LB simulated scenario (Splatter, 100 datasets of 1,000 genes); values on other scenarios or real data will differ and need re-baselining against the Original data
- The metric rewards preservation of original structure, so it must be read jointly with iLISI — a trivial 'no-correction' run scores perfectly on RNN but fails batch mixing (Original iLISI 1.82 vs BATMAN 1.84)
- The paper reports the metric for the mutual-nearest-neighbor-paradigm methods it criticizes; interpretations for other correction paradigms are extrapolation

## Evidence
