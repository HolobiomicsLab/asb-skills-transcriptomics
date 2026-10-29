---
name: kernel-regularized-nmf-factorization
description: Use when decomposing a log-normalized single-cell RNA-seq matrix into
  nonnegative gene expression programs while regularizing cell GEP weights toward
  similarity to nearby cells in a q-diffused graph.
when_to_use_negative:
- When the input matrix has already undergone feature selection or dimensionality
  reduction — qNMF is designed to work on the full log-normalized gene library (*.normalized
  element)
- When batch-effect correction or alignment of disparate datasets is required — the
  paper notes this was not considered and pre-existing tools must be coopted upstream
- When a fast runtime is essential — q-diffused kernel evaluation across all cell
  pairs takes 20 min to 3 h per affinity matrix
edam_operation: http://edamontology.org/operation_3947
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: QDiffusion.jl
  role: Julia package providing preprocess_cell_counts, estimate_inner_bandwidth,
    AdaptiveKernel/DeformedKernel, and par_estimate_nmf used to run kernel-regularized
    NMF at rank 16 with GPU acceleration via cuda_devices
  repo: github.com/marmarelis/QDiffusion.jl
- name: q-diffusion
  role: Framework whose q-diffused kernel geometry constrains the NMF factorization
    through the symmetrized cell affinity matrix
  repo: github.com/marmarelis/QDiffusion.jl
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s42003-024-06104-w
    title: q-Diffusion leverages the full dimensionality of gene coexpression in single-cell
      transcriptomics
  source_content_hash: sha256:8c1b197401c00c0991204c17e96fda75ca17cf1bd57322380f1a13d34aede1f3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - QDiffusion.jl
  - q-Diffusion.jl
  - q-diffusion
  edam_operation: http://edamontology.org/operation_3947
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5333333333333333
n_claims: 15
---

# kernel-regularized-nmf-factorization

## Summary

Kernel-regularized nonnegative matrix factorization (qNMF) estimates gene expression programs (GEPs) by constraining NMF cell weights with a q-diffused cell affinity matrix, reducing underdetermination of the factorization. It is implemented in QDiffusion.jl via par_estimate_nmf on a log-normalized (genes x cells) scRNAseq matrix.

## When to use

Use when decomposing a log-normalized single-cell RNA-seq matrix into nonnegative gene expression programs while regularizing cell GEP weights toward similarity to nearby cells in a q-diffused graph.

## When NOT to use

- When the input matrix has already undergone feature selection or dimensionality reduction — qNMF is designed to work on the full log-normalized gene library (*.normalized element)
- When batch-effect correction or alignment of disparate datasets is required — the paper notes this was not considered and pre-existing tools must be coopted upstream
- When a fast runtime is essential — q-diffused kernel evaluation across all cell pairs takes 20 min to 3 h per affinity matrix

## Inputs

- Log-normalized (genes x cells) scRNAseq count matrix from preprocess_cell_counts (*.normalized element; no feature selection or dimensionality reduction)
- Estimated inner bandwidth from estimate_inner_bandwidth
- q-diffused affinity matrix A built from DeformedKernel(q=1.2) wrapped in AdaptiveKernel and symmetrized as A = (A~ + A~T)/2

## Outputs

- Nonnegative GEP weight matrix (cells x 16 programs) and gene loading matrix interpretable as gene expression programs
- Sparsity-screened GEP weights (programs with sparsity between 5% and 95%)
- Optionally binarized per-cell/per-patient GEP weights (0 for zero, 1 for nonzero) as a qNMF biomarker

## How to apply

Load the log-normalized (genes x cells) matrix from preprocess_cell_counts using the *.normalized element (log-normalized, no feature selection or dimensionality reduction). Estimate the inner bandwidth with estimate_inner_bandwidth, which computes average pairwise squared distance per gene and takes the median (50% quantile) of gene scatters. Build a DeformedKernel with deformation parameter q = 1.2 wrapped in AdaptiveKernel (outer bandwidth set to the kth nearest q-diffused norm in each cell's neighborhood), then symmetrize the anisotropic affinity matrix arithmetically as A = (A~ + A~T)/2. Run par_estimate_nmf at rank 16 GEPs with regularizations regularizing cell GEP weights toward the q-diffused affinity matrix, a fixed random seed (e.g. seed=1337 in the README example), and n_iterations=512. Screen the estimated GEP weights for sparsity between 5% and 95%, excluding entirely zero or entirely nonzero programs; the paper further binarizes patient GEP weights (0 for zero, 1 for nonzero) to obtain a qNMF biomarker.

## Related tools

- **QDiffusion.jl** (Julia package providing preprocess_cell_counts, estimate_inner_bandwidth, AdaptiveKernel/DeformedKernel, and par_estimate_nmf used to run kernel-regularized NMF at rank 16 with GPU acceleration via cuda_devices) — github.com/marmarelis/QDiffusion.jl
- **q-diffusion** (Framework whose q-diffused kernel geometry constrains the NMF factorization through the symmetrized cell affinity matrix) — github.com/marmarelis/QDiffusion.jl

## Examples

```
factors = par_estimate_nmf(kernel, data, bandwidth, rank=16, regularizations=[0f0, 1f0], n_iterations=512, cuda_devices=[0], seed=1337)
```

## Evaluation signals

- Top-weighted genes in each qNMF GEP are almost equivalent to those of plain NMF initialized with the same random seed, as the paper reports for the resultant programs
- Estimated GEP weights have sparsity between 5% and 95% — entirely zero or entirely nonzero programs are excluded
- Factorization uses a fixed random seed (e.g. seed=1337) so re-running par_estimate_nmf reproduces the same GEPs
- Binarized GEP weights yield a qNMF biomarker with a robust differential hazard on progression-free survival under bootstrap resampling in the CALGB/SWOG 80405 trial

## Limitations

- No algorithm for selecting the q-diffusion parameters q and φ (deformation and inner bandwidth) is provided; q = 1.2 is chosen heuristically
- Runtime of the q-diffused kernel evaluation is suboptimal — 20 min to 3 h for one affinity matrix across all cell pairs
- q-diffusion makes parametric assumptions on the data that might be restrictive in some use cases
- No batch-effect correction or alignment of disparate datasets was considered; pre-existing tools must be coopted upstream

## Evidence

- [results] qNMF represents cells in a small linear (nonnegative) basis interpretable as gene expression programs (GEPs): [evidence span withheld — anchor status: not_found]
- [results] We then binarized the patients' weights for that GEP based on sparsity: 0 for zero and 1 for nonzero, obtaining the qNMF biomarker.: [evidence span withheld — anchor status: not_found]
- [readme] The `*.normalized` element is simply log-normalized, sans feature selection or dimensionality reduction, like we do in our study.: "The `*.normalized` element is simply log-normalized, sans feature selection or dimensionality reduction, like we do in our study."
- [readme] par_estimate_nmf(kernel, data, bandwidth, rank=16, regularizations=[0f0, 1f0], n_iterations=512, cuda_devices=[0], seed=1337): "factors = par_estimate_nmf(kernel, data, bandwidth,
  rank=16, # number of gene expression programs (GEPs)
  regularizations=[0f0, 1f0], # how much to regularize by kernel"
- [readme] It is recommended to wrap one of these in an `AdaptiveKernel` type that adjusts the base kernel's bandwidth relative to $k$-nn distances.: "It is recommended to wrap one of these in an `AdaptiveKernel` type that adjusts the base kernel's bandwidth relative to $k$-nn distances."
