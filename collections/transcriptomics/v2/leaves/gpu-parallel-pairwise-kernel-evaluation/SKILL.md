---
name: gpu-parallel-pairwise-kernel-evaluation
description: Use when evaluating a q-diffused or Gaussian kernel over a full n×n cell-pair
  affinity matrix on a (genes × cells) matrix, and the pairwise gene-to-gene computation
  is too slow on CPU (the paper reports 20 min to 3 h per affinity matrix).
when_to_use_negative:
- No CUDA-enabled GPU is available — omitting cuda_devices delegates to a CPU-bound
  method, so this skill cannot be applied (expect 20 min–3 h runtime per affinity
  matrix).
- The data matrix has already undergone feature selection or dimensionality reduction
  — the kernel design assumes the full log-normalized transcriptome, so GPU-accelerated
  full-pairwise evaluation would bake in discarded high-order gene interactions.
edam_operation: http://edamontology.org/operation_2938
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_3308
tools:
- name: q-Diffusion.jl
  role: Provides the cuda_devices argument on kernel construction (AdaptiveKernel/DeformedKernel)
    and analysis functions (par_estimate_nmf, leave_it_to_phate, par_spatially_cluster_mmd)
    to parallelize cell-by-cell, gene-to-gene kernel computations across CUDA GPUs;
    without it, execution falls back to CPU-bound methods.
  repo: github.com/marmarelis/QDiffusion.jl
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s42003-024-06104-w
    title: q-Diffusion leverages the full dimensionality of gene coexpression in single-cell
      transcriptomics
  source_content_hash: sha256:45af8d37a6f65f0375b53725d7bfb0f2f7ad71c908e8d0f1124a9c5a447d5d3c
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - q-Diffusion.jl
  - q-diffusion
  edam_operation: http://edamontology.org/operation_2938
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# gpu-parallel-pairwise-kernel-evaluation

## Summary

Accelerates evaluation of q-diffused (or Gaussian) kernels between all pairs of scRNAseq cells by delegating cell-by-cell, gene-to-gene computations to CUDA GPUs via the cuda_devices argument in QDiffusion.jl, instead of the CPU-bound default.

## When to use

Use when evaluating a q-diffused or Gaussian kernel over a full n×n cell-pair affinity matrix on a (genes × cells) matrix, and the pairwise gene-to-gene computation is too slow on CPU (the paper reports 20 min to 3 h per affinity matrix).

## When NOT to use

- No CUDA-enabled GPU is available — omitting cuda_devices delegates to a CPU-bound method, so this skill cannot be applied (expect 20 min–3 h runtime per affinity matrix).
- The data matrix has already undergone feature selection or dimensionality reduction — the kernel design assumes the full log-normalized transcriptome, so GPU-accelerated full-pairwise evaluation would bake in discarded high-order gene interactions.

## Inputs

- Log-normalized (genes x cells) scRNAseq count matrix (e.g. the *.normalized element from preprocess_cell_counts, with no feature selection or dimensionality reduction)
- Kernel configuration: DeformedKernel/AdaptiveKernel parameters (deformation q, inner bandwidth, k-nn scale such as 16th nearest neighbor)
- List of CUDA device ID(s) and Julia thread count

## Outputs

- n×n cell-pair affinity/adjacency matrix (ready for symmetrization as A = (Ã + Ãᵀ)/2 and downstream embedding or NMF)
- Optionally downstream artifacts: NMF factors (par_estimate_nmf) or qPHATE embedding (leave_it_to_phate) computed with GPU acceleration

## How to apply

Construct the kernel (e.g. AdaptiveKernel wrapping a DeformedKernel with deformation parameter q and inner bandwidth from estimate_inner_bandwidth) and pass the cuda_devices argument when building it or when running analysis algorithms like par_estimate_nmf or leave_it_to_phate. The array you pass should either have a single element (one CUDA device ID used by all threads) or length equal to the number of threads your Julia instance runs, specifying the CUDA device ID for each thread; omitting cuda_devices delegates to a CPU-bound method. This parallelization exists because the q-diffused kernel must be expanded over all orders of gene-interaction terms via the recursive q-sum for every cell pair, which is expensive across cell-by-cell, gene-to-gene computations. Log-transform (in_log=true) remains necessary for numerical stability regardless of the compute device.

## Related tools

- **q-Diffusion.jl** (Provides the cuda_devices argument on kernel construction (AdaptiveKernel/DeformedKernel) and analysis functions (par_estimate_nmf, leave_it_to_phate, par_spatially_cluster_mmd) to parallelize cell-by-cell, gene-to-gene kernel computations across CUDA GPUs; without it, execution falls back to CPU-bound methods.) — github.com/marmarelis/QDiffusion.jl

## Examples

```
kernel = AdaptiveKernel(16, DeformedKernel(1.2f0, data, inner_bandwidth=estimate_inner_bandwidth(data)), data, cuda_devices=[0], in_log=true)
```

## Evaluation signals

- The affinity matrix produced by the GPU path should match the CPU path for the same kernel parameters (same q, inner bandwidth, in_log=true), since the device only changes parallelization, not values.
- Affinity matrix shape is n×n matching cell count, symmetric after A = (Ã + Ãᵀ)/2 symmetrization, with kernel values in (0,1] as expected for a q-exponential kernel.
- Check that the length of the cuda_devices array is either 1 or equal to the number of running Julia threads — any other length is invalid per the package convention.
- Wall-clock time for kernel evaluation should drop substantially relative to the CPU-bound runtime reported (20 min to 3 h per affinity matrix).
- GPU memory must accommodate the cell-by-cell, gene-to-gene intermediate tensors; if a device OOMs, split cells into batches rather than silently falling back.

## Limitations

- Runtime of the q-diffused kernel evaluation is suboptimal even with parallelization — the paper reports 20 min to 3 h for one affinity matrix, so O(n²) cell pairs plus full gene-order expansion dominate cost.
- Device-to-thread mapping is constrained: the cuda_devices array must have a single element or length equal to Julia's thread count; other configurations are not supported.
- No batch-effect correction or alignment of disparate datasets was considered upstream, so GPU-accelerated affinity matrices assume the input matrix is not confounded by batch effects.

## Evidence

- [readme] GPU acceleration via cuda_devices: "you would probably benefit from using a GPU to accelerate all the cell-by-cell, gene-to-gene computations that need to be performed"
- [readme] cuda_devices array convention: "the array you pass in should either have a single element or length equal to the number of threads that your Julia instance has running"
- [readme] CPU fallback: "If you don't pass a value to the `cuda_devices` argument, it will delegate to a CPU-bound method."
- [discussion] kernel evaluation cost: "there is room for improving the runtime efficiency of the algorithm evaluating the q-diffused kernel across all pairs of cells."
- [results] all-pairs kernel then symmetrize: "the q-diffused kernel was evaluated between all pairs of cells, and then the adjacency matrix was symmetrified to produce a weighted, undirected graph of the cells"
