---
name: inner-bandwidth-estimation-from-gene-scatter
description: Use when constructing a DeformedKernel/AdaptiveKernel affinity matrix
  on a log-normalized (genes x cells) scRNAseq matrix and a data-driven inner bandwidth
  is needed instead of hand-tuning φ.
when_to_use_negative:
- The input is not a (genes x cells) expression matrix — e.g. already dimensionality-reduced
  embeddings or PCA scores — since the heuristic relies on per-gene pairwise squared
  distances on the same matrix passed to the kernel
- The data is raw counts or otherwise not log-normalized, which the authors' study
  pipeline assumes before kernel construction
- Batch-effect correction or alignment of disparate datasets is needed upstream, which
  the paper explicitly did not consider
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: q-Diffusion.jl
  role: Provides estimate_inner_bandwidth and the kernel constructors (DeformedKernel,
    AdaptiveKernel) that consume the estimated bandwidth
  repo: github.com/marmarelis/QDiffusion.jl
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s42003-024-06104-w
    title: q-Diffusion leverages the full dimensionality of gene coexpression in single-cell
      transcriptomics
  source_content_hash: sha256:ef2364a8c130b43bcda672f957b236fc4a91d1ffd0807261e2c2fb19c1907369
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - q-Diffusion.jl
  - q-diffusion
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 7
---

# inner-bandwidth-estimation-from-gene-scatter

## Summary

Heuristic estimation of the inner bandwidth (φ) parameter of the q-diffused kernel by computing the median (50% quantile) of per-gene scatters on a log-normalized (genes x cells) scRNAseq matrix, via QDiffusion.jl's estimate_inner_bandwidth.

## When to use

Use when constructing a DeformedKernel/AdaptiveKernel affinity matrix on a log-normalized (genes x cells) scRNAseq matrix and a data-driven inner bandwidth is needed instead of hand-tuning φ.

## When NOT to use

- The input is not a (genes x cells) expression matrix — e.g. already dimensionality-reduced embeddings or PCA scores — since the heuristic relies on per-gene pairwise squared distances on the same matrix passed to the kernel
- The data is raw counts or otherwise not log-normalized, which the authors' study pipeline assumes before kernel construction
- Batch-effect correction or alignment of disparate datasets is needed upstream, which the paper explicitly did not consider

## Inputs

- Log-normalized (genes x cells) single-cell expression matrix, e.g. the *.normalized element from preprocess_cell_counts(data)

## Outputs

- A scalar inner bandwidth value (e.g. Float32, as in the README's 1.125f0 example) supplied as the inner_bandwidth keyword to DeformedKernel or PossiblyDeformedKernel

## How to apply

Prepare the (genes x cells) matrix by log-normalizing or taking the *.normalized element returned by preprocess_cell_counts (no feature selection or dimensionality reduction). Call estimate_inner_bandwidth(data), passing exactly the same matrix you will pass to the kernel constructor; the function computes the average pairwise squared distance per gene and takes the 50% (median) quantile of gene scatters. Pass the result as the inner_bandwidth keyword argument to DeformedKernel (e.g. the README example uses inner_bandwidth=1.125f0 with q=1.5, while the qNMF workflow uses q=1.2), then wrap in AdaptiveKernel. Because the kernel is multiscale, this bandwidth sets which gene-expression magnitudes the deformed kernel takes seriously downstream in NMF, PHATE, or LDS.

## Related tools

- **q-Diffusion.jl** (Provides estimate_inner_bandwidth and the kernel constructors (DeformedKernel, AdaptiveKernel) that consume the estimated bandwidth) — github.com/marmarelis/QDiffusion.jl

## Examples

```
inner_bandwidth = estimate_inner_bandwidth(data)
```

## Evaluation signals

- The returned bandwidth is a single positive scalar computed from the same data matrix passed to the kernel, matching the README instruction to 'pass in the same data matrix as you would to the kernel'
- The estimate is deterministic: re-running estimate_inner_bandwidth on the identical log-normalized matrix reproduces the same median-gene-scatter value
- Downstream validation mirrors the qNMF acceptance criterion: the top-weighted genes in each qNMF GEP should be almost equivalent to those of plain NMF initialized with the same random seed
- The estimated GEP weights show sparsity between 5% and 95%, excluding entirely zero or entirely nonzero programs

## Limitations

- The heuristic is only approximate — the README cautions it 'very approximately' tells you what gene expressions to take seriously
- The paper acknowledges no formal selection algorithm exists: it notes the need for 'a careful selection algorithm for non-Euclidean deformation q and inner bandwidth φ, the two vital parameters to q-diffusion'
- q-diffusion makes parametric assumptions that may be restrictive in some use cases, affecting how well the bandwidth transfers across datasets

## Evidence

- [readme] The inner bandwidth controls the multiscale nature of the deformed kernel. Very approximately, it tells us what gene expressions to take seriously.: "The inner bandwidth controls the multiscale nature of the deformed kernel. Very approximately, it tells us what gene expressions to take seriously."
- [methods] inner_bandwidth = estimate_inner_bandwidth(data): "inner_bandwidth = estimate_inner_bandwidth(data)"
- [readme] pass in the same data matrix as you would to the kernel: "# pass in the same data matrix as you would to the kernel"
- [full_text] Estimate the inner bandwidth with estimate_inner_bandwidth, which computes the average pairwise squared distance per gene and takes the 50% (median) quantile of gene scatters.: [evidence span withheld — anchor status: not_found]
