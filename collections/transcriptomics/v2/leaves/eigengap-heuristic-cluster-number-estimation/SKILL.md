---
name: eigengap-heuristic-cluster-number-estimation
description: Use when a learned cell-to-cell similarity or affinity matrix is available
  and the number of cell types must be estimated without labels, or when comparing
  spectral-cluster-number estimates against pre-annotated cluster numbers.
when_to_use_negative:
- Ground-truth cell-type labels are already fixed and no cluster-number estimate is
  needed (only direct clustering metrics like NMI, Purity, ARI are required)
- The similarity matrix is not suitable for spectral decomposition (e.g. noisy, unnormalized
  input rather than the learned SSRE/SSR similarity)
- Datasets outside the ten real scRNA-seq benchmark whose pre-annotated cluster numbers
  cannot be retrieved for validation
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: SSRE
  role: Produces the learned cell-to-cell similarity matrices whose Laplacian eigenvalues
    feed the eigengap cluster-number estimation
  repo: https://github.com/CSUBioGroup/SSRE
- name: MATLAB
  role: Implementation environment for SSRE and for the eigendecomposition / eigengap
    computation on similarity matrices
  repo: https://github.com/CSUBioGroup/SSRE
- name: Python
  role: Alternative implementation environment for SSRE available for computing eigengap
    estimates
  repo: https://github.com/CSUBioGroup/SSRE
- name: MPSSC
  role: Related spectral clustering framework with similarity learning whose similarity
    matrices and clustering outputs can be compared in eigengap-based estimation workflows
  repo: https://github.com/ishspsy/project
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.gpb.2020.09.004
    title: 'SSRE: Cell Type Detection Based on Sparse Subspace Representation and
      Similarity Enhancement'
  source_content_hash: sha256:2adb17d126604a58530aaa07fa7e36b2229b1d4e6b1e91449270720b42f981a4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSUBioGroup/SSRE
  tools:
  - MATLAB
  - MPSSC
  - Python
  - SSRE
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# eigengap-heuristic-cluster-number-estimation

## Summary

Estimates the number of clusters in scRNA-seq data by applying the eigengap heuristic to the ordered eigenvalues of the graph Laplacian derived from learned cell-to-cell similarity matrices, as done in SSRE to validate cluster-number estimates against pre-annotated cell-type counts (supplementary Table S3).

## When to use

Use when a learned cell-to-cell similarity or affinity matrix is available and the number of cell types must be estimated without labels, or when comparing spectral-cluster-number estimates against pre-annotated cluster numbers.

## When NOT to use

- Ground-truth cell-type labels are already fixed and no cluster-number estimate is needed (only direct clustering metrics like NMI, Purity, ARI are required)
- The similarity matrix is not suitable for spectral decomposition (e.g. noisy, unnormalized input rather than the learned SSRE/SSR similarity)
- Datasets outside the ten real scRNA-seq benchmark whose pre-annotated cluster numbers cannot be retrieved for validation

## Inputs

- Learned cell-to-cell similarity matrix per method (SSRE, SSR, and similarity-learning comparison methods) for each of the ten real scRNA-seq datasets
- Pre-annotated cell-type labels / cluster counts from benchmark dataset metadata (e.g. in_X expression matrix with true_labs as in Data_Deng.mat-style .mat files)

## Outputs

- Per-dataset, per-method table of eigengap-estimated cluster number vs. pre-annotated cluster number (table_s3_cluster_number_estimation.csv, matching supplementary Table S3)

## How to apply

Load the learned similarity matrices produced by SSRE (and the SSR variant or comparison methods such as SIMLR) for each of the ten real scRNA-seq datasets. Compute the spectral (Laplacian) eigenvalue decomposition of each similarity matrix to obtain the ordered eigenvalues used by the eigengap criterion. Estimate the cluster number for each dataset and method as the index of the largest eigengap in the eigenvalue sequence, the standard eigengap heuristic used with spectral clustering. Retrieve the pre-annotated cell-type counts for each dataset from benchmark metadata and tabulate, per dataset and per method, the eigengap-estimated cluster number alongside the pre-annotated number, consolidating results into a Table S3-style comparison table.

## Related tools

- **SSRE** (Produces the learned cell-to-cell similarity matrices whose Laplacian eigenvalues feed the eigengap cluster-number estimation) — https://github.com/CSUBioGroup/SSRE
- **MATLAB** (Implementation environment for SSRE and for the eigendecomposition / eigengap computation on similarity matrices) — https://github.com/CSUBioGroup/SSRE
- **Python** (Alternative implementation environment for SSRE available for computing eigengap estimates) — https://github.com/CSUBioGroup/SSRE
- **MPSSC** (Related spectral clustering framework with similarity learning whose similarity matrices and clustering outputs can be compared in eigengap-based estimation workflows) — https://github.com/ishspsy/project

## Examples

```
[P] = clus_fin_update(rho, lam, lam2, eta, c, in_X, true_labs); % then eig of normalized Laplacian from P; k = argmax(diff(eigvals))
```

## Evaluation signals

- The eigengap-estimated cluster counts in table_s3_cluster_number_estimation.csv must match the values reported in supplementary Table S3 for SSRE, SSR, and the four comparison methods across all ten real datasets
- For each similarity matrix, the estimated cluster number equals the index of the largest gap in the sorted Laplacian eigenvalue sequence (verify by re-inspecting the eigengap plot)
- Estimated cluster numbers should be integers >= 1 and, for reliable methods, close to the pre-annotated cell-type counts for the ten real scRNA-seq datasets
- The output table layout must mirror supplementary Table S3 (rows: datasets; columns: methods, with estimated and pre-annotated cluster numbers)

## Limitations

- SSRE is relatively time-consuming on large-scale datasets, so eigendecomposition on very large similarity matrices may be costly; parallel computing is proposed as a possible acceleration strategy
- Eigengap estimates are reliable only relative to pre-annotated numbers; discrepancies across the ten real datasets reflect limits of the heuristic and of the learned similarities
- Results depend on the quality of the learned similarity matrix; noisy or high-dropout data can distort the eigenvalue sequence and hence the eigengap location

## Evidence

- [supplementary] Eigengap estimates compared with pre-annotated numbers: [evidence span withheld — anchor status: not_found]
- [intro] Spectral clustering with learned similarities: "The experimental results show that when combined with spectral clustering, the learned similarities by SSRE can reveal the block structure of scRNA-seq data reliably."
- [abstract] Tools for SSRE implementation: "The matlab and python implementations of SSRE are available at https://github.com/CSUBioGroup/SSRE."
- [readme] Benchmark data format with labels: [evidence span withheld — anchor status: not_found]
- [discussion] Runtime limitation: "One limitation of SSRE is relatively time-consuming in large-scale datasets; therefore, parallel computing is a possible strategy to accelerate the framework"
