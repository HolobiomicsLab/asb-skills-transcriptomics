---
name: matrix-projection-filtering-of-single-cell-expression
description: Use when a single-cell expression matrix carries an annotated nuisance
  signal (cell cycle, sex, spatial zonation, batch, or healthy-vs-disease status)
  that must be removed at the gene expression level before downstream analysis.
when_to_use_negative:
- Downstream tasks require properly log-transformed expression data — SiFT's correction
  modifies gene expression so the data is no longer properly log-transformed
- No reliable prior knowledge or annotation exists to define the cell–cell similarity
  kernel — an inaccurate or coarse-grained mapping may lead to a loss of signal
- The goal is integration or correction at a joint latent representation level (e.g.,
  scVI-style models) rather than correction at the gene expression level
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: SiFT (sift-sc)
  role: Implements the kernel construction (MappingKernel, KnnKernel, RBFKernel, LaplacianKernel),
    the projection P = K·X, the subtraction, and the pseudocount correction
  repo: https://github.com/nitzanlab/sift-sc
- name: pip
  role: Installs the sift-sc package from PyPI
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-024-44757-7
    title: 'SiFT: uncovering hidden biological processes by probabilistic filtering
      of single-cell data'
  source_content_hash: sha256:4c171a0121afaf26c9e26205d049b7a7d4df145e39f92a272a109d5e954b5853
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/nitzanlab/sift-sc
  tools:
  - SiFT
  - SiFT (sift-sc)
  - pip
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# matrix-projection-filtering-of-single-cell-expression

## Summary

Kernel-based projection filtering for single-cell gene expression data: a cell–cell similarity kernel encoding the signal to remove is built with SiFT, the count matrix is projected onto it, and the projection is deducted (X~ = X − K·X) with a pseudocount correction, filtering unwanted variation at the gene expression level.

## When to use

Use when a single-cell expression matrix carries an annotated nuisance signal (cell cycle, sex, spatial zonation, batch, or healthy-vs-disease status) that must be removed at the gene expression level before downstream analysis.

## When NOT to use

- Downstream tasks require properly log-transformed expression data — SiFT's correction modifies gene expression so the data is no longer properly log-transformed
- No reliable prior knowledge or annotation exists to define the cell–cell similarity kernel — an inaccurate or coarse-grained mapping may lead to a loss of signal
- The goal is integration or correction at a joint latent representation level (e.g., scVI-style models) rather than correction at the gene expression level

## Inputs

- Single-cell gene expression count matrix (e.g., AnnData/.h5ad)
- Cell attribute annotation encoding the signal to be filtered (e.g., cell cycle score, sex, spatial coordinates, healthy/disease label)
- Optional reference sample labels or mapping used to define the cell–cell similarity kernel

## Outputs

- Filtered expression matrix X~ = X − K·X with pseudocount correction α = min(X~), written to filtered_expression.h5ad
- Invariant check results (variance preservation, pairwise gene-difference preservation) written to a report such as sift_invariants_report

## How to apply

Install the package with `pip install sift-sc` and import it as `import sift`. Build a row-stochastic cell–cell similarity kernel from the annotated attribute using sift.kernels — MappingKernel when the signal is defined by a mapping (e.g., cell-type or reference labels), KnnKernel for neighborhood-based similarity, or RBFKernel/LaplacianKernel for distance-based similarity (e.g., spatial coordinates). Compute the projection of the count matrix onto the signal via kernel multiplication (P = K·X) and subtract it from the original data to obtain the filtered expression X~ = X − K·X using the sift.SiFT / sift.sifter interface. Apply the global pseudocount correction α = min(X~) so filtered values are shifted back to a valid expression scale while gene-to-gene differences are retained. The rationale is that after filtering, each cell's expression represents its deviation from the expected expression of its neighboring cells, so downstream analysis (PCA, clustering, trajectories) reflects the signal of interest rather than the filtered nuisance process.

## Related tools

- **SiFT (sift-sc)** (Implements the kernel construction (MappingKernel, KnnKernel, RBFKernel, LaplacianKernel), the projection P = K·X, the subtraction, and the pseudocount correction) — https://github.com/nitzanlab/sift-sc
- **pip** (Installs the sift-sc package from PyPI)

## Examples

```
pip install sift-sc; import sift
```

## Evaluation signals

- Total variance of the filtered matrix matches the original within a small numerical tolerance
- Pairwise gene differences are unchanged after the α = min(X~) pseudocount shift
- Kernel rows are row-stochastic (each cell's similarities sum to 1) before projection
- Filtered values lie on a valid expression scale after the α shift, with gene-to-gene differences retained
- Filtered matrix is successfully written to filtered_expression.h5ad with invariant check results reported

## Limitations

- Lacks a ground truth reference to evaluate results; application-specific integration metrics are suggested as mitigation
- Filtered data is no longer properly log-transformed, which certain downstream analysis tasks expect
- SiFT strongly relies on the prior knowledge used to define the cell–cell similarity kernel; an inaccurate or coarse-grained mapping may lead to a loss of signal
- Statistical implications of applying commonly used analysis tools to the corrected gene expression data remain unexplored

## Evidence

- [intro] we compute a probabilistic cell–cell similarity kernel, which captures the similarity between cells according to the biological signal we wish to filter: "we compute a probabilistic cell–cell similarity kernel, which captures the similarity between cells according to the biological signal we wish to filter"
- [intro] we obtain a projection of the expression count matrix onto the signal we seek to filter using matrix multiplication: "we obtain a projection of the expression count matrix onto the signal we seek to filter using matrix multiplication"
- [methods] Install `SiFT` in one of the following ways: Through **pip**:: pip install sift-sc: "Install `SiFT` in one of the following ways: Through **pip**:: pip install sift-sc"
- [discussion] This kernel is then used to obtain a projection of gene expression onto the signal, which is then removed from the original expression.: "This kernel is then used to obtain a projection of gene expression onto the signal, which is then removed from the original expression."
- [discussion] it modifies the gene expression so that data is no longer properly log-transformed, a property that is expected in certain downstream analysis tasks: "it modifies the gene expression so that data is no longer properly log-transformed, a property that is expected in certain downstream analysis tasks"
- [discussion] As we showcase in various examples, an inaccurate or coarse-grained mapping may lead to a loss of signal.: "As we showcase in various examples, an inaccurate or coarse-grained mapping may lead to a loss of signal."
