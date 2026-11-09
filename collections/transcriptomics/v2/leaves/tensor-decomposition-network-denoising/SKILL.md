---
name: tensor-decomposition-network-denoising
description: Use when multiple PC-regression-derived scGRNs have been built from random
  cell subsamples (e.g., 10 networks x 500 cells) and edge-weight noise must be smoothed
  before downstream comparison or recall evaluation.
when_to_use_negative:
- Input is a single, already-denoised GRN with no replicate/subsampled layers to smooth
  across
- The goal is comparing highly dissimilar samples, since scTenifoldNet (including
  its denoising step) is designed for 'comparing highly similar samples, such as two
  populations of cells of the same type'
- Input is normalized or QC-processed expression data rather than the raw counts matrix
  expected by the scTenifoldNet pipeline
edam_operation: null
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: scTenifoldNet
  role: R package supplying the tensorDecomposition function (step 4 of the six-step
    pipeline) for CP tensor denoising of multilayer scGRNs built by pcNet/makeNetworks
  repo: github.com/cailab-tamu/scTenifoldNet
- name: Rcpp
  role: R dependency (Imports) used for efficient computation in the scTenifoldNet
    implementation, including the low-rank tensor approximation
- name: RSpectra
  role: R dependency (Imports) used for spectral computations underlying the CP tensor
    decomposition
- name: igraph
  role: Converts the denoised sparse dgCMatrix tensor networks into igraph objects
    for downstream topological analysis
provenance:
  source_task_ids:
  - task_004
  - task_003
  source_papers:
  - doi: 10.1016/j.patter.2020.100139
    title: 'scTenifoldNet: A Machine Learning Workflow for Constructing and Comparing
      Transcriptome-wide Gene Regulatory Networks from Single-Cell Data'
  source_content_hash: sha256:ab1f9b6a5260d23650b0049bfddba0a15f92bf67afcac5576e0428f91dc44505
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/cailab-tamu/scTenifoldNet
  tools:
  - R
  - RSpectra
  - Rcpp
  - igraph
  - scTenifoldNet
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

# tensor-decomposition-network-denoising

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Denoises multilayer single-cell gene regulatory networks (scGRNs) built by PC regression from subsampled scRNA-seq data using CANDECOMP/PARAFAC (CP) low-rank tensor decomposition, then recovers regulatory relationships with ~25% higher recall than the individual noisy networks.

## When to use

Use when multiple PC-regression-derived scGRNs have been built from random cell subsamples (e.g., 10 networks x 500 cells) and edge-weight noise must be smoothed before downstream comparison or recall evaluation.

## When NOT to use

- Input is a single, already-denoised GRN with no replicate/subsampled layers to smooth across
- The goal is comparing highly dissimilar samples, since scTenifoldNet (including its denoising step) is designed for 'comparing highly similar samples, such as two populations of cells of the same type'
- Input is normalized or QC-processed expression data rather than the raw counts matrix expected by the scTenifoldNet pipeline

## Inputs

- Multilayer scGRN: a stack of gene-by-gene regulatory networks (e.g., 10 networks) inferred by principal-component regression from random cell subsamples (e.g., 500 cells each) of a raw scRNA-seq count matrix (genes as rows, cells as columns)
- Rank parameter td_K (default K = 3 in the documented example)
- Ground-truth simulated regulatory relationships (for recall-based validation)

## Outputs

- tensorNetworks: weight-averaged denoised gene regulatory networks after CP tensor decomposition, stored as sparse dgCMatrix matrices (one per condition, e.g., X and Y)
- Recall comparison table (pre- vs post-denoising recall of ground-truth edges, e.g., recall_improvement.csv)
- Sparse network objects exportable to igraph via graph_from_adjacency_matrix for downstream topological analysis

## How to apply

Build a stack of gene regulatory networks from repeatedly subsampled cells (e.g., 10 random draws of 500 cells from a 2,000-cell x 100-gene negative binomial count matrix, size = 20, prob = 0.98) using scTenifoldNet's makeNetworks/pcNet, producing a multilayer scGRN. Apply the tensorDecomposition function with rank td_K = 3 (CANDECOMP/PARAFAC) to smooth edge weights across all networks in the stack. The function returns weight-averaged denoised tensorNetworks as sparse dgCMatrix matrices, one per input condition. Evaluate denoising by computing recall of known (simulated ground-truth) regulatory relationships before versus after denoising; the CP-denoised consensus network should recover the perturbed relationships (e.g., genes 10, 2, 3 carrying profiles of genes 50, 11, 5) among top hits. Denoising works because tensor decomposition identifies low-rank structure shared across the independently subsampled networks, effectively smoothing edge weights across all layers.

## Related tools

- **scTenifoldNet** (R package supplying the tensorDecomposition function (step 4 of the six-step pipeline) for CP tensor denoising of multilayer scGRNs built by pcNet/makeNetworks) — github.com/cailab-tamu/scTenifoldNet
- **Rcpp** (R dependency (Imports) used for efficient computation in the scTenifoldNet implementation, including the low-rank tensor approximation)
- **RSpectra** (R dependency (Imports) used for spectral computations underlying the CP tensor decomposition)
- **igraph** (Converts the denoised sparse dgCMatrix tensor networks into igraph objects for downstream topological analysis)

## Examples

```
outputHA <- scTenifoldNet(X = X, Y = Y, nc_nNet = 10, nc_nCells = 500, td_K = 3, qc_minLibSize = 30)
```

## Evaluation signals

- Recall of ground-truth regulatory relationships in the CP-denoised consensus network exceeds the average recall of the individual pre-denoising networks by ~25% (accept a stochastic band such as 15-35%, matching Figure 2B)
- The denoised output object contains weight-averaged tensorNetworks as sparse matrices of class dgCMatrix for each condition (X and Y)
- Known perturbation relationships (genes 10, 2, 3 carrying the profiles of genes 50, 11, 5 in the simulation) are recovered among the top hits after denoising
- Running tensorDecomposition with td_K = 3 on the 10-network stack completes without error and the result converts cleanly to an igraph object via graph_from_adjacency_matrix

## Limitations

- Denoising operates within scTenifoldNet, whose overall applicability is restricted: it is 'more suitable for comparing highly similar samples, such as two populations of cells of the same type'
- Validation of recovery depends on ground truth, but 'causal factors and their target genes remain unknown in many biological systems studied'
- Recall improvement is stochastic; individual runs vary, so the ~25% gain should be judged within a tolerance band rather than exactly

## Evidence
