---
name: single-cell-rna-seq-count-simulation
description: Use when you need a synthetic sparse count matrix with a known perturbation
  (e.g., swapped genes) to validate whether a comparative GRN workflow detects true
  signal without spurious findings.
when_to_use_negative:
- The analysis goal is inference on real biological samples, not benchmarking — a
  simulated negative-binomial matrix with ~67% zeros will not capture real dropout,
  library-size, or biological variability structure
- You need a perturbation affecting regulatory rewiring rather than expression-profile
  swaps — this simulation only changes gene expression values (copying profiles between
  genes), not the network edge structure directly
- Very large gene/cell regimes are required — the README documents that network construction
  runtime scales steeply (e.g., 7,500 genes x 7,500 cells takes ~10 h and 22.6 GB),
  so simulation-based validation at that scale is costly
edam_operation: http://edamontology.org/operation_3566
edam_topics:
- http://edamontology.org/topic_2229
- http://edamontology.org/topic_0203
tools:
- name: scTenifoldNet
  role: 'Downstream method under test: consumes the simulated X and Y raw counts matrices
    via scTenifoldNet() to produce H0/HA diffRegulation tables'
  repo: https://github.com/cailab-tamu/scTenifoldNet
- name: R
  role: Language used for the negative binomial simulation (rnbinom), the gene-swap
    perturbation, and the scTenifoldNet calls
- name: igraph
  role: 'Optional downstream check: converts the denoised tensorNetworks sparse matrices
    to igraph objects for topological inspection'
provenance:
  source_task_ids:
  - task_004
  - task_001
  source_papers:
  - doi: 10.1016/j.patter.2020.100139
    title: 'scTenifoldNet: A Machine Learning Workflow for Constructing and Comparing
      Transcriptome-wide Gene Regulatory Networks from Single-Cell Data'
  source_content_hash: sha256:d64f5f3b1e52fb37fdfe8200c84419426e6cb43648b75305b4df83888a186d37
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
  - igraph
  - scTenifoldNet
  edam_operation: http://edamontology.org/operation_3566
  edam_topics:
  - http://edamontology.org/topic_2229
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# single-cell-rna-seq-count-simulation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Simulate a sparse raw scRNA-seq count matrix from a negative binomial distribution and engineer a known gene-expression perturbation, so that differential regulation methods like scTenifoldNet can be benchmarked against a controlled ground truth (H0 self-comparison vs HA perturbed comparison).

## When to use

Use when you need a synthetic sparse count matrix with a known perturbation (e.g., swapped genes) to validate whether a comparative GRN workflow detects true signal without spurious findings.

## When NOT to use

- The analysis goal is inference on real biological samples, not benchmarking — a simulated negative-binomial matrix with ~67% zeros will not capture real dropout, library-size, or biological variability structure
- You need a perturbation affecting regulatory rewiring rather than expression-profile swaps — this simulation only changes gene expression values (copying profiles between genes), not the network edge structure directly
- Very large gene/cell regimes are required — the README documents that network construction runtime scales steeply (e.g., 7,500 genes x 7,500 cells takes ~10 h and 22.6 GB), so simulation-based validation at that scale is costly

## Inputs

- Simulation parameters: number of cells (e.g., nCells = 2000), number of genes (e.g., nGenes = 100), negative binomial size (20) and probability (0.98), and a random seed (set.seed(1))
- Perturbation specification: source-to-target gene index mapping for expression swaps (gene 10 <- 50, gene 2 <- 11, gene 3 <- 5)

## Outputs

- X: a raw counts matrix of 100 genes x 2,000 cells with ~67% zeros, last ten genes named 'mt-1'...'mt-10'
- Y: a perturbed copy of X in which genes 10, 2, and 3 carry the expression profiles of genes 50, 11, and 5
- H0 and HA differential regulation tables (gene, distance, Z, FC, p.value, p.adj) suitable for FDR-based validation and export to diff_regulation_results.csv

## How to apply

Simulate a genes-by-cells raw counts matrix (e.g., 100 genes x 2,000 cells) with rnbinom using size = 20 and prob = 0.98, which yields roughly 67% zeros; prefix the last ten genes with 'mt-' to mimic mitochondrial genes. Build a perturbed copy Y by replacing genes 10, 2, and 3 with the expression profiles of genes 50, 11, and 5, so exactly six genes are involved in the perturbation. Run the downstream method twice: once under H0 supplying the same matrix for both conditions (X vs X) and once under HA (X vs Y), using identical parameters (nc_nNet = 10, nc_nCells = 500, td_K = 3, qc_minLibSize = 30). The simulated H0/HA pair provides a ground truth for ranking and FDR evaluation: under H0 no gene should reach significance (FDR < 0.1), and under HA exactly the six perturbed genes should be top-ranked.

## Related tools

- **scTenifoldNet** (Downstream method under test: consumes the simulated X and Y raw counts matrices via scTenifoldNet() to produce H0/HA diffRegulation tables) — https://github.com/cailab-tamu/scTenifoldNet
- **R** (Language used for the negative binomial simulation (rnbinom), the gene-swap perturbation, and the scTenifoldNet calls)
- **igraph** (Optional downstream check: converts the denoised tensorNetworks sparse matrices to igraph objects for topological inspection)

## Examples

```
set.seed(1); X <- matrix(rnbinom(100*2000, size=20, prob=0.98), ncol=2000); Y <- X; Y[10,]<-Y[50,]; Y[2,]<-Y[11,]; Y[3,]<-Y[5,]; scTenifoldNet(X=X, Y=Y, nc_nNet=10, nc_nCells=500, td_K=3, qc_minLibSize=30)
```

## Evaluation signals

- Under H0 (X vs X), no gene attains FDR < 0.1 — e.g., the top gene ng59 has p.adj = 0.6411916
- Under HA (X vs Y), exactly the six perturbed genes (ng2, ng50, ng11, ng3, ng10, ng5) rank at the top with p.adj < 0.1 (e.g., ng2 p.adj = 0.02414332)
- The simulated matrix has ~67% zeros and the last ten genes carry the 'mt-' prefix as specified
- QQ plots of the FC statistic against chi-square(1) quantiles show FDR < 0.1 genes highlighted in red, matching the readmeExample.png figure
- The combined H0 and HA diffRegulation tables export cleanly to diff_regulation_results.csv with all six columns (gene, distance, Z, FC, p.value, p.adj)

## Limitations

- Detection power was demonstrated only for a perturbation restricted to six genes (three swapped gene pairs) on a sparse 100-gene x 2,000-cell negative binomial dataset; larger or subtler perturbations are untested here
- scTenifoldNet is more suitable for comparing highly similar samples, such as two populations of cells of the same type, which constrains how far simulated HA conditions should deviate
- Results depend on the random seed (set.seed(1)) and subsampling (nc_nNet = 10 networks of nc_nCells = 500 cells), so exact p-values are not reproducible without fixing these

## Evidence
