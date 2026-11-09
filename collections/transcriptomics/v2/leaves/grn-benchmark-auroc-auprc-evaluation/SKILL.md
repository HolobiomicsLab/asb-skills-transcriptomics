---
name: grn-benchmark-auroc-auprc-evaluation
description: Use when comparing multiple GRN-inference algorithms on the same scRNA-seq
  input matrix (e.g., BEELINE GSD 19 genes x 2,000 cells or SERGIO-simulated 100-gene
  matrices) where ground-truth regulatory edges exist and AUROC/AUPRC/runtime ranking
  is the goal.
when_to_use_negative:
- No ground-truth regulatory network is available for the dataset, so AUROC/AUPRC
  cannot be computed and only relative internal validation is possible
- The goal is differential-regulation discovery between two conditions rather than
  comparing GRN-construction algorithms; use the full scTenifoldNet manifold-alignment
  workflow instead
- The input is normalized/processed expression rather than a raw counts matrix with
  genes as rows and cells as columns, breaking comparability with the uniform benchmark
  pipeline
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3308
tools:
- name: scTenifoldNet
  role: Provides the PC-regression GRN construction step (pcNet/makeNetworks) being
    benchmarked against other algorithms
  repo: https://github.com/cailab-tamu/scTenifoldNet
- name: R
  role: Implementation environment for scTenifoldNet and the benchmark scoring pipeline
- name: Rcpp
  role: Backend dependency of pcNet for fast PC-regression network construction during
    timing benchmarks
- name: RSpectra
  role: Backend dependency of pcNet used in the network-construction step under benchmark
provenance:
  source_task_ids:
  - task_001
  - task_002
  source_papers:
  - doi: 10.1016/j.patter.2020.100139
    title: 'scTenifoldNet: A Machine Learning Workflow for Constructing and Comparing
      Transcriptome-wide Gene Regulatory Networks from Single-Cell Data'
  source_content_hash: sha256:34c8a99246f3a13ed0fff8ea38bbef946e701bbf98ca3bbb3dddc0f88d6d4d5c
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
  - R
  - RSpectra
  - Rcpp
  - scTenifoldNet
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# grn-benchmark-auroc-auprc-evaluation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Uniformly benchmark single-cell GRN-construction methods (scTenifoldNet PC regression, PIDC, SCC, MI, GENIE3, etc.) by scoring predicted edge rankings against a ground-truth network with AUROC and AUPRC, and recording per-method computation time. In this article's benchmarks, PC regression outperformed the other GRN construction algorithms in specificity, sensitivity, and computational efficiency.

## When to use

Use when comparing multiple GRN-inference algorithms on the same scRNA-seq input matrix (e.g., BEELINE GSD 19 genes x 2,000 cells or SERGIO-simulated 100-gene matrices) where ground-truth regulatory edges exist and AUROC/AUPRC/runtime ranking is the goal.

## When NOT to use

- No ground-truth regulatory network is available for the dataset, so AUROC/AUPRC cannot be computed and only relative internal validation is possible
- The goal is differential-regulation discovery between two conditions rather than comparing GRN-construction algorithms; use the full scTenifoldNet manifold-alignment workflow instead
- The input is normalized/processed expression rather than a raw counts matrix with genes as rows and cells as columns, breaking comparability with the uniform benchmark pipeline

## Inputs

- Raw scRNA-seq counts matrix (genes as rows, cell barcodes as columns), e.g., BEELINE GSD reference dataset (19 genes x 2,000 cells)
- SERGIO-simulated count matrices (100 genes, 10 to 3,000 cells) with known ground-truth regulatory edges
- Ground-truth gene regulatory network (list of true regulatory edges) for each dataset

## Outputs

- benchmark_comparison.csv with per-method AUROC, AUPRC, and computation time on identical hardware
- Per-method edge-ranking lists scored against the ground-truth network
- Accuracy (specificity) and recall (sensitivity) curves versus number of cells (Figure 2A-style plot, e.g., figure_2a_accuracy_recall.png)

## How to apply

Load the reference dataset as a raw counts matrix with genes as rows and cell barcodes as columns (e.g., the BEELINE gonadal sex determination dataset of 19 genes x 2,000 cells). Construct each method's GRN on identical inputs: for scTenifoldNet use the makeNetworks/pcNet step (pcNet(X, nComp, scaleScores, symmetric, q, priorNetwork, verbose, nCores, useRcpp)) on randomly subsampled cells with nc_nNet = 10 networks of nc_nCells = 500 cells, and run competing algorithms (PIDC, Spearman correlation, mutual information, GENIE3) through the uniform BEELINE pipeline or equivalently uniform settings. Rank each method's edges by weight and score against the ground-truth network to compute AUROC and AUPRC, sweeping cell counts (10 to 3,000 cells in the SERGIO simulations) to assess sensitivity to sample size. Record per-method runtime under identical hardware and input settings. Compile an AUROC/AUPRC/runtime comparison table and, for the cell-count sweep, plot accuracy (specificity) and recall (sensitivity) versus number of cells, validating that the PC-regression curves lie above the SCC, MI, and GENIE3 curves.

## Related tools

- **scTenifoldNet** (Provides the PC-regression GRN construction step (pcNet/makeNetworks) being benchmarked against other algorithms) — https://github.com/cailab-tamu/scTenifoldNet
- **R** (Implementation environment for scTenifoldNet and the benchmark scoring pipeline)
- **Rcpp** (Backend dependency of pcNet for fast PC-regression network construction during timing benchmarks)
- **RSpectra** (Backend dependency of pcNet used in the network-construction step under benchmark)

## Examples

```
output <- scTenifoldNet(X = X, Y = X, nc_nNet = 10, nc_nCells = 500, td_K = 3, qc_minLibSize = 30)
```

## Evaluation signals

- PC-regression accuracy (specificity) and recall (sensitivity) curves lie above the Spearman correlation, mutual information, and GENIE3 curves across cell-count settings (Figure 2A validation)
- PC regression and PIDC rank at the top of the AUROC/AUPRC comparison table on the BEELINE GSD dataset
- AUROC/AUPRC values fall in [0,1] and are computed against the same ground-truth edge list for every method on the identical input matrix
- Computation times are recorded under identical hardware and input settings so runtime comparisons are meaningful
- Network construction uses the documented subsampling parameters (nc_nNet = 10, nc_nCells = 500) as in the package example

## Limitations

- scTenifoldNet is more suitable for comparing highly similar samples, such as two populations of cells of the same type
- Constructing scGRNs from scRNA-seq data consisting of cells in many different states is challenging, and it is difficult to control for technical noise in the data
- Causal factors and their target genes remain unknown in many biological systems studied, limiting ground-truth validation outside reference datasets

## Evidence
