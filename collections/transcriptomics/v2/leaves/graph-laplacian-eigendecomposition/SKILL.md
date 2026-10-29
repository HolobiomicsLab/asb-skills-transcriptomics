---
name: graph-laplacian-eigendecomposition
description: Use when two networks over the same node set (e.g., denoised scGRNs for
  samples X and Y) must be aligned so each node's coordinates can be compared across
  conditions.
when_to_use_negative:
- The two networks do not share the same gene set, so cross-condition counterpart
  ties in the off-diagonal λI blocks cannot be defined
- Networks are still noisy pre-tensor-decomposition adjacency matrices — the alignment
  expects the CP tensor-decomposition denoised networks
- Comparing highly dissimilar samples (e.g., different cell types) rather than two
  populations of the same type, which scTenifoldNet is more suitable for
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: scTenifoldNet
  role: Package whose manifoldAlignment function implements the joint-Laplacian eigendecomposition
    step of the workflow
  repo: github.com/cailab-tamu/scTenifoldNet
- name: RSpectra
  role: Sparse eigensolver used to retain the d = 30 lowest-frequency non-trivial
    eigenvectors of the joint Laplacian
- name: Rcpp
  role: Imported dependency supporting the fast implementation of the alignment/eigensolver
    code
- name: Matrix
  role: Provides the sparse dgCMatrix representation of the symmetrized networks Wx
    and Wy
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.patter.2020.100139
    title: 'scTenifoldNet: A Machine Learning Workflow for Constructing and Comparing
      Transcriptome-wide Gene Regulatory Networks from Single-Cell Data'
  source_content_hash: sha256:d2729f2b12aea70969776060e169fec4fb1feceb4d09b71a8663082d74d9dc4b
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
  - scTenifoldNet
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6875
n_claims: 16
---

# Graph Laplacian Eigendecomposition for Manifold Alignment

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Solves a sparse eigenvalue problem on a joint Laplacian built from two symmetrized gene regulatory networks so that genes from both samples are projected onto a shared low-dimensional manifold, enabling direct gene-by-gene comparison of regulatory positions.

## When to use

Use when two networks over the same node set (e.g., denoised scGRNs for samples X and Y) must be aligned so each node's coordinates can be compared across conditions.

## When NOT to use

- The two networks do not share the same gene set, so cross-condition counterpart ties in the off-diagonal λI blocks cannot be defined
- Networks are still noisy pre-tensor-decomposition adjacency matrices — the alignment expects the CP tensor-decomposition denoised networks
- Comparing highly dissimilar samples (e.g., different cell types) rather than two populations of the same type, which scTenifoldNet is more suitable for

## Inputs

- Denoised gene regulatory network for sample X (sparse dgCMatrix adjacency matrix)
- Denoised gene regulatory network for sample Y (sparse dgCMatrix adjacency matrix)
- Manifold dimension d (default d = 30)
- Regularization weight λ for the cross-sample identity blocks
- Number of cores (nCores) for the eigensolver

## Outputs

- Data frame of low-dimensional manifold features with 2 × n gene rows and d columns (each gene has two km-dimensional coordinate vectors, Fjx and Fjy)

## How to apply

Load the two denoised networks (sparse dgCMatrix objects) from the CP tensor decomposition step and symmetrize each adjacency matrix into symmetric weight matrices Wx and Wy over the same gene set. Assemble the joint Laplacian W = [Wx, λI/2; λIᵀ/2, Wy], where the off-diagonal blocks tie each gene in sample X to its counterpart in sample Y with weight λ/2. Solve the sparse eigenvalue problem with RSpectra's eigensolver (function signature: manifoldAlignment(X, Y, d, nCores)), retaining the d = 30 lowest-frequency non-trivial eigenvectors (km ≪ n). Format the eigenvectors as a data frame of low-dimensional features with 2 × n gene rows and d columns, distinguishing each gene's coordinates under sample X (Fjx) and sample Y (Fjy). Downstream differential regulation (dRegulation) then computes Euclidean distances between a gene's two coordinate vectors, transformed by Box-Cox and tested against a chi-square distribution with one degree of freedom.

## Related tools

- **scTenifoldNet** (Package whose manifoldAlignment function implements the joint-Laplacian eigendecomposition step of the workflow) — github.com/cailab-tamu/scTenifoldNet
- **RSpectra** (Sparse eigensolver used to retain the d = 30 lowest-frequency non-trivial eigenvectors of the joint Laplacian)
- **Rcpp** (Imported dependency supporting the fast implementation of the alignment/eigensolver code)
- **Matrix** (Provides the sparse dgCMatrix representation of the symmetrized networks Wx and Wy)

## Examples

```
FA <- manifoldAlignment(outputHA$tensorNetworks$X, outputHA$tensorNetworks$Y, d = 30, nCores = 1)
```

## Evaluation signals

- Output data frame has exactly 2n rows (n genes × 2 samples) and d = 30 columns
- Under the simulated perturbation (genes 10, 2, 3 replaced by profiles of genes 50, 11, 5), the manipulated genes (e.g., ng2, ng50, ng11) show the largest cross-condition coordinate distances
- Under H0 (same matrix supplied for both conditions), differential-regulation distances are near machine zero (~1e-15) and no genes reach FDR < 0.1
- Joint Laplacian is correctly assembled with symmetric Wx and Wy blocks and λ/2 off-diagonal ties between counterpart genes
- Retained eigenvectors are the lowest-frequency non-trivial ones from the sparse eigenproblem solved via RSpectra

## Limitations

- scTenifoldNet is more suitable for comparing highly similar samples, such as two populations of cells of the same type, restricting its applicability
- The final ranked-gene interpretation depends on ground truth; causal factors and their target genes remain unknown in many biological systems, limiting validation
- Manifold dimension d defaults to 30, which assumes km ≪ n; eigenproblem cost scales with the number of genes

## Evidence

- [results] alignment via eigenvalue problem: [evidence span withheld — anchor status: not_found]
