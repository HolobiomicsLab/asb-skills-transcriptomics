---
name: cross-sample-network-coordinate-comparison
description: Use when you already hold two denoised scGRNs (sparse dgCMatrix X and
  Y over the same gene set) and need commensurable per-gene coordinates to detect
  differential regulation between conditions.
when_to_use_negative:
- The two samples are highly dissimilar (e.g., different cell types) rather than two
  populations of the same cell type — the article states scTenifoldNet is more suitable
  for comparing highly similar samples
- Input is still raw count data or un-denoised PC-regression networks — manifold alignment
  must be applied after tensorDecomposition
- You only need a ranked gene list without network-structure comparison — the upstream
  dRegulation output alone would not exist without this step, but a simpler differential-expression
  analysis would not require network alignment
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: scTenifoldNet
  role: Provides the exported `manifoldAlignment` function (step 5 of the six-step
    pipeline) implementing the joint-Laplacian alignment of the two denoised networks
  repo: github.com/cailab-tamu/scTenifoldNet
- name: RSpectra
  role: Sparse eigensolver used to solve the joint Laplacian eigenvalue problem and
    retain the 30 lowest-frequency non-trivial eigenvectors
- name: Rcpp
  role: Imported dependency of scTenifoldNet supporting the compiled internals of
    the pipeline functions
- name: igraph
  role: Optional downstream conversion of the denoised sparse adjacency matrices for
    topological inspection before/after alignment
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.patter.2020.100139
    title: 'scTenifoldNet: A Machine Learning Workflow for Constructing and Comparing
      Transcriptome-wide Gene Regulatory Networks from Single-Cell Data'
  source_content_hash: sha256:021bebe03b839a46a2eccd8fcaaef7fc698e4fe6535be224b5f6bacf976134ac
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
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: critique
yes_rate: 0.5833333333333334
n_claims: 12
---

# cross-sample-network-coordinate-comparison

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Non-linear manifold alignment (scTenifoldNet's `manifoldAlignment` step) that projects two denoised single-cell gene regulatory networks onto a shared low-dimensional manifold via a joint-Laplacian eigenvalue problem, so each gene's regulatory position can be compared across samples.

## When to use

Use when you already hold two denoised scGRNs (sparse dgCMatrix X and Y over the same gene set) and need commensurable per-gene coordinates to detect differential regulation between conditions.

## When NOT to use

- The two samples are highly dissimilar (e.g., different cell types) rather than two populations of the same cell type — the article states scTenifoldNet is more suitable for comparing highly similar samples
- Input is still raw count data or un-denoised PC-regression networks — manifold alignment must be applied after tensorDecomposition
- You only need a ranked gene list without network-structure comparison — the upstream dRegulation output alone would not exist without this step, but a simpler differential-expression analysis would not require network alignment

## Inputs

- Denoised sample-X gene regulatory network as a sparse dgCMatrix (tensorNetworks$X)
- Denoised sample-Y gene regulatory network as a sparse dgCMatrix (tensorNetworks$Y)
- Embedding dimension d (default 30) and number of cores nCores

## Outputs

- Data frame of low-dimensional manifold-alignment features with 2 × n gene rows and d = 30 columns (per-gene coordinate vectors Fjx and Fjy)
- Per-gene Euclidean cross-condition distances consumed by dRegulation (distance, Z, FC, p.value, p.adj columns)

## How to apply

Load the two denoised networks X and Y from the CP tensor-decomposition step (`tensorNetworks$X`, `tensorNetworks$Y`) and symmetrize each adjacency matrix into symmetric weight matrices Wx and Wy over the same gene set. Assemble the joint Laplacian W = [Wx, λI/2; λIᵀ/2, Wy], where the off-diagonal blocks tie each gene in sample X to its counterpart in sample Y with weight λ/2. Solve the sparse eigenvalue problem for W with RSpectra's eigensolver, retaining the d = 30 lowest-frequency non-trivial eigenvectors via `manifoldAlignment(X, Y, d, nCores)`. Format the eigenvectors as a data frame of low-dimensional features with 2 × n gene rows and d columns, distinguishing each gene's coordinates under sample X (Fjx) and sample Y (Fjy); downstream `dRegulation` then computes Euclidean cross-condition distances per gene, Box-Cox-transforms them, and applies chi-square statistics.

## Related tools

- **scTenifoldNet** (Provides the exported `manifoldAlignment` function (step 5 of the six-step pipeline) implementing the joint-Laplacian alignment of the two denoised networks) — github.com/cailab-tamu/scTenifoldNet
- **RSpectra** (Sparse eigensolver used to solve the joint Laplacian eigenvalue problem and retain the 30 lowest-frequency non-trivial eigenvectors)
- **Rcpp** (Imported dependency of scTenifoldNet supporting the compiled internals of the pipeline functions)
- **igraph** (Optional downstream conversion of the denoised sparse adjacency matrices for topological inspection before/after alignment)

## Examples

```
mA <- manifoldAlignment(outputHA$tensorNetworks$X, outputHA$tensorNetworks$Y, d = 30)
```

## Evaluation signals

- Output data frame has exactly 2 × n gene rows and d = 30 columns (default dimensionality)
- Under the simulated perturbation (ng10←ng50, ng2←ng11, ng3←ng5), the manipulated genes ng2, ng50, ng11, ng3, ng10, ng5 show the largest cross-condition Euclidean distances (top six in diffRegulation)
- Input matrices are symmetric weight matrices over the same gene set, of class dgCMatrix, produced by tensorDecomposition
- Under H0 (same matrix supplied as both X and Y), all per-gene distances are near zero and no genes reach significance at FDR < 0.1
- diffRegulation output schema is complete: gene, distance, Z (post Box-Cox), FC, p.value (chi-square, 1 df), and p.adj (Benjamini & Hochberg FDR)

## Limitations

- scTenifoldNet is more suitable for comparing highly similar samples, such as two populations of cells of the same type, restricting its applicability
- Regulatory relationships between genes from scRNA-seq data are difficult to establish, and controlling for technical noise is difficult, so aligned coordinates inherit upstream construction uncertainty
- Causal factors and their target genes remain unknown in many biological systems studied, limiting ground-truth validation of the alignment

## Evidence

- [methods] Non-linear manifold alignment of the two denoised networks: "| 5 | `manifoldAlignment` | Non-linear manifold alignment of the two denoised networks |"
- [readme] manifoldAlignment output is a data frame of low-dimensional features with 2n rows and d = 30 columns: "**`manifoldAlignment`** — A data frame of low-dimensional features from the non-linear manifold alignment, with 2 × *n* genes rows and *d* columns (default *d* = 30)."
- [discussion] scTenifoldNet performs nonlinear manifold alignment to align two networks: "Fourth, scTenifoldNet performs nonlinear manifold alignment to align two networks."
