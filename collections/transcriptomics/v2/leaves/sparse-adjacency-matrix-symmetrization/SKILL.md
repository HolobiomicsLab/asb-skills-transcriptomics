---
name: sparse-adjacency-matrix-symmetrization
description: Use when two directed or asymmetric weighted GRN adjacency matrices must
  be converted to symmetric form before spectral manifold alignment or eigen-decomposition
  of a joint Laplacian.
when_to_use_negative:
- The matrices are already symmetric undirected adjacency weights (e.g., previously
  symmetrized Wx/Wy or an undirected igraph graph converted back to a matrix)
- The input is a raw counts matrix or normalized expression matrix rather than a denoised
  network adjacency matrix — symmetrization applies to the GRN layer, not the expression
  layer
- The downstream analysis is a directed-network method (e.g., directed regulatory-edge
  inference) where edge directionality is meaningful and should not be collapsed
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0085
- http://edamontology.org/topic_2258
tools:
- name: scTenifoldNet
  role: R package whose tensorDecomposition output (dgCMatrix networks X and Y) is
    symmetrized and whose manifoldAlignment function consumes the symmetric weight
    matrices
  repo: github.com/cailab-tamu/scTenifoldNet
- name: Matrix
  role: Provides the sparse dgCMatrix class holding the denoised adjacency matrices
    that are symmetrized
- name: igraph
  role: 'Optional conversion target: the denoised sparse networks can be converted
    to igraph objects for downstream topological analysis'
  repo: https://igraph.org/r/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.patter.2020.100139
    title: 'scTenifoldNet: A Machine Learning Workflow for Constructing and Comparing
      Transcriptome-wide Gene Regulatory Networks from Single-Cell Data'
  source_content_hash: sha256:18b0ff42ff36b91db1ccb59d878ca7ab03e0e4c9e3a891c3fec0bb4724a02515
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/cailab-tamu/scTenifoldNet
  tools:
  - Matrix
  - R
  - RSpectra
  - Rcpp
  - igraph
  - scTenifoldNet
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0085
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# Sparse adjacency matrix symmetrization

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Symmetrize the two denoised single-cell gene regulatory networks (sparse dgCMatrix objects X and Y from CP tensor decomposition) into symmetric weight matrices Wx and Wy over the same gene set, so they can be assembled into the joint Laplacian used for nonlinear manifold alignment in scTenifoldNet.

## When to use

Use when two directed or asymmetric weighted GRN adjacency matrices must be converted to symmetric form before spectral manifold alignment or eigen-decomposition of a joint Laplacian.

## When NOT to use

- The matrices are already symmetric undirected adjacency weights (e.g., previously symmetrized Wx/Wy or an undirected igraph graph converted back to a matrix)
- The input is a raw counts matrix or normalized expression matrix rather than a denoised network adjacency matrix — symmetrization applies to the GRN layer, not the expression layer
- The downstream analysis is a directed-network method (e.g., directed regulatory-edge inference) where edge directionality is meaningful and should not be collapsed

## Inputs

- Denoised gene regulatory network X for sample X (sparse matrix of class dgCMatrix)
- Denoised gene regulatory network Y for sample Y (sparse matrix of class dgCMatrix)
- Both matrices defined over the same gene set with matched gene ordering

## Outputs

- Symmetric weight matrix Wx over the shared gene set
- Symmetric weight matrix Wy over the shared gene set
- Symmetric matrices ready for assembly into the joint Laplacian W = [Wx, λI/2; λIᵀ/2, Wy]

## How to apply

Load the denoised networks X and Y (sparse dgCMatrix objects) produced by the tensorDecomposition (CANDECOMP/PARAFAC) step of scTenifoldNet. Symmetrize each adjacency matrix so that Wx and Wy are symmetric weight matrices defined over the same gene set — this is a prerequisite because the subsequent joint Laplacian W = [Wx, λI/2; λIᵀ/2, Wy] and the RSpectra sparse eigensolver assume symmetric structure. Keep the matrices sparse throughout to preserve memory efficiency (the converted networks are also usable as igraph objects for downstream topological analysis). After symmetrization, pass Wx and Wy into the manifoldAlignment step, which solves for the d = 30 lowest-frequency non-trivial eigenvectors and projects each gene into a shared low-dimensional manifold. Confirm correctness by verifying matrix symmetry (W equals its transpose) and that both matrices share identical row/column gene ordering before assembling the joint Laplacian.

## Related tools

- **scTenifoldNet** (R package whose tensorDecomposition output (dgCMatrix networks X and Y) is symmetrized and whose manifoldAlignment function consumes the symmetric weight matrices) — github.com/cailab-tamu/scTenifoldNet
- **Matrix** (Provides the sparse dgCMatrix class holding the denoised adjacency matrices that are symmetrized)
- **igraph** (Optional conversion target: the denoised sparse networks can be converted to igraph objects for downstream topological analysis) — https://igraph.org/r/

## Examples

```
Wx <- (X + Matrix::t(X)) / 2; Wy <- (Y + Matrix::t(Y)) / 2; FA <- manifoldAlignment(Wx, Wy, d = 30)
```

## Evaluation signals

- Wx and Wy each equal their own transpose (symmetry check on the sparse matrices)
- Wx and Wy share identical rownames/colnames and gene ordering so the off-diagonal identity blocks of the joint Laplacian tie each gene in sample X to its counterpart in sample Y
- Matrices remain sparse (dgCMatrix class) after symmetrization, consistent with the README's memory benchmarks
- Downstream manifoldAlignment output has 2n rows and d = 30 columns, confirming the symmetrized matrices fed a valid eigendecomposition
- Under the simulated perturbation (genes ng2, ng50, ng11 swapped), the manipulated genes show the largest cross-condition coordinate distances

## Limitations

- Symmetrization discards edge directionality, so directed regulatory relationships from the PC-regression-constructed networks are collapsed into undirected weights
- scTenifoldNet is more suitable for comparing highly similar samples (e.g., two populations of cells of the same type), which constrains the interpretability of the symmetrized alignment for divergent samples
- Regulatory relationships and their directionality are difficult to establish from noisy, sparse scRNA-seq data, so the symmetrized weights remain statistical estimates

## Evidence

- [methods] Symmetrize each adjacency matrix so that Wx and Wy are symmetric weight matrices over the same gene set.: [evidence span withheld — anchor status: not_found]
- [methods] Assemble the joint Laplacian W = [Wx, λI/2; λIᵀ/2, Wy], where the off-diagonal blocks tie each gene in sample X to its counterpart in sample Y with weight λ/2.: [evidence span withheld — anchor status: not_found]
