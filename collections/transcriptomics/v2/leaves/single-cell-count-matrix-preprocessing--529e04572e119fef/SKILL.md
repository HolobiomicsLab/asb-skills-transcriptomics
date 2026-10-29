---
name: single-cell-count-matrix-preprocessing
description: Use when a raw single-cell gene expression count matrix must be preprocessed
  to remove an annotated unwanted signal (batch, cell cycle, sex, spatial zonation,
  healthy-reference trajectory) before downstream analysis.
when_to_use_negative:
- When no prior knowledge is available to define a meaningful cell–cell similarity
  kernel — SiFT strongly relies on the prior knowledge used to define the kernel,
  and an inaccurate or coarse-grained mapping may lead to a loss of signal.
- When the downstream task requires properly log-transformed expression data, since
  SiFT modifies gene expression so the data are no longer properly log-transformed.
- When batch-effect correction at the level of a joint latent representation suffices
  (e.g., via scVI, Harmony, or bbknn), rather than correction at the gene expression
  level.
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3170
tools:
- name: SiFT (sift-sc)
  role: Python package implementing kernel construction, kernel projection (K·X),
    subtraction, and pseudocount correction of the count matrix
  repo: https://github.com/nitzanlab/sift-sc
- name: pip
  role: Installation channel for the sift-sc package (pip install sift-sc)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-44757-7
    title: 'SiFT: uncovering hidden biological processes by probabilistic filtering
      of single-cell data'
  source_content_hash: sha256:153471b1a2f738b93a787a5c6d9f6d734d3eb485a14be0de52807a4ce1964008
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nitzanlab/sift-sc
  tools:
  - SiFT
  - SiFT (sift-sc)
  - pip
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# single-cell-count-matrix-preprocessing

## Summary

Kernel-based preprocessing of single-cell gene expression count matrices in which the counts are projected onto a cell–cell similarity kernel (P = K·X) and the projection is subtracted to filter an unwanted or known biological signal, with a pseudocount correction restoring a valid expression scale.

## When to use

Use when a raw single-cell gene expression count matrix must be preprocessed to remove an annotated unwanted signal (batch, cell cycle, sex, spatial zonation, healthy-reference trajectory) before downstream analysis.

## When NOT to use

- When no prior knowledge is available to define a meaningful cell–cell similarity kernel — SiFT strongly relies on the prior knowledge used to define the kernel, and an inaccurate or coarse-grained mapping may lead to a loss of signal.
- When the downstream task requires properly log-transformed expression data, since SiFT modifies gene expression so the data are no longer properly log-transformed.
- When batch-effect correction at the level of a joint latent representation suffices (e.g., via scVI, Harmony, or bbknn), rather than correction at the gene expression level.

## Inputs

- single-cell gene expression count matrix (e.g., AnnData/.h5ad object)
- cell attribute annotation encoding the signal to be filtered (e.g., cell cycle phase, spatial coordinates, sample/batch label, mapped reference trajectory)

## Outputs

- filtered expression matrix X~ = X − K·X with pseudocount correction α = min(X~) (e.g., written to filtered_expression.h5ad)
- cell–cell similarity kernel constructed from the annotated attribute

## How to apply

Install the SiFT package (pip install sift-sc) and load it with 'import sift'. Load the count matrix together with a cell attribute annotation encoding the signal to filter, then build a row-stochastic cell–cell similarity kernel from that attribute using one of sift.kernels (MappingKernel for mapping-defined signals, KnnKernel for neighborhood similarity, or RBFKernel/LaplacianKernel for distance-based similarity). Project the expression matrix onto the signal by kernel multiplication (P = K·X) via the sift.SiFT / sift.sifter interface and subtract the projection to obtain the filtered data X~ = X − K·X, so each cell's expression becomes its deviation from the expected expression of its neighboring cells. Apply a global pseudocount correction α = min(X~) to shift the filtered values back to a valid expression scale while retaining gene-to-gene differences, then write the result (e.g., filtered_expression.h5ad). Choose the kernel variant according to whether the signal is a categorical mapping, a neighborhood structure, or a continuous distance metric on cell attributes.

## Related tools

- **SiFT (sift-sc)** (Python package implementing kernel construction, kernel projection (K·X), subtraction, and pseudocount correction of the count matrix) — https://github.com/nitzanlab/sift-sc
- **pip** (Installation channel for the sift-sc package (pip install sift-sc))

## Examples

```
pip install sift-sc; import sift; sft = sift.SiFT(data=adata, kernel=sift.kernels.KnnKernel(n_neighbors=30)); sft.filter()
```

## Evaluation signals

- Total variance of the filtered matrix matches the original within a small numerical tolerance (invariant check)
- Pairwise gene differences are unchanged after the pseudocount α shift
- Filtered values are shifted back to a valid expression scale after applying α = min(X~)
- Downstream integration metrics (application-specific) indicate the annotated nuisance signal is removed while biological attributes are preserved
- Filtered matrix written successfully to filtered_expression.h5ad with invariant check results reported

## Limitations

- SiFT strongly relies on the prior knowledge used to define the cell–cell similarity kernel; an inaccurate or coarse-grained mapping may lead to a loss of signal.
- There is no ground truth reference to evaluate computational filtering results; evaluation relies on integration metrics that may be application-specific.
- Filtered data are no longer properly log-transformed, which some downstream analysis tasks expect; statistical implications of applying common tools to corrected gene expression data are unexplored.

## Evidence

- [intro] we obtain a projection of the expression count matrix onto the signal we seek to filter using matrix multiplication: "we obtain a projection of the expression count matrix onto the signal we seek to filter using matrix multiplication"
- [methods] Install SiFT in one of the following ways: Through pip:: pip install sift-sc: "Through **pip**::

    pip install sift-sc"
- [methods] sift.kernels.RBFKernel, LaplacianKernel, KnnKernel, MappingKernel API variants: "sift.kernels.RBFKernel
    sift.kernels.LaplacianKernel
    sift.kernels.KnnKernel
    sift.kernels.MappingKernel"
- [discussion] This kernel is then used to obtain a projection of gene expression onto the signal, which is then removed from the original expression.: "This kernel is then used to obtain a projection of gene expression onto the signal, which is then removed from the original expression."
- [readme] README: SiFT can be applied from removal of unwanted variation to uncovering trajectories of interest: "It can be applied to a wide range of tasks, from the removal of unwanted variation as a pre-processing step,
through revealing hidden biological structure by utilizing prior knowledge with respect to"
