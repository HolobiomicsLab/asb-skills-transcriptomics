---
name: single-cell-expression-matrix-preprocessing
description: Use when a raw (genes x cells) scRNAseq count matrix (e.g. PBMC or Tabula
  Sapiens) must be prepared for q-diffused kernel construction, qPHATE, NMF/GEP extraction,
  or LDS, and the analysis goal requires retaining the full gene set.
when_to_use_negative:
- The input matrix is already log-normalized or preprocessed by another pipeline —
  re-running preprocess_cell_counts would double-transform
- The analysis requires batch-effect correction or alignment of disparate datasets,
  which the article explicitly did not consider within q-diffusion
- A marker-gene or reduced-dimension (e.g. PCA/HVG-selected) representation is the
  required end product, since this skill's whole purpose is to avoid discarding gene
  interactions
tools:
- name: q-Diffusion.jl
  role: Provides preprocess_cell_counts, estimate_inner_bandwidth, and the kernel-building
    types (DeformedKernel, AdaptiveKernel) that consume the preprocessed matrix
  repo: github.com/marmarelis/QDiffusion.jl
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s42003-024-06104-w
    title: q-Diffusion leverages the full dimensionality of gene coexpression in single-cell
      transcriptomics
  source_content_hash: sha256:05af4bb357ef483c3b97222778d22ee3455bbbd53bdb52c95493a2042b1b6fa1
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - q-Diffusion.jl
  - q-diffusion
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# single-cell-expression-matrix-preprocessing

## Summary

Prepare a raw scRNAseq count matrix for full-dimensional q-diffusion analysis by log-normalizing it via QDiffusion.jl's preprocess_cell_counts, deliberately skipping feature selection and dimensionality reduction so that all orders of gene-interaction terms survive into the q-diffused kernel.

## When to use

Use when a raw (genes x cells) scRNAseq count matrix (e.g. PBMC or Tabula Sapiens) must be prepared for q-diffused kernel construction, qPHATE, NMF/GEP extraction, or LDS, and the analysis goal requires retaining the full gene set.

## When NOT to use

- The input matrix is already log-normalized or preprocessed by another pipeline — re-running preprocess_cell_counts would double-transform
- The analysis requires batch-effect correction or alignment of disparate datasets, which the article explicitly did not consider within q-diffusion
- A marker-gene or reduced-dimension (e.g. PCA/HVG-selected) representation is the required end product, since this skill's whole purpose is to avoid discarding gene interactions

## Inputs

- Raw scRNAseq count matrix of shape (genes x cells), e.g. from the PBMC (IFN-gamma stimulation) dataset or Tabula Sapiens
- Optional: list of CUDA device IDs for GPU acceleration

## Outputs

- Named tuple from preprocess_cell_counts containing different versions of the matrix
- Log-normalized (genes x cells) matrix with no feature selection or dimensionality reduction
- Scalar inner_bandwidth from estimate_inner_bandwidth (median quantile of per-gene scatters)

## How to apply

Start with a (genes x cells) matrix called data and pass it through preprocess_cell_counts, which returns a named tuple of matrix versions; select the *.normalized element, which is log-normalized but — per the study's own pipeline — has no feature selection and no dimensionality reduction applied. Then estimate the inner bandwidth with estimate_inner_bandwidth(data), which computes average pairwise squared distances per gene and takes the median (50%) quantile of gene scatters as a multiscale heuristic for which gene expressions to take seriously. If proceeding to kernel construction, feed the log-normalized matrix into a DeformedKernel with a deformation parameter q>1 (e.g. q=1.2, so alpha=q-1) and the estimated inner bandwidth, wrapped in an AdaptiveKernel that scales bandwidth to the k-th nearest neighbor (k=16, following the PHATE adaptive-resolution scheme) and computes in log space (in_log=true) for numerical stability. Use the cuda_devices argument to parallelize cell-by-cell, gene-to-gene computations when GPUs are available; omit it for a CPU-bound method.

## Related tools

- **q-Diffusion.jl** (Provides preprocess_cell_counts, estimate_inner_bandwidth, and the kernel-building types (DeformedKernel, AdaptiveKernel) that consume the preprocessed matrix) — github.com/marmarelis/QDiffusion.jl

## Examples

```
data = preprocess_cell_counts(raw_counts).normalized; inner_bandwidth = estimate_inner_bandwidth(data)
```

## Evaluation signals

- preprocess_cell_counts returns a named tuple and the .normalized element preserves the full gene count (no HVG subsetting or PCA applied)
- estimate_inner_bandwidth returns a positive scalar approximately equal to the median per-gene pairwise squared distance
- Downstream q-diffused kernel values lie in (0,1] as expected for a q-exponential kernel
- Resulting affinity matrix is symmetric and n x n matching the cell count
- Kernel construction runs stably when in_log=true (no numerical overflow in the adaptive outer bandwidth)

## Limitations

- No batch-effect correction or alignment of disparate datasets was considered by the q-diffusion authors
- No algorithm is provided for selecting the deformation parameter q and inner bandwidth phi beyond the median-scatter heuristic
- Runtime of evaluating the q-diffused kernel across all cell pairs is suboptimal (20 minutes to 3 hours for one affinity matrix)
- q-diffusion makes parametric assumptions on the data that might be restrictive in some use cases

## Evidence

- [readme] preprocess_cell_counts returns a named tuple with log-normalized matrix: "There is also a convenience mathod called `preprocess_cell_counts(data)` that returns a named tuple containing different versions of the matrix"
- [readme] normalized element is log-normalized without feature selection: "The `*.normalized` element is simply log-normalized, sans feature selection or dimensionality reduction, like we do in our study."
- [methods] inner bandwidth estimation from the data matrix: "inner_bandwidth = estimate_inner_bandwidth(data)"
- [results] kernel evaluation and symmetrization into undirected graph: "the q-diffused kernel was evaluated between all pairs of cells, and then the adjacency matrix was symmetrified to produce a weighted, undirected graph of the cells"
- [discussion] q and phi are the vital parameters lacking a selection algorithm: "Such an exploration would invite a careful selection algorithm for non-Euclidean deformation q and inner bandwidth φ, the two vital parameters to q-diffusion."
