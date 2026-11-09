---
name: spectral-eigendecomposition-of-similarity-matrices
description: Use when a cell-to-cell similarity or affinity matrix has been learned
  (e.g. by SSRE, SSR, MPSSC/SIMLR-style methods) and the cluster number must be estimated
  from its Laplacian eigenvalue spectrum rather than pre-annotated cell-type counts.
when_to_use_negative:
- When pre-annotated cluster numbers are available and the goal is clustering performance
  evaluation (NMI, Purity, ARI) rather than cluster-number estimation
- When the input is raw gene expression rather than a learned similarity matrix —
  SSRE first requires adaptive gene selection and similarity enhancement before spectral
  decomposition
- Very large-scale datasets where eigendecomposition is impractical — SSRE is 'relatively
  time-consuming in large-scale datasets' and parallel computing was deferred to future
  work
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: MATLAB
  role: Implementation environment for eigendecomposition of the Laplacian built from
    learned similarity matrices (SSRE and MPSSC pipelines are MATLAB-based)
  repo: https://github.com/CSUBioGroup/SSRE
- name: Python
  role: Alternative implementation language for SSRE (e.g. numpy/scipy eigendecomposition
    for the eigengap criterion)
  repo: https://github.com/CSUBioGroup/SSRE
- name: SSRE
  role: Produces the learned similarity matrices whose Laplacian eigenvalues feed
    the eigengap-based cluster-number estimate
  repo: https://github.com/CSUBioGroup/SSRE
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.gpb.2020.09.004
    title: 'SSRE: Cell Type Detection Based on Sparse Subspace Representation and
      Similarity Enhancement'
  source_content_hash: sha256:adf9e14d1f88acac18904c73f7e2343f031922f280bbbadabbb380e00a44a595
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSUBioGroup/SSRE
  tools:
  - MATLAB
  - Python
  - SSRE
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# spectral-eigendecomposition-of-similarity-matrices

## Summary

Computes the spectral (Laplacian) eigenvalue decomposition of a learned cell-to-cell similarity matrix and applies the eigengap heuristic to estimate the number of clusters in scRNA-seq data, as used with SSRE/SSR learned similarities.

## When to use

Use when a cell-to-cell similarity or affinity matrix has been learned (e.g. by SSRE, SSR, MPSSC/SIMLR-style methods) and the cluster number must be estimated from its Laplacian eigenvalue spectrum rather than pre-annotated cell-type counts.

## When NOT to use

- When pre-annotated cluster numbers are available and the goal is clustering performance evaluation (NMI, Purity, ARI) rather than cluster-number estimation
- When the input is raw gene expression rather than a learned similarity matrix — SSRE first requires adaptive gene selection and similarity enhancement before spectral decomposition
- Very large-scale datasets where eigendecomposition is impractical — SSRE is 'relatively time-consuming in large-scale datasets' and parallel computing was deferred to future work

## Inputs

- Learned cell-to-cell similarity/affinity matrix (one per dataset per method: SSRE, SSR, and four similarity-learning comparison methods)
- Pre-annotated cell-type labels or cluster counts for the ten real scRNA-seq datasets (benchmark metadata, e.g. true_labs in Data_*.mat files)
- Gene expression matrix used upstream to learn the similarities (n cells x p genes)

## Outputs

- Ordered Laplacian eigenvalue sequence per similarity matrix
- Eigengap-estimated cluster number per dataset and per method
- Consolidated comparison table of estimated vs pre-annotated cluster numbers (table_s3_cluster_number_estimation.csv, supplementary Table S3 layout)

## How to apply

Load the learned similarity matrices produced for each dataset and method (SSRE, SSR, and similarity-learning comparisons). For each matrix, form the normalized graph Laplacian — MPSSC-style pipelines build 'a normalized graph Laplacian' from the affinity matrices — and compute the ordered eigenvalue decomposition (MATLAB `eig` / Python `numpy.linalg.eigvalsh`). Estimate the cluster number as the index of the largest eigengap in the eigenvalue sequence (standard eigengap heuristic used with spectral clustering). Retrieve the pre-annotated cell-type counts for each of the ten real scRNA-seq datasets (e.g. Deng, Ting, Treutlein, Ginhoux, Buettner, Pollen, Zeisel, Tasic, Macosko) from benchmark metadata and tabulate estimated versus pre-annotated numbers per dataset and method. SSRE combines these learned similarities with spectral clustering so the learned similarities 'can reveal the block structure of scRNA-seq data reliably'; MATLAB and Python implementations are available at https://github.com/CSUBioGroup/SSRE.

## Related tools

- **MATLAB** (Implementation environment for eigendecomposition of the Laplacian built from learned similarity matrices (SSRE and MPSSC pipelines are MATLAB-based)) — https://github.com/CSUBioGroup/SSRE
- **Python** (Alternative implementation language for SSRE (e.g. numpy/scipy eigendecomposition for the eigengap criterion)) — https://github.com/CSUBioGroup/SSRE
- **SSRE** (Produces the learned similarity matrices whose Laplacian eigenvalues feed the eigengap-based cluster-number estimate) — https://github.com/CSUBioGroup/SSRE

## Examples

```
L = diag(sum(S)) - S; [V, D] = eig(L); gaps = diff(diag(D)); k_est = find(gaps == max(gaps), 1);
```

## Evaluation signals

- Eigengap-estimated cluster numbers must match the values reported in supplementary Table S3 for SSRE, SSR, and the four comparison methods across all ten real datasets
- Estimated cluster numbers should be tabulated against pre-annotated cell-type counts per dataset; systematic large deviations indicate a wrongly normalized Laplacian or misordered eigenvalues
- Eigenvalues must be sorted in ascending order before computing gaps (diff of consecutive eigenvalues); the estimate is the index of the largest gap
- Reproducibility check: re-running the decomposition on the same similarity matrix must yield identical eigenvalues (deterministic symmetric eigensolver)

## Limitations

- SSRE is relatively time-consuming on large-scale datasets, limiting eigendecomposition-based estimation there
- Reliability depends on the quality of the learned similarity matrix; noisy, high-dimensional scRNA-seq data with high dropouts can degrade the similarity and hence the eigengap signal
- The eigengap heuristic provides an estimate that may not equal the pre-annotated cluster number, so agreement must be checked dataset by dataset

## Evidence

- [abstract] The matlab and python implementations of SSRE are available at https://github.com/CSUBioGroup/SSRE.: "The matlab and python implementations of SSRE are available at https://github.com/CSUBioGroup/SSRE."
- [intro] Spectral clustering reveals block structure: "The experimental results show that when combined with spectral clustering, the learned similarities by SSRE can reveal the block structure of scRNA-seq data reliably."
- [other] Eigengap workflow from source card: [evidence span withheld — anchor status: not_found]
- [readme] Normalized graph Laplacian construction: [evidence span withheld — anchor status: not_found]
