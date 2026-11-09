---
name: spatial-transcriptomics-coordinate-processing
description: Use when spot spatial coordinates (e.g. simu_st_metadata.csv from the
  Figshare DOI 10.6084/m9.figshare.24452812 deposit) are available and an N x N ground-truth
  adjacency target is needed for STEM's spatial-relationship reconstruction.
when_to_use_negative:
- You need SC-SC pseudo-spatial adjacency between single cells — that is predicted
  from STEM embeddings (netst2sc/netsc2st softmax mapping), not derived directly from
  spot coordinates
- The input lacks spot coordinate metadata (only expression counts are available),
  so no Euclidean spot distances can be computed
- You are building the SC-ST mapping matrix from embeddings (F.softmax(scembedding.mm(stembedding.t()),dim=1))
  rather than the ground-truth spot-level target
tools:
- name: STEM
  role: Method whose spatial-adjacency predictor consumes this Gaussian-kernel ground-truth
    matrix; sigma is set as half the ST spot adjacent distance
  repo: https://github.com/WhirlFirst/STEM
- name: PyTorch
  role: Backend used to compute the Gaussian kernel weights and hold coordinates as
    tensors during training (torch.tensor(spcoor.values).float())
- name: scanpy
  role: AnnData container used to attach spot coordinates via stadata.obsm['spatial']
    = spcoor during preprocessing
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s42003-023-05640-1
    title: STEM enables mapping of single-cell and spatial transcriptomics data with
      transfer learning
  source_content_hash: sha256:3e5f9c3d80c1188c1fc65a4fec047fa8b9f5a0499f55f4052be1b8b5789ce5e0
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
yes_rate: 0.5555555555555556
n_claims: 9
---

# spatial-transcriptomics-coordinate-processing

## Summary

Constructs the normalized ground-truth ST-ST spatial adjacency matrix that supervises STEM's predictors: pairwise Euclidean spot distances are converted to Gaussian-kernel weights (W_ij = exp(-d_ij^2 / (2*sigma^2))) and row-normalized so each spot's adjacency probabilities summarize to 1.

## When to use

Use when spot spatial coordinates (e.g. simu_st_metadata.csv from the Figshare DOI 10.6084/m9.figshare.24452812 deposit) are available and an N x N ground-truth adjacency target is needed for STEM's spatial-relationship reconstruction.

## When NOT to use

- You need SC-SC pseudo-spatial adjacency between single cells — that is predicted from STEM embeddings (netst2sc/netsc2st softmax mapping), not derived directly from spot coordinates
- The input lacks spot coordinate metadata (only expression counts are available), so no Euclidean spot distances can be computed
- You are building the SC-ST mapping matrix from embeddings (F.softmax(scembedding.mm(stembedding.t()),dim=1)) rather than the ground-truth spot-level target

## Inputs

- ST spot spatial coordinates from a metadata CSV (e.g. simu_st_metadata.csv with index_col=0, from Figshare DOI 10.6084/m9.figshare.24452812)
- Number of ST spots N (to validate the N x N output shape)

## Outputs

- Row-normalized N x N ST-ST spatial adjacency weight matrix (W_ij = exp(-d_ij^2/(2*sigma^2)), rows summing to 1), e.g. saved as spatial_adjacency_matrix.npy

## How to apply

Load the ST spot spatial coordinates from the metadata CSV (simu_st_metadata.csv, indexed by spot) as a NumPy array, and compute pairwise Euclidean distances between spots. Derive the mean adjacent-spot distance and set sigma to half of it — the demo uses sigma=3, described as 'the spatial variance parameter in the Gaussian function', and the README notes this empirically gives STEM's best performance. Build the N x N weight matrix with W_ij = exp(-d_ij^2 / (2*sigma^2)) using NumPy/PyTorch, then row-normalize W by dividing each row by its sum. Save the normalized matrix (e.g. spatial_adjacency_matrix.npy) and verify every row sums to 1 within floating-point tolerance and the shape is N x N. This matrix is the prediction goal from which STEM's predictors learn gene-spatial relations for SC-SC pseudo-adjacency inference.

## Related tools

- **STEM** (Method whose spatial-adjacency predictor consumes this Gaussian-kernel ground-truth matrix; sigma is set as half the ST spot adjacent distance) — https://github.com/WhirlFirst/STEM
- **PyTorch** (Backend used to compute the Gaussian kernel weights and hold coordinates as tensors during training (torch.tensor(spcoor.values).float()))
- **scanpy** (AnnData container used to attach spot coordinates via stadata.obsm['spatial'] = spcoor during preprocessing)

## Examples

```
import numpy as np; from scipy.spatial.distance import cdist; D = cdist(coor, coor, 'euclidean'); W = np.exp(-D**2/(2*3**2)); Wn = W / W.sum(axis=1, keepdims=True); np.save('spatial_adjacency_matrix.npy', Wn)
```

## Evaluation signals

- Output matrix shape is N x N where N is the number of ST spots
- Every row of the normalized matrix sums to 1 within floating-point tolerance
- sigma equals half the mean adjacent-spot distance (demo value: sigma=3)
- Pre-normalization weight matrix is symmetric (d_ij = d_ji); row-normalization only rescales rows
- Each row's largest weight corresponds to the spatially nearest neighbor spot

## Limitations

- The demo sigma=3 is dataset-specific; README guidance is to set sigma as half the ST spot adjacent distance, which must be recomputed per dataset
- Row normalization makes the matrix asymmetric, so it encodes directed adjacency probabilities rather than a symmetric similarity graph
- The skill requires reliable spot coordinates; coordinate errors propagate directly into the supervision target

## Evidence

- [readme] sigma as half of the ST spot adjacent distance: "Empirically we found by setting the sigma as half of the ST spot adjacent distance, STEM achieves the best performance."
- [readme] Gaussian spatial variance parameter: "setattr(opt, 'sigma', 3)  # the spatial variance parameter in the Gaussian function"
- [readme] spot coordinates attached to AnnData: "stadata.obsm['spatial'] = spcoor"
- [readme] coordinates as training tensors: "torch.tensor(spcoor.values).float())"
- [readme] data source for reproducing results: "The processed data and trained models used for reproducing the results are deposited in [Figshare](https://doi.org/10.6084/m9.figshare.24452812)."
- [discussion] learning gene-spatial relations: "The key is to infer the SC–SC spatial adjacency and SC-ST mapping by learning gene-spatial relations from the ST data."
