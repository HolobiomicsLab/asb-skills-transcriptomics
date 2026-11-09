---
name: matrix-row-normalization
description: Use when you have ST spot spatial coordinates (e.g. simu_st_metadata.csv)
  and need the normalized ground-truth adjacency matrix that STEM's predictors are
  trained to reconstruct from embeddings.
when_to_use_negative:
- You need the SC-ST or ST-SC mapping matrices rather than the ground-truth spot adjacency
  — those come from softmax over embedding dot products (netst2sc/netsc2sc), not from
  coordinates.
- The spot coordinates are unavailable or unreliable (e.g. you only have expression
  matrices) — the Gaussian kernel requires true spatial coordinates.
tools:
- name: STEM
  role: Consumes the row-normalized adjacency matrix as the ground-truth target that
    its predictors reconstruct from spatially aware embeddings
  repo: https://github.com/WhirlFirst/STEM
- name: PyTorch
  role: Tensor backend used to build the Gaussian-kernel weight matrix and train STEM
    against it
- name: scanpy
  role: AnnData workflow context in which spot metadata (spatial coordinates) are
    loaded and preprocessed
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s42003-023-05640-1
    title: STEM enables mapping of single-cell and spatial transcriptomics data with
      transfer learning
  source_content_hash: sha256:3c37666fc7484d317c4b7f79a6a3a6ba8bb1a13433d66dfd1736211b47d0f842
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WhirlFirst/STEM
  tools:
  - PyTorch
  - STEM
  - scanpy
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# matrix-row-normalization

## Summary

Constructs the row-normalized N x N spatial adjacency matrix that serves as STEM's ground-truth prediction target, by applying a Gaussian kernel exp(-d_ij^2/(2*sigma^2)) to pairwise Euclidean distances between ST spot coordinates. Row-normalization makes each spot's adjacency weights sum to 1 so they act as probability distributions supervising STEM's adjacency predictors.

## When to use

Use when you have ST spot spatial coordinates (e.g. simu_st_metadata.csv) and need the normalized ground-truth adjacency matrix that STEM's predictors are trained to reconstruct from embeddings.

## When NOT to use

- You need the SC-ST or ST-SC mapping matrices rather than the ground-truth spot adjacency — those come from softmax over embedding dot products (netst2sc/netsc2sc), not from coordinates.
- The spot coordinates are unavailable or unreliable (e.g. you only have expression matrices) — the Gaussian kernel requires true spatial coordinates.

## Inputs

- NumPy array or CSV of ST spot spatial coordinates (e.g. simu_st_metadata.csv from Figshare DOI 10.6084/m9.figshare.24452812)
- Gaussian spatial variance parameter sigma (set to half the mean adjacent-spot distance; demo value 3)

## Outputs

- Row-normalized N x N spatial adjacency matrix saved as spatial_adjacency_matrix.npy, with each row summing to 1
- Validation check: every row sums to 1 within floating-point tolerance and matrix shape is N x N

## How to apply

Load the ST spot coordinates from the metadata CSV (simu_st_metadata.csv from the Figshare deposit DOI 10.6084/m9.figshare.24452812) as a NumPy array and compute pairwise Euclidean distances between spots. Set sigma to half of the mean adjacent-spot distance — the STEM demo uses sigma=3, described as 'the spatial variance parameter in the Gaussian function', and the authors found empirically that setting sigma to half the ST spot adjacent distance gives the best performance. Build the N x N weight matrix with W_ij = exp(-d_ij^2 / (2*sigma^2)) using NumPy or PyTorch. Row-normalize W by dividing each row by its sum so each spot's adjacency probabilities summarize to 1, then save the result to spatial_adjacency_matrix.npy and verify every row sum equals 1 within floating-point tolerance and the shape is N x N.

## Related tools

- **STEM** (Consumes the row-normalized adjacency matrix as the ground-truth target that its predictors reconstruct from spatially aware embeddings) — https://github.com/WhirlFirst/STEM
- **PyTorch** (Tensor backend used to build the Gaussian-kernel weight matrix and train STEM against it)
- **scanpy** (AnnData workflow context in which spot metadata (spatial coordinates) are loaded and preprocessed)

## Examples

```
import numpy as np; D = np.linalg.norm(spcoor.values[:,None]-spcoor.values[None,:],axis=-1); W = np.exp(-D**2/(2*3**2)); W = W/W.sum(1,keepdims=True); np.save('spatial_adjacency_matrix.npy',W)
```

## Evaluation signals

- Matrix shape is exactly N x N where N is the number of ST spots
- Every row sums to 1 within floating-point tolerance after row normalization
- All entries lie in [0, 1] and are symmetric (W_ij == W_ji) before row normalization
- sigma equals half the mean adjacent-spot distance computed from the coordinates (demo: sigma=3)
- Saved artifact spatial_adjacency_matrix.npy reloads and matches the in-memory matrix

## Limitations

- The choice of sigma is empirical: half the ST spot adjacent distance was found to give STEM's best performance in the demo, but other tissues/spacings may need recalibration.
- Row normalization discards each spot's total adjacency mass, so absolute proximity strength between neighbor pairs is not preserved — only relative neighbor probabilities.

## Evidence

- [readme] Empirically we found by setting the sigma as half of the ST spot adjacent distance, STEM achieves the best performance.: "Empirically we found by setting the sigma as half of the ST spot adjacent distance, STEM achieves the best performance."
- [readme] the spatial variance parameter in the Gaussian function: "setattr(opt, 'sigma', 3)  # the spatial variance parameter in the Gaussian function"
- [methods] ground-truth spatial adjacency matrix via Gaussian kernel from coordinates: [evidence span withheld — anchor status: not_found]
