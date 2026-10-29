---
name: row-stochastic-normalization-of-cell-cell-kernels
description: Use when a single-cell gene expression matrix must be filtered by projecting
  it onto a cell-cell similarity kernel derived from an annotated attribute (cell
  type, spatial coordinates, cell cycle phase), before subtracting the projection.
when_to_use_negative:
- The goal is integration/correction in a joint latent representation (e.g., scVI-style)
  rather than correction at the gene expression level
- No reliable prior knowledge is available to define the cell-cell similarity kernel
  — an inaccurate or coarse-grained mapping may lead to a loss of signal
edam_operation: http://edamontology.org/operation_2928
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: SiFT (sift-sc)
  role: Python package providing the kernel classes (MappingKernel, KnnKernel, RBFKernel,
    LaplacianKernel) and the SiFT/sifter interface used to normalize the kernel and
    compute the filtered matrix
  repo: https://github.com/nitzanlab/sift-sc
- name: pip
  role: Installation route for the sift-sc package
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-44757-7
    title: 'SiFT: uncovering hidden biological processes by probabilistic filtering
      of single-cell data'
  source_content_hash: sha256:31ed5cd4631d5cf3a3bec625b1323c7c4b338bc423b4c0d375d705e206b269b3
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nitzanlab/sift-sc
  tools:
  - SiFT
  - SiFT (sift-sc)
  - pip
  edam_operation: http://edamontology.org/operation_2928
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# row-stochastic-normalization-of-cell-cell-kernels

## Summary

Constructs and normalizes a cell-cell similarity kernel so each row sums to 1, making it a probabilistic operator that can project a gene expression count matrix onto a biological signal for SiFT-style filtering (X~ = X − K·X).

## When to use

Use when a single-cell gene expression matrix must be filtered by projecting it onto a cell-cell similarity kernel derived from an annotated attribute (cell type, spatial coordinates, cell cycle phase), before subtracting the projection.

## When NOT to use

- The goal is integration/correction in a joint latent representation (e.g., scVI-style) rather than correction at the gene expression level
- No reliable prior knowledge is available to define the cell-cell similarity kernel — an inaccurate or coarse-grained mapping may lead to a loss of signal

## Inputs

- single-cell gene expression count matrix (e.g., AnnData .h5ad)
- cell attribute annotation encoding the signal to be filtered (e.g., cell type, cell cycle score, spatial coordinates)

## Outputs

- row-stochastic cell-cell similarity kernel K (rows sum to 1)
- projection of the expression matrix onto the signal (P = K·X)
- filtered expression matrix X~ = X − K·X (e.g., written to filtered_expression.h5ad)

## How to apply

Choose the kernel variant that matches the signal encoding: sift.kernels.MappingKernel when the signal is a cell attribute mapping (e.g., cell type labels), KnnKernel for neighborhood-based similarity, or RBFKernel/LaplacianKernel for distance-based similarity (e.g., from spatial coordinates). Normalize the kernel row-stochastically so each row sums to 1, making K a probabilistic averaging operator whose application to X yields the expected expression of each cell's neighbors with respect to the signal. Compute the projection P = K·X by matrix multiplication, then subtract it from the original counts to get the filtered matrix X~ = X − K·X via the sift.SiFT / sift.sifter interface. The row-stochastic property is what makes the projection interpretable as a weighted average over signal-similar cells, which the paper grounds in computing a 'probabilistic cell-cell similarity kernel'.

## Related tools

- **SiFT (sift-sc)** (Python package providing the kernel classes (MappingKernel, KnnKernel, RBFKernel, LaplacianKernel) and the SiFT/sifter interface used to normalize the kernel and compute the filtered matrix) — https://github.com/nitzanlab/sift-sc
- **pip** (Installation route for the sift-sc package)

## Examples

```
pip install sift-sc; import sift; sft = sift.SiFT(data=adata, kernel=sift.kernels.RBFKernel()); sft.fit(); sft.transform()
```

## Evaluation signals

- Total variance of the filtered matrix matches the original within a small numerical tolerance (the shift-invariance invariant)
- Pairwise gene differences are unchanged after the pseudocount shift α = min(X~)
- Filtered values, after applying α = min(X~), fall on a valid expression scale
- Kernel rows each sum to 1 (row-stochastic property) before projection

## Limitations

- SiFT strongly relies on the prior knowledge used to define the cell-cell similarity kernel; an inaccurate or coarse-grained mapping may lead to a loss of signal
- Filtering modifies gene expression so the data is no longer properly log-transformed, which is expected in certain downstream analysis tasks
- No ground truth reference exists to evaluate computational filtering results; integration metrics may be application-specific

## Evidence

- [intro] we compute a probabilistic cell-cell similarity kernel, which captures the similarity between cells according to the biological signal we wish to filter: "we compute a probabilistic cell–cell similarity kernel, which captures the similarity between cells according to the biological signal we wish to filter"
- [intro] we obtain a projection of the expression count matrix onto the signal we seek to filter using matrix multiplication: "we obtain a projection of the expression count matrix onto the signal we seek to filter using matrix multiplication"
- [methods] sift.kernels.RBFKernel LaplacianKernel KnnKernel MappingKernel list: "sift.kernels.RBFKernel
    sift.kernels.LaplacianKernel
    sift.kernels.KnnKernel
    sift.kernels.MappingKernel"
- [intro] These attributes are used to compute a cell-cell similarity kernel with respect to the encoded signal: "These attributes are used to compute a cell–cell similarity kernel with respect to the encoded signal"
