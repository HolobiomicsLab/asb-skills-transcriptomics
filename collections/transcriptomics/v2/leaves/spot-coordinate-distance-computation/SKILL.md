---
name: spot-coordinate-distance-computation
description: Use when ST data provides spot spatial coordinates (e.g. simu_st_metadata.csv
  from the Figshare deposit DOI 10.6084/m9.figshare.24452812) and you need the N x
  N row-normalized spatial adjacency matrix as a prediction target.
when_to_use_negative:
- Input coordinates are inferred single-cell coordinates (e.g. adata.obsm['spatialDA']
  from the SC-ST mapping) rather than actual ST spot coordinates from the metadata
  file
- The ST data has no spot-level spatial coordinate metadata, so Euclidean distances
  between spots cannot be computed
- You need the SC-SC pseudo-spatial adjacency (predicted from embeddings) rather than
  the spot-level ground-truth adjacency matrix
tools:
- name: PyTorch
  role: Compute the Gaussian-kernel weight matrix W_ij = exp(-d_ij^2 / (2*sigma^2))
    from spot coordinates as tensors
- name: NumPy
  role: Load spot coordinates from the metadata CSV and build/row-normalize the pairwise
    distance weight matrix
- name: scanpy
  role: Handles the AnnData objects whose obsm['spatial'] slot carries the ST spot
    coordinates used for distance computation
- name: STEM
  role: Model whose spatial adjacency prediction objective is supervised by the matrix
    this skill produces
  repo: https://github.com/WhirlFirst/STEM
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s42003-023-05640-1
    title: STEM enables mapping of single-cell and spatial transcriptomics data with
      transfer learning
  source_content_hash: sha256:5522ca5a55526a25dbed0feabc0196412dc311f19fef6b63260e65254529c913
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WhirlFirst/STEM
  tools:
  - NumPy
  - PyTorch
  - STEM
  - scanpy
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# spot-coordinate-distance-computation

## Summary

Constructs the normalized ground-truth spot-spot spatial adjacency matrix that supervises STEM's spatial-relationship predictors by applying a Gaussian kernel to pairwise Euclidean distances between ST spot coordinates.

## When to use

Use when ST data provides spot spatial coordinates (e.g. simu_st_metadata.csv from the Figshare deposit DOI 10.6084/m9.figshare.24452812) and you need the N x N row-normalized spatial adjacency matrix as a prediction target.

## When NOT to use

- Input coordinates are inferred single-cell coordinates (e.g. adata.obsm['spatialDA'] from the SC-ST mapping) rather than actual ST spot coordinates from the metadata file
- The ST data has no spot-level spatial coordinate metadata, so Euclidean distances between spots cannot be computed
- You need the SC-SC pseudo-spatial adjacency (predicted from embeddings) rather than the spot-level ground-truth adjacency matrix

## Inputs

- ST spot spatial coordinates table (e.g. simu_st_metadata.csv from Figshare deposit DOI 10.6084/m9.figshare.24452812), loaded as a NumPy array
- Gaussian spatial variance parameter sigma (set to half the mean adjacent-spot distance; demo value sigma=3)

## Outputs

- N x N row-normalized spatial adjacency matrix saved as spatial_adjacency_matrix.npy, with each row summing to 1

## How to apply

Load the ST spot spatial coordinates from the metadata CSV (e.g. simu_st_metadata.csv) as a NumPy array and compute all pairwise Euclidean distances between spots. Derive the mean adjacent-spot distance and set sigma to half of it — the STEM README states this empirically gives the best performance, and its demo uses sigma=3 as 'the spatial variance parameter in the Gaussian function'. Build the N x N weight matrix with W_ij = exp(-d_ij^2 / (2*sigma^2)) in NumPy or PyTorch, then row-normalize W by dividing each row by its sum so each spot's adjacency probabilities summarize to 1. Save the result as spatial_adjacency_matrix.npy and verify every row sums to 1 within floating-point tolerance and the shape is N x N.

## Related tools

- **PyTorch** (Compute the Gaussian-kernel weight matrix W_ij = exp(-d_ij^2 / (2*sigma^2)) from spot coordinates as tensors)
- **NumPy** (Load spot coordinates from the metadata CSV and build/row-normalize the pairwise distance weight matrix)
- **scanpy** (Handles the AnnData objects whose obsm['spatial'] slot carries the ST spot coordinates used for distance computation)
- **STEM** (Model whose spatial adjacency prediction objective is supervised by the matrix this skill produces) — https://github.com/WhirlFirst/STEM

## Examples

```
D = torch.cdist(coords, coords); W = torch.exp(-(D**2)/(2*sigma**2)); W = W / W.sum(dim=1, keepdim=True); np.save('spatial_adjacency_matrix.npy', W.numpy())
```

## Evaluation signals

- Output matrix shape is exactly N x N where N is the number of ST spots
- Every row of the normalized matrix sums to 1 within floating-point tolerance
- sigma is set to half the mean adjacent-spot distance (the demo uses sigma=3 as the spatial variance parameter in the Gaussian function)
- The pre-normalization kernel matrix is symmetric with values in (0, 1], peaking near 1 for close spots and decaying with distance
- Saved artifact spatial_adjacency_matrix.npy loads and matches the in-memory matrix

## Limitations

- sigma choice is empirical: STEM's authors found best performance by setting sigma to half of the ST spot adjacent distance, so results are sensitive to this heuristic
- Row normalization means the matrix encodes relative adjacency probabilities per spot, not absolute pairwise similarities

## Evidence

- [methods] The ground-truth spatial adjacency matrix that supervises STEM's predictors is calculated directly from the spatial coordinates of the ST data via a Gaussian kernel.: [evidence span withheld — anchor status: not_found]
- [readme] sigma heuristic: "Empirically we found by setting the sigma as half of the ST spot adjacent distance, STEM achieves the best performance."
- [readme] sigma=3 demo parameter: "setattr(opt, 'sigma', 3)  # the spatial variance parameter in the Gaussian function"
