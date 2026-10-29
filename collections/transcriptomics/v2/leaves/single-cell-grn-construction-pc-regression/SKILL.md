---
name: single-cell-grn-construction-pc-regression
description: Use when reconstructing gene regulatory networks from raw scRNA-seq count
  matrices (genes as rows, cells as columns) and you need a GRN construction method
  with high specificity and sensitivity across a wide range of cell numbers (10 to
  3,000 cells).
when_to_use_negative:
- The goal is differential regulation testing or comparing two conditions end-to-end
  — use the full scTenifoldNet pipeline (tensorDecomposition, manifoldAlignment, dRegulation)
  instead of bare PC regression network construction
- The input is already normalized/processed rather than raw counts, unless you intentionally
  substitute custom preprocessing (scTenifoldNet's modular design permits this but
  the default expects raw counts)
- The samples being compared are highly dissimilar — scTenifoldNet is more suitable
  for comparing highly similar samples, such as two populations of cells of the same
  type
edam_operation: http://edamontology.org/operation_1781
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0602
tools:
- name: scTenifoldNet
  role: R package providing pcNet/makeNetworks for PC regression-based scGRN construction
    from subsampled cells
  repo: https://github.com/cailab-tamu/scTenifoldNet
- name: scTenifoldpy
  role: Python implementation of the scTenifold suite including plain GRN construction
  repo: https://github.com/qwerty239qwe/scTenifoldpy
- name: scGEAToolbox
  role: MATLAB implementation alternative of the scTenifold workflow
  repo: https://github.com/jamesjcai/scGEAToolbox
- name: R
  role: Runtime environment; scTenifoldNet installs via install.packages or remotes::install_github
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.patter.2020.100139
    title: 'scTenifoldNet: A Machine Learning Workflow for Constructing and Comparing
      Transcriptome-wide Gene Regulatory Networks from Single-Cell Data'
  source_content_hash: sha256:cb2e95064f2e12aadcdcfb110dda96329b06ec31614d275fe45481e153db92a6
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
  - scGEAToolbox
  - scTenifoldNet
  - scTenifoldpy
  edam_operation: http://edamontology.org/operation_1781
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# single-cell-grn-construction-pc-regression

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Construct single-cell gene regulatory networks (scGRNs) from raw scRNA-seq count matrices using principal-component (PC) regression via scTenifoldNet's pcNet/makeNetworks functions, benchmarked to outperform Spearman correlation, mutual information, and GENIE3 in specificity (accuracy) and sensitivity (recall).

## When to use

Use when reconstructing gene regulatory networks from raw scRNA-seq count matrices (genes as rows, cells as columns) and you need a GRN construction method with high specificity and sensitivity across a wide range of cell numbers (10 to 3,000 cells).

## When NOT to use

- The goal is differential regulation testing or comparing two conditions end-to-end — use the full scTenifoldNet pipeline (tensorDecomposition, manifoldAlignment, dRegulation) instead of bare PC regression network construction
- The input is already normalized/processed rather than raw counts, unless you intentionally substitute custom preprocessing (scTenifoldNet's modular design permits this but the default expects raw counts)
- The samples being compared are highly dissimilar — scTenifoldNet is more suitable for comparing highly similar samples, such as two populations of cells of the same type

## Inputs

- Raw scRNA-seq counts matrix (genes as rows, cells as columns; unnormalized for the main scTenifoldNet function)
- Ground-truth regulatory edges (e.g., SERGIO simulation ground truth) when benchmarking accuracy/recall

## Outputs

- Weighted gene regulatory network (adjacency/edge-weight matrix) inferred by PC regression
- Per-method accuracy (specificity) and recall (sensitivity) values against ground-truth edges, e.g., plotted versus number of cells (Figure 2A-style curves)

## How to apply

Start from a raw, unnormalized counts matrix with genes as rows and cells as columns; optionally run scQC (library size filtering, outlier cell removal, minimum gene expression fraction, mitochondrial read ratio) and cpmNormalization first. Randomly subsample cells, then apply principal-component regression via makeNetworks/pcNet (with parameters nComp, scaleScores, symmetric, q, priorNetwork, nCores, useRcpp) to infer weighted regulatory edges between genes. For benchmarking against ground truth (e.g., SERGIO-simulated data with known regulatory edges), reconstruct comparable networks with Spearman correlation, mutual information, and GENIE3, then compute accuracy (specificity) and recall (sensitivity) of each method's edges against the ground-truth regulatory interactions. Rationale: PC regression significantly outperformed the other GRN construction algorithms on specificity, sensitivity, computational efficiency, and the minimum number of cells required, holding across cell counts from 10 to 3,000 in simulations with 100 genes in two regulatory modules.

## Related tools

- **scTenifoldNet** (R package providing pcNet/makeNetworks for PC regression-based scGRN construction from subsampled cells) — https://github.com/cailab-tamu/scTenifoldNet
- **scTenifoldpy** (Python implementation of the scTenifold suite including plain GRN construction) — https://github.com/qwerty239qwe/scTenifoldpy
- **scGEAToolbox** (MATLAB implementation alternative of the scTenifold workflow) — https://github.com/jamesjcai/scGEAToolbox
- **R** (Runtime environment; scTenifoldNet installs via install.packages or remotes::install_github)

## Examples

```
library(scTenifoldNet); net <- makeNetworks(X, nNet = 10, nCells = 500, scaleScores = TRUE, symmetric = TRUE, q = 0.95)
```

## Evaluation signals

- PC regression accuracy (specificity) and recall (sensitivity) curves lie above the Spearman correlation, mutual information, and GENIE3 curves across cell-number settings (10 to 3,000 cells), reproducing the Figure 2A pattern
- Reconstructed network edges are scored against SERGIO ground-truth regulatory edges; specificity and sensitivity are computed per method per cell count
- Input matrices are raw counts with genes as rows and cells as columns, per the README input requirement
- pcNet is invoked with documented parameters (nComp, scaleScores, symmetric, q, priorNetwork, verbose, nCores, useRcpp) and the resulting network is a gene-by-gene weighted adjacency matrix
- Results are stable across a wide range of cell numbers, consistent with the reported finding that the advantage held across a wide range of cell numbers

## Limitations

- Constructing scGRNs from scRNA-seq data consisting of cells in many different states is challenging; regulatory relationships between genes are difficult to establish even from theoretically complete data
- It is difficult to control for technical noise in scRNA-seq data
- Causal factors and their target genes remain unknown in many biological systems, limiting ground-truth validation outside simulated data
- Running time is largely determined by the network construction step and scales with the number of cells and genes (e.g., 7,500 cells × 7,500 genes takes ~10 h and ~22.6 GB)

## Evidence
