---
name: cross-method-runtime-comparison
description: Use when a study must rank GRN-inference algorithms fairly on one dataset
  — e.g., AUROC/AUPRC vs. a reference network plus per-method runtime on identical
  hardware.
when_to_use_negative:
- The input is not a raw counts matrix (e.g., already normalized or dimensionality-reduced
  data), since scTenifoldNet expects unnormalized counts with genes as rows and cells
  as columns
- The goal is a single method's differential-regulation analysis rather than a fair
  cross-method benchmark on one shared dataset
- No ground-truth network is available, so AUROC/AUPRC against a reference cannot
  be computed
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3308
tools:
- name: scTenifoldNet
  role: Benchmarked GRN-construction method whose makeNetworks/pcNet step (PC regression)
    produces the predicted edge ranking; its documented running-time scaling informs
    the runtime benchmark
  repo: https://github.com/cailab-tamu/scTenifoldNet
- name: R
  role: Language in which the benchmarked pipeline and competing-algorithm wrappers
    are executed
- name: Rcpp
  role: Imported dependency enabling the fast pcNet network-construction implementation
    that determines scTenifoldNet's runtime in the comparison
- name: RSpectra
  role: Imported dependency for the spectral computations underlying the PC-regression
    network construction
- name: Matrix
  role: Sparse-matrix handling (dgCMatrix) for the constructed GRNs fed into scoring
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.patter.2020.100139
    title: 'scTenifoldNet: A Machine Learning Workflow for Constructing and Comparing
      Transcriptome-wide Gene Regulatory Networks from Single-Cell Data'
  source_content_hash: sha256:41801591969119e8793d4cd4b4b111c7e6f7d5f35d0134d4d35f01626678f182
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/cailab-tamu/scTenifoldNet
  repo_url: https://github.com/cailab-tamu/scTenifoldNet
  tools:
  - Matrix
  - R
  - RSpectra
  - Rcpp
  - scTenifoldNet
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# cross-method-runtime-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Benchmark multiple GRN-inference algorithms (including scTenifoldNet's PC regression) uniformly on the same raw scRNA-seq count matrix, scoring each method's predicted edge ranking against a ground-truth network (AUROC, AUPRC) and recording per-method computation time under identical hardware and input settings.

## When to use

Use when a study must rank GRN-inference algorithms fairly on one dataset — e.g., AUROC/AUPRC vs. a reference network plus per-method runtime on identical hardware.

## When NOT to use

- The input is not a raw counts matrix (e.g., already normalized or dimensionality-reduced data), since scTenifoldNet expects unnormalized counts with genes as rows and cells as columns
- The goal is a single method's differential-regulation analysis rather than a fair cross-method benchmark on one shared dataset
- No ground-truth network is available, so AUROC/AUPRC against a reference cannot be computed

## Inputs

- Raw counts matrix of the reference dataset (e.g., BEELINE GSD: 19 genes x 2,000 cells, genes as rows, cell barcodes as columns)
- Ground-truth regulatory network for the reference dataset (e.g., BEELINE GSD ground truth)
- Implementations of the competing GRN-inference algorithms (11 methods, including scTenifoldNet/PC regression and PIDC)

## Outputs

- Per-method AUROC and AUPRC computed against the ground-truth network
- Per-method computation time under identical hardware and input settings
- Comparison table saved as benchmark_comparison.csv

## How to apply

Load the reference dataset (e.g., the BEELINE gonadal sex determination matrix of 19 genes x 2,000 cells, genes as rows and cell barcodes as columns) as a raw counts matrix. Run each method — including scTenifoldNet's network construction via pcNet(X, nComp, scaleScores, symmetric, q, priorNetwork, verbose, nCores, useRcpp) with nc_nNet = 10 subsampled networks of nc_nCells = 500 cells — and the competing algorithms through a uniform pipeline on the same input. Score each method's predicted edge ranking against the ground-truth network to compute AUROC and AUPRC. Record per-method computation time under identical hardware and input settings, since running time is largely determined by the network-construction step and scales with the number of cells and genes. Compile the AUROC/AUPRC/runtime comparison table and verify expected rankings (e.g., PC regression and PIDC at the top) before saving it as benchmark_comparison.csv.

## Related tools

- **scTenifoldNet** (Benchmarked GRN-construction method whose makeNetworks/pcNet step (PC regression) produces the predicted edge ranking; its documented running-time scaling informs the runtime benchmark) — https://github.com/cailab-tamu/scTenifoldNet
- **R** (Language in which the benchmarked pipeline and competing-algorithm wrappers are executed)
- **Rcpp** (Imported dependency enabling the fast pcNet network-construction implementation that determines scTenifoldNet's runtime in the comparison)
- **RSpectra** (Imported dependency for the spectral computations underlying the PC-regression network construction)
- **Matrix** (Sparse-matrix handling (dgCMatrix) for the constructed GRNs fed into scoring)

## Examples

```
outputH0 <- scTenifoldNet(X = X, Y = X, nc_nNet = 10, nc_nCells = 500, td_K = 3, qc_minLibSize = 30)
```

## Evaluation signals

- Each method's predicted edge list is scored against the same GSD ground-truth network, yielding per-method AUROC and AUPRC values
- Per-method computation times are recorded under identical hardware and input settings and are directly comparable in one table
- The comparison confirms PC regression and PIDC rank at the top of the AUROC/AUPRC table before the result is accepted
- scTenifoldNet settings match the package example (nc_nNet = 10 subsampled networks of nc_nCells = 500 cells)
- Final table is written as benchmark_comparison.csv with one row per method and columns for AUROC, AUPRC, and runtime

## Limitations

- Running time is dominated by the network-construction step and scales steeply with cells and genes (e.g., 7,500 cells x 7,500 genes takes about 10 h 16 min and 22.6 GB of memory)
- Benchmark rankings depend on the chosen ground-truth dataset; causal factors and target genes remain unknown in many biological systems, limiting ground-truth validation
- Runtime comparisons are only valid under identical hardware and input settings; changing subsampling parameters (nc_nNet, nc_nCells) changes both accuracy and time

## Evidence

- [discussion] PC regression significantly outperforms the other GRN construction algorithms in all aspects of methodology metrics: "PC regression significantly outperforms the other GRN construction algorithms in all aspects of methodology metrics"
- [readme] Running time is largely determined by the network construction step and scales with the number of cells and genes: "Running time is largely determined by the network construction step and scales with the number of cells and genes."
- [methods] pcNet call signature: "- `pcNet(X, nComp, scaleScores, symmetric, q, priorNetwork, verbose, nCores, useRcpp)`"
- [readme] makeNetworks step description: "| 3 | `makeNetworks` | Constructs gene regulatory networks from subsampled cells using principal component regression (`pcNet`) |"
