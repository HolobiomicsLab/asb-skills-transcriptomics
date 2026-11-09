---
name: gene-expression-program-estimation
description: Use when a (genes x cells) log-normalized scRNAseq matrix must be decomposed
  into a small set of interpretable, nonnegative gene expression programs whose cell
  weights are regularized toward similarity of nearby cells in a q-diffused graph.
when_to_use_negative:
- Input matrix has been feature-selected or dimensionality-reduced — the q-diffused
  framework relies on full transcriptome dimensionality without aggressive feature
  selection
- Goal is clustering/cell-type annotation rather than decomposing expression into
  continuous linear programs — Leiden community detection on the kernel graph is the
  appropriate alternative
- Cells lack a meaningful neighborhood structure or the affinity matrix is degenerate,
  since the kernel regularization constraining GEP weights would be unfounded
edam_operation: http://edamontology.org/operation_3919
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: q-Diffusion.jl
  role: Julia package providing preprocess_cell_counts, estimate_inner_bandwidth,
    DeformedKernel/AdaptiveKernel, and par_estimate_nmf for qNMF estimation
  repo: github.com/marmarelis/QDiffusion.jl
- name: q-diffusion
  role: Framework supplying the q-diffused kernel geometry whose cell graph regularizes
    the NMF factorization toward GEP-weight similarity of nearby cells
provenance:
  source_task_ids:
  - task_006
  - task_007
  source_papers:
  - doi: 10.1038/s42003-024-06104-w
    title: q-Diffusion leverages the full dimensionality of gene coexpression in single-cell
      transcriptomics
  source_content_hash: sha256:2b947c53b407d77a4eb5d9fd7460c5ac7627db501f56a7d55896af27e5f8bdb2
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - PHATE
  - q-Diffusion.jl
  - q-diffusion
  edam_operation: http://edamontology.org/operation_3919
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# gene-expression-program-estimation

## Summary

Estimates gene expression programs (GEPs) by kernel-regularized nonnegative matrix factorization (qNMF) constrained by a q-diffused cell graph, representing cells in a small nonnegative basis of interpretable GEPs while lowering the risk of underdetermination relative to plain NMF.

## When to use

Use when a (genes x cells) log-normalized scRNAseq matrix must be decomposed into a small set of interpretable, nonnegative gene expression programs whose cell weights are regularized toward similarity of nearby cells in a q-diffused graph.

## When NOT to use

- Input matrix has been feature-selected or dimensionality-reduced — the q-diffused framework relies on full transcriptome dimensionality without aggressive feature selection
- Goal is clustering/cell-type annotation rather than decomposing expression into continuous linear programs — Leiden community detection on the kernel graph is the appropriate alternative
- Cells lack a meaningful neighborhood structure or the affinity matrix is degenerate, since the kernel regularization constraining GEP weights would be unfounded

## Inputs

- log-normalized (genes x cells) scRNAseq matrix (the *.normalized element of preprocess_cell_counts)
- q-diffused or Gaussian kernel built via DeformedKernel/AdaptiveKernel with an estimated inner bandwidth
- random seed for reproducible NMF initialization

## Outputs

- set of ~16 gene expression programs (GEPs) and per-cell nonnegative GEP weight matrices
- sparsity-screened GEP weights (5%-95% nonzero fraction) usable as latent biomarkers for downstream Cox regression

## How to apply

Load the log-normalized matrix via preprocess_cell_counts using the *.normalized element (log-normalized, no feature selection or dimensionality reduction), then estimate the inner bandwidth with estimate_inner_bandwidth as the 50% (median) quantile of per-gene average pairwise squared distances. Build the q-diffused kernel with DeformedKernel at q = 1.2 wrapped in AdaptiveKernel (outer bandwidth scaled to the kth nearest neighbor), computing in log space on GPU, and symmetrize the anisotropic affinity arithmetically as A = (A~ + A~T)/2. Run par_estimate_nmf at rank 16 with kernel regularization (e.g., regularizations=[0f0, 1f0], n_iterations=512) and a fixed random seed (e.g., 1337). Screen the estimated GEP weights for sparsity between 5% and 95%, excluding entirely zero or entirely nonzero programs; for biomarker use, binarize patient weights (0 for zero, 1 for nonzero).

## Related tools

- **q-Diffusion.jl** (Julia package providing preprocess_cell_counts, estimate_inner_bandwidth, DeformedKernel/AdaptiveKernel, and par_estimate_nmf for qNMF estimation) — github.com/marmarelis/QDiffusion.jl
- **q-diffusion** (Framework supplying the q-diffused kernel geometry whose cell graph regularizes the NMF factorization toward GEP-weight similarity of nearby cells)

## Examples

```
factors = par_estimate_nmf(kernel, data, bandwidth, rank=16, regularizations=[0f0, 1f0], n_iterations=512, cuda_devices=[0], seed=1337)
```

## Evaluation signals

- Top-weighted genes in each qNMF GEP are almost equivalent to those of plain NMF initialized with the same random seed, as the paper reports for the resultant programs
- Every retained GEP has sparsity between 5% and 95% nonzero cell weights — exclude entirely zero or entirely nonzero programs
- Estimation uses a fixed random seed (e.g., 1337) and reruns reproduce identical factorizations
- Rank is set to 16 GEPs and regularization is applied via the q-diffused affinity matrix (bandwidth ~1.0 with an adaptive kernel)
- Downstream, one qNMF GEP shows a significant differential treatment effect at Benjamini-Hochberg FDR < 0.05 in the CALGB/SWOG 80405 Cox analysis, whereas classical NMF with the same seed produces no such GEP

## Limitations

- No algorithm is provided for selecting the q-diffusion parameters q and φ (inner bandwidth); q = 1.2 was chosen empirically
- Runtime of the q-diffused kernel evaluation is suboptimal (20 min to 3 h for one affinity matrix), though GPU acceleration via cuda_devices helps
- q-diffusion makes parametric assumptions on the data that might be restrictive in some use cases
- Compared to the q-diffused version, classical NMF with the same seed yielded no statistically significant differential treatment effect GEPs

## Evidence

- [results] qNMF represents cells in a small linear (nonnegative) basis interpretable as gene expression programs: [evidence span withheld — anchor status: not_found]
- [methods] inner bandwidth estimate: "inner_bandwidth = estimate_inner_bandwidth(data)"
- [readme] NMF estimation parameters: "factors = par_estimate_nmf(kernel, data, bandwidth,
  rank=16, # number of gene expression programs (GEPs)"
- [methods] normalized element of preprocessing: "The `*.normalized` element is simply log-normalized, sans feature selection or dimensionality reduction, like we do in our study."
- [results] binarization of GEP weights into the qNMF biomarker: [evidence span withheld — anchor status: not_found]
