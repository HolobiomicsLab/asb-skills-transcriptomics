---
name: low-dimensional-embedding-construction
description: Use when two denoised, same-gene-set GRN adjacency matrices (e.g., sparse
  dgCMatrix networks X and Y from CP tensor decomposition) must be aligned into a
  shared low-dimensional space to compare each gene's regulatory positions across
  conditions.
when_to_use_negative:
- Input networks come from different gene sets or have not been denoised — the joint
  Laplacian requires symmetric Wx and Wy over the same gene set
- The goal is comparing highly dissimilar samples — scTenifoldNet is more suitable
  for comparing highly similar samples, such as two populations of cells of the same
  type
- A ranked gene list is needed without per-gene cross-condition coordinates — the
  downstream dRegulation step alone may suffice
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: scTenifoldNet
  role: R package whose manifoldAlignment function performs the non-linear manifold
    alignment of the two denoised networks
  repo: github.com/cailab-tamu/scTenifoldNet
- name: RSpectra
  role: Provides the sparse eigensolver used to solve the joint Laplacian eigenvalue
    problem for the embedding
- name: Rcpp
  role: Imported dependency supporting fast computation in the scTenifoldNet package
- name: Matrix
  role: Handles the sparse dgCMatrix adjacency matrices fed into the alignment
- name: igraph
  role: Optional conversion of the denoised sparse networks to igraph objects for
    downstream topological inspection
  repo: https://igraph.org/r/
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.patter.2020.100139
    title: 'scTenifoldNet: A Machine Learning Workflow for Constructing and Comparing
      Transcriptome-wide Gene Regulatory Networks from Single-Cell Data'
  source_content_hash: sha256:ec345c36ef9f5b6e3556bbd58f93f5d0b94c5a505f7dbad9d049f111d57119f6
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
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# low-dimensional-embedding-construction

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Constructs a shared low-dimensional embedding of two denoised single-cell gene regulatory networks (scGRNs) by solving an eigenvalue problem on a joint Laplacian, so each gene from both samples receives coordinates on a common manifold that can be directly compared. This is the manifold-alignment step (manifoldAlignment) of the scTenifoldNet workflow.

## When to use

Use when two denoised, same-gene-set GRN adjacency matrices (e.g., sparse dgCMatrix networks X and Y from CP tensor decomposition) must be aligned into a shared low-dimensional space to compare each gene's regulatory positions across conditions.

## When NOT to use

- Input networks come from different gene sets or have not been denoised — the joint Laplacian requires symmetric Wx and Wy over the same gene set
- The goal is comparing highly dissimilar samples — scTenifoldNet is more suitable for comparing highly similar samples, such as two populations of cells of the same type
- A ranked gene list is needed without per-gene cross-condition coordinates — the downstream dRegulation step alone may suffice

## Inputs

- Two denoised gene regulatory networks X and Y (sparse dgCMatrix adjacency matrices) produced by CP tensor decomposition
- Embedding dimension d (default d = 30, with km ≪ n)
- Off-diagonal coupling weight λ tying each gene in sample X to its counterpart in sample Y

## Outputs

- A data frame of low-dimensional features from the non-linear manifold alignment with 2 × n gene rows and d columns (default d = 30), giving each gene two km-dimensional coordinate vectors Fjx and Fjy
- Per-gene coordinates that feed the dRegulation differential regulation test (Euclidean distance, Box-Cox Z, FC, p.value, p.adj)

## How to apply

Load the two denoised networks X and Y output by the tensorDecomposition step (sparse dgCMatrix objects) and symmetrize each adjacency matrix into symmetric weight matrices Wx and Wy over the same gene set. Assemble the joint Laplacian W = [Wx, λI/2; λIᵀ/2, Wy], where the off-diagonal blocks tie each gene in sample X to its counterpart in sample Y with weight λ/2. Solve the sparse eigenvalue problem for W using RSpectra's eigensolver (function signature: manifoldAlignment(X, Y, d, nCores)), retaining the d = 30 lowest-frequency non-trivial eigenvectors. Format the eigenvectors as a data frame of low-dimensional features with 2 × n gene rows and d columns, distinguishing each gene's coordinates under sample X and sample Y (Fjx and Fjy); these coordinates feed the downstream dRegulation differential regulation test, which computes Euclidean distances between a gene's two embeddings followed by Box-Cox transformation and chi-square statistics.

## Related tools

- **scTenifoldNet** (R package whose manifoldAlignment function performs the non-linear manifold alignment of the two denoised networks) — github.com/cailab-tamu/scTenifoldNet
- **RSpectra** (Provides the sparse eigensolver used to solve the joint Laplacian eigenvalue problem for the embedding)
- **Rcpp** (Imported dependency supporting fast computation in the scTenifoldNet package)
- **Matrix** (Handles the sparse dgCMatrix adjacency matrices fed into the alignment)
- **igraph** (Optional conversion of the denoised sparse networks to igraph objects for downstream topological inspection) — https://igraph.org/r/

## Examples

```
MA <- scTenifoldNet::manifoldAlignment(X = outputHA$tensorNetworks$X, Y = outputHA$tensorNetworks$Y, d = 30)
```

## Evaluation signals

- Output data frame has exactly 2 × n gene rows and d = 30 columns
- Under the simulated perturbation (genes ng2/ng50/ng11 replaced by other genes' profiles), the manipulated genes (e.g., ng2, ng50, ng11) show the largest cross-condition coordinate distances
- Each gene appears twice in the embedding, once with coordinates under sample X (Fjx) and once under sample Y (Fjy), over the same gene set
- Wx and Wy are symmetric weight matrices before assembling the joint Laplacian W = [Wx, λI/2; λIᵀ/2, Wy]
- The retained eigenvectors are the d = 30 lowest-frequency non-trivial eigenvectors of the joint Laplacian

## Limitations

- scTenifoldNet is more suitable for comparing highly similar samples, such as two populations of cells of the same type, restricting its applicability
- Causal factors and their target genes remain unknown in many biological systems studied, limiting ground-truth validation of the aligned embedding
- Constructing scGRNs from scRNA-seq data consisting of cells in many different states, and controlling for technical noise, is challenging, which propagates into the alignment input

## Evidence

- [methods] The manifold alignment is performed by solving an eigenvalue problem with a joint Laplacian W=[Wdx, λI/2;λIT/2, Wdy], after which all genes from both samples are projected onto a shared low-dimensional manifold: [evidence span withheld — anchor status: not_found]
- [methods] Solve the sparse eigenvalue problem for the joint Laplacian using RSpectra's eigensolver, retaining the d = 30 lowest-frequency non-trivial eigenvectors: [evidence span withheld — anchor status: not_found]
