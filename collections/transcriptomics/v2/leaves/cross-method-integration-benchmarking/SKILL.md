---
name: cross-method-integration-benchmarking
description: Use when two scRNA-seq batches must be integrated and several correction
  tools should be compared quantitatively on the same inputs using iLISI (batch mixing)
  and 50-RNN (local structure retention) against an uncorrected Original baseline.
when_to_use_negative:
- The input comprises more than two datasets — the current BATMAN implementation works
  only with two datasets, so multi-dataset integration cannot follow this skill as-is
- There is only a single batch (no batch effects to correct), so iLISI mixing gains
  and integration benchmarking are meaningless
- The analysis goal is cell-type clustering or trajectory inference rather than quantifying
  batch-correction quality and local structure retention
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: BATMAN
  role: Method under test; builds a weighted bipartite graph of anchors with Euclidean-distance
    edges, solves minimum-weight matching (exact blossom or greedy, ratio 2), and
    applies local shifting vectors for batch correction
  repo: https://github.com/mandricigor/batman
- name: Seurat V3.0
  role: Provides shared pre-processing to log-counts for all methods and serves as
    one of the three comparison integration methods
- name: MNNcorrect
  role: Comparison method (mutual nearest neighbor based) benchmarked on the same
    preprocessed pair
- name: Scanorama
  role: Comparison method benchmarked on the same preprocessed pair
- name: Splatter
  role: Simulates the 100-dataset-per-scenario benchmark pairs (1,000 genes) used
    to evaluate all methods
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.isci.2020.101185
    title: 'BATMAN: Fast and Accurate Integration of Single-Cell RNA-Seq Datasets
      via Minimum-Weight Matching'
  source_content_hash: sha256:2b62d8a2889f01e67d0cb729602b8ab8bf1c647b3cf077e86b0816e2a362e67f
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
  - Seurat V3.0
  - Splatter
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.8421052631578947
n_claims: 19
---

# cross-method-integration-benchmarking

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark scRNA-seq batch-effect correction methods head-to-head on an identical preprocessed dataset pair, scoring batch mixing (mean iLISI) and local structure preservation (mean 50-RNN) against a no-correction baseline. Used in the BATMAN paper to show BATMAN retains local structure (50-RNN 0.92) where Seurat V3.0/MNNcorrect/Scanorama collapse (0.58/0.51/0.14).

## When to use

Use when two scRNA-seq batches must be integrated and several correction tools should be compared quantitatively on the same inputs using iLISI (batch mixing) and 50-RNN (local structure retention) against an uncorrected Original baseline.

## When NOT to use

- The input comprises more than two datasets — the current BATMAN implementation works only with two datasets, so multi-dataset integration cannot follow this skill as-is
- There is only a single batch (no batch effects to correct), so iLISI mixing gains and integration benchmarking are meaningless
- The analysis goal is cell-type clustering or trajectory inference rather than quantifying batch-correction quality and local structure retention

## Inputs

- Two scRNA-seq count matrices (simulated with Splatter at 1,000 genes per dataset, 100 datasets per scenario, or real datasets such as the Muraro CEL-Seq2 and Segerstolpe Smart-Seq2 pancreatic sets)
- Seurat V3.0 log-count matrices produced by shared pre-processing of each dataset
- No-correction (Original) concatenation of the two log-count matrices as baseline
- Metric parameters: neighborhood size k=50 for 50-RNN; iLISI computed over batches

## Outputs

- lb_integration_metrics.csv — table of mean iLISI and mean 50-RNN for Original, BATMAN, Seurat V3.0, MNNcorrect, and Scanorama
- Batch-corrected expression matrices from each integration method
- Comparison verdict: which method best balances batch mixing (iLISI) against local structure preservation (50-RNN)

## How to apply

Generate or select the dataset pair to be compared (in the paper, Splatter simulations of 100 datasets per scenario with 1,000 genes, e.g. the large batch effects 'LB' scenario). Preprocess every input identically with Seurat V3.0 pre-processing steps to obtain log-counts, and keep the uncorrected concatenation as the Original baseline. Run each method on the same preprocessed pair: BATMAN (build a weighted bipartite graph with edges weighted by Euclidean distance, find the minimum-weight matching via the exact blossom-based algorithm or the greedy approximation with ratio 2, determine local shifting vectors, apply batch correction) plus the comparison methods Seurat V3.0, MNNcorrect, and Scanorama. Compute mean iLISI (batch mixing) and mean 50-RNN (fraction of 50 nearest-neighbor relationships retained) for Original, BATMAN, Seurat V3.0, MNNcorrect, and Scanorama, and tabulate the two metrics across methods (e.g. lb_integration_metrics.csv). Judge success by whether corrected methods improve mixing without destroying local structure — in the LB scenario the reference values are mean iLISI 1.82/1.84/1.00/1.00/1.01 and mean 50-RNN 1.00/0.92/0.58/0.51/0.14 for Original/BATMAN/Seurat V3.0/MNNcorrect/Scanorama.

## Related tools

- **BATMAN** (Method under test; builds a weighted bipartite graph of anchors with Euclidean-distance edges, solves minimum-weight matching (exact blossom or greedy, ratio 2), and applies local shifting vectors for batch correction) — https://github.com/mandricigor/batman
- **Seurat V3.0** (Provides shared pre-processing to log-counts for all methods and serves as one of the three comparison integration methods)
- **MNNcorrect** (Comparison method (mutual nearest neighbor based) benchmarked on the same preprocessed pair)
- **Scanorama** (Comparison method benchmarked on the same preprocessed pair)
- **Splatter** (Simulates the 100-dataset-per-scenario benchmark pairs (1,000 genes) used to evaluate all methods)

## Examples

```
python3.7 batman.py -data sim_lb_dataset1.csv,sim_lb_dataset2.csv -output corrected_lb.csv -anchors 100
```

## Evaluation signals

- Reproduce the LB-scenario reference values: mean iLISI 1.82/1.84/1.00/1.00/1.01 for Original/BATMAN/Seurat V3.0/MNNcorrect/Scanorama (within reasonable simulation variance)
- Reproduce mean 50-RNN 1.00/0.92/0.58/0.51/0.14 for the same five methods
- BATMAN must preserve the highest 50-RNN among corrected methods while its iLISI stays above the Original baseline (1.84 vs 1.82)
- Seurat V3.0 and MNNcorrect iLISI should collapse toward 1.00 (near-total failure of batch mixing) while Scanorama reaches only 1.01 — a sanity check that the metric pipeline distinguishes methods
- Output CSV contains exactly five methods × two metrics, all computed on identically preprocessed log-count inputs

## Limitations

- Current BATMAN implementation supports integration of only two datasets; multi-dataset integration is possible 'in principle' but not implemented
- Greedy matching is only an approximation with ratio 2 (recommended for extra large datasets), so results may differ from the exact blossom-based matching
- Reference metric values come from Splatter simulations (100 datasets per scenario); real-data results will vary and should be compared within reasonable simulation/empirical variance

## Evidence

- [readme] BATMAN's graph construction step used within the benchmarked pipeline: [evidence span withheld — anchor status: external_doc]
