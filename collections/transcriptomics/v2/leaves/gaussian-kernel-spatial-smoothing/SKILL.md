---
name: gaussian-kernel-spatial-smoothing
description: Use when you have ST spot spatial coordinates (e.g. simu_st_metadata.csv
  from the Figshare deposit DOI 10.6084/m9.figshare.24452812) and need the normalized
  adjacency matrix that serves as STEM's prediction goal for spot-level spatial relationships.
when_to_use_negative:
- When you need the SC-SC pseudo-spatial adjacency between single cells — that is
  predicted by the trained STEM model from embeddings, not computed from coordinates
- When constructing the SC-ST or ST-SC mapping matrices (netst2sc / netsc2st) — those
  come from softmax over dot products of scembedding and stembedding, not from the
  Gaussian kernel
- When the ST data has no per-spot spatial coordinates (no spatial metadata CSV),
  since the kernel requires pairwise Euclidean spot distances
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: STEM
  role: The spatial adjacency matrix is the ground-truth prediction goal that supervises
    STEM's predictors; sigma is set in the STEM model config (setattr(opt, 'sigma',
    3))
  repo: https://github.com/WhirlFirst/STEM
- name: PyTorch
  role: Used to build the Gaussian kernel matrix and run STEM training on GPU tensors
    (e.g. torch.tensor(spcoor.values).float() on 'cuda:0')
  repo: https://pytorch.org/get-started/locally/
- name: scanpy
  role: AnnData container into which the spot coordinates are attached (stadata.obsm['spatial']
    = spcoor) during preprocessing
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s42003-023-05640-1
    title: STEM enables mapping of single-cell and spatial transcriptomics data with
      transfer learning
  source_content_hash: sha256:f69ce777be73bce61636771158a3aca50d3848a8400d6494260902fa17b39aa8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WhirlFirst/STEM
  tools:
  - PyTorch
  - STEM
  - scanpy
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# gaussian-kernel-spatial-smoothing

## Summary

Constructs the N x N row-normalized ground-truth ST-ST spatial adjacency matrix that supervises STEM's adjacency predictors, by applying a Gaussian kernel (W_ij = exp(-d_ij^2 / (2*sigma^2))) to pairwise Euclidean distances between ST spot coordinates.

## When to use

Use when you have ST spot spatial coordinates (e.g. simu_st_metadata.csv from the Figshare deposit DOI 10.6084/m9.figshare.24452812) and need the normalized adjacency matrix that serves as STEM's prediction goal for spot-level spatial relationships.

## When NOT to use

- When you need the SC-SC pseudo-spatial adjacency between single cells — that is predicted by the trained STEM model from embeddings, not computed from coordinates
- When constructing the SC-ST or ST-SC mapping matrices (netst2sc / netsc2st) — those come from softmax over dot products of scembedding and stembedding, not from the Gaussian kernel
- When the ST data has no per-spot spatial coordinates (no spatial metadata CSV), since the kernel requires pairwise Euclidean spot distances

## Inputs

- ST spot spatial coordinate table loaded from simu_st_metadata.csv (index_col=0), as used via stadata.obsm['spatial'] = spcoor
- Gaussian spatial variance parameter sigma (half the mean adjacent-spot ST distance; sigma=3 in the E1z2 demo)
- Number of ST spots N (defines the N x N output dimensionality)

## Outputs

- N x N row-normalized ST-ST spatial adjacency matrix saved as spatial_adjacency_matrix.npy, where each spot's adjacency probabilities sum to 1

## How to apply

Load the ST spot coordinates from the metadata CSV (spcoor) as a NumPy array and compute all pairwise Euclidean distances between spots. Set sigma to half the mean adjacent-spot distance; empirically the STEM authors found this yields the best performance, and the demo for the simulated ST data (E1z2, 351 genes) uses sigma=3 as 'the spatial variance parameter in the Gaussian function'. Build the weight matrix W with W_ij = exp(-d_ij^2 / (2*sigma^2)) using NumPy or PyTorch tensors on the GPU device configured for STEM (e.g. 'cuda:0'). Row-normalize W by dividing each row by its sum so each spot's adjacency probabilities summarize to 1, matching the convention STEM uses for its softmax-derived SC-ST mapping matrices (netsc2st). Save the result as spatial_adjacency_matrix.npy and verify every row sums to 1 within floating-point tolerance and the shape is N x N, where N is the number of ST spots.

## Related tools

- **STEM** (The spatial adjacency matrix is the ground-truth prediction goal that supervises STEM's predictors; sigma is set in the STEM model config (setattr(opt, 'sigma', 3))) — https://github.com/WhirlFirst/STEM
- **PyTorch** (Used to build the Gaussian kernel matrix and run STEM training on GPU tensors (e.g. torch.tensor(spcoor.values).float() on 'cuda:0')) — https://pytorch.org/get-started/locally/
- **scanpy** (AnnData container into which the spot coordinates are attached (stadata.obsm['spatial'] = spcoor) during preprocessing)

## Examples

```
import numpy as np; from scipy.spatial.distance import cdist; D = cdist(spcoor.values, spcoor.values); sigma = 3; W = np.exp(-D**2 / (2*sigma**2)); W = W / W.sum(axis=1, keepdims=True); np.save('spatial_adjacency_matrix.npy', W)
```

## Evaluation signals

- Shape check: the output matrix is exactly N x N, where N equals the number of rows in simu_st_metadata.csv (the ST spots)
- Row-stochasticity invariant: every row of the normalized W sums to 1 within floating-point tolerance
- Kernel range check: all W_ij values lie in (0, 1], and diagonal entries equal exactly 1 since d_ii = 0 gives exp(0) = 1 before normalization
- Sigma provenance check: sigma equals half the mean adjacent-spot distance (the demo uses sigma=3 for the E1z2 simulation with 351 genes)
- File round-trip check: np.load('spatial_adjacency_matrix.npy') reproduces the normalized matrix used as STEM's supervision target

## Limitations

- The sigma choice is empirical rather than principled: 'Empirically we found by setting the sigma as half of the ST spot adjacent distance, STEM achieves the best performance'; the sigma=3 value is tied to the simulated E1z2 dataset and must be re-derived for other tissues
- The article/README demonstrate this matrix construction only for the simulated ST data (Figshare deposit DOI 10.6084/m9.figshare.24452812); behavior for real ST platforms with non-uniform spot layouts is not demonstrated
- The Gaussian kernel assumes isotropic spatial relationships; no directional or anisotropic adjacency is modeled

## Evidence

- [readme] Empirically we found by setting the sigma as half of the ST spot adjacent distance, STEM achieves the best performance.: "Empirically we found by setting the sigma as half of the ST spot adjacent distance, STEM achieves the best performance."
- [readme] setattr(opt, 'sigma', 3)  # the spatial variance parameter in the Gaussian function: "setattr(opt, 'sigma', 3)  # the spatial variance parameter in the Gaussian function"
- [methods] The processed data and trained models used for reproducing the results are deposited in [Figshare](https://doi.org/10.6084/m9.figshare.24452812).: "The processed data and trained models used for reproducing the results are deposited in [Figshare](https://doi.org/10.6084/m9.figshare.24452812)"
