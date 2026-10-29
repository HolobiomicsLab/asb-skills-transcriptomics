---
name: estimated-versus-annotated-cluster-comparison-across-methods
description: Use when benchmarking similarity-learning clustering methods (e.g., SSRE
  vs. SSR and four comparison methods) and you must judge how reliably each method
  recovers the known number of cell types on pre-annotated scRNA-seq data.
when_to_use_negative:
- When evaluating clustering quality metrics (NMI, Purity, ARI) rather than the number
  of clusters — use metric-based evaluation instead
- On simulated datasets without pre-annotated cluster numbers, since the comparison
  is anchored to pre-annotated cell-type counts
- When the clustering method does not produce a similarity/affinity matrix amenable
  to Laplacian eigendecomposition
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_2229
tools:
- name: MATLAB
  role: Implementation language for SSRE and for computing Laplacian eigendecompositions
    and eigengap estimates on the learned similarity matrices
  repo: https://github.com/CSUBioGroup/SSRE
- name: Python
  role: Alternative implementation language of SSRE, usable to compute eigenvalues
    and assemble the comparison table
  repo: https://github.com/CSUBioGroup/SSRE
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.gpb.2020.09.004
    title: 'SSRE: Cell Type Detection Based on Sparse Subspace Representation and
      Similarity Enhancement'
  source_content_hash: sha256:ac04b2495bfb41d9eb93a8af200f25244add7618306e335d28427e4768229eb1
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSUBioGroup/SSRE
  tools:
  - MATLAB
  - Python
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# estimated-versus-annotated-cluster-comparison-across-methods

## Summary

Estimates the number of clusters for each learned similarity matrix via the spectral eigengap heuristic and tabulates the estimates against pre-annotated cell-type counts across ten real scRNA-seq datasets and multiple similarity-learning methods, as done for supplementary Table S3 of SSRE.

## When to use

Use when benchmarking similarity-learning clustering methods (e.g., SSRE vs. SSR and four comparison methods) and you must judge how reliably each method recovers the known number of cell types on pre-annotated scRNA-seq data.

## When NOT to use

- When evaluating clustering quality metrics (NMI, Purity, ARI) rather than the number of clusters — use metric-based evaluation instead
- On simulated datasets without pre-annotated cluster numbers, since the comparison is anchored to pre-annotated cell-type counts
- When the clustering method does not produce a similarity/affinity matrix amenable to Laplacian eigendecomposition

## Inputs

- Learned cell-to-cell similarity matrices from SSRE and the SSR variant for each of the ten real scRNA-seq datasets
- Learned similarity matrices from four similarity-learning comparison methods
- Pre-annotated cell-type counts (ground-truth cluster numbers) from benchmark dataset metadata

## Outputs

- Per-dataset, per-method table of eigengap-estimated cluster numbers versus pre-annotated cluster numbers (table_s3_cluster_number_estimation.csv, in supplementary Table S3 layout)

## How to apply

Load the learned cell-to-cell similarity matrices produced by SSRE, the SSR variant, and the four similarity-learning comparison methods for each of the ten real scRNA-seq datasets. For each similarity matrix, compute the spectral (Laplacian) eigenvalue decomposition to obtain the ordered eigenvalues, then estimate the cluster number as the index of the largest eigengap in the eigenvalue sequence — the standard eigengap heuristic used with spectral clustering. Retrieve the pre-annotated cell-type counts from the benchmark metadata and tabulate, per dataset and per method, the eigengap-estimated cluster number beside the pre-annotated number. Consolidate into a single CSV laid out like supplementary Table S3 so estimates can be compared across SSRE, SSR, and competitors. The rationale is that SSRE combines learned similarities with spectral clustering, so the eigenstructure of the learned similarity matrix should reveal the block structure of scRNA-seq data reliably.

## Related tools

- **MATLAB** (Implementation language for SSRE and for computing Laplacian eigendecompositions and eigengap estimates on the learned similarity matrices) — https://github.com/CSUBioGroup/SSRE
- **Python** (Alternative implementation language of SSRE, usable to compute eigenvalues and assemble the comparison table) — https://github.com/CSUBioGroup/SSRE

## Examples

```
[eigvals] = eig(L); gaps = abs(diff(sort(diag(eigvals),'descend'))); [~, k_est] = max(gaps(1:20)); % eigengap cluster-number estimate
```

## Evaluation signals

- The estimated-versus-pre-annotated cluster counts in the output table must match the values reported in supplementary Table S3 for SSRE, SSR, and the four comparison methods across all ten real datasets
- Each eigengap-estimated cluster number equals the index of the largest gap in the ordered Laplacian eigenvalue sequence of the corresponding similarity matrix
- The output CSV contains one row per dataset-method pair for all ten real scRNA-seq datasets and six methods (SSRE, SSR, plus four similarity-learning methods)
- Estimated cluster numbers are positive integers and are comparable in magnitude to the pre-annotated cell-type counts

## Limitations

- SSRE is relatively time-consuming on large-scale datasets, which limits this eigengap comparison on very large scRNA-seq data
- Reliability is assessed against pre-annotated cluster numbers, so datasets without accurate ground-truth annotations cannot be judged this way
- The eigengap criterion is only as good as the learned similarity matrix; noisy similarities may yield eigengap estimates that deviate from pre-annotated counts

## Evidence

- [supplementary] eigengap estimates from SSRE and SSR were compared with pre-annotated numbers, alongside four other similarity-learning methods, with results summarized in Table S3.: [evidence span withheld — anchor status: not_found]
- [intro] The experimental results show that when combined with spectral clustering, the learned similarities by SSRE can reveal the block structure of scRNA-seq data reliably.: "The experimental results show that when combined with spectral clustering, the learned similarities by SSRE can reveal the block structure of scRNA-seq data reliably."
- [abstract] The matlab and python implementations of SSRE are available at https://github.com/CSUBioGroup/SSRE.: "The matlab and python implementations of SSRE are available at https://github.com/CSUBioGroup/SSRE."
- [abstract] Tested on ten real scRNA-seq datasets and five simulated datasets, SSRE achieved the superior performance in most cases compared to several state-of-the-art single-cell clustering methods.: "Tested on ten real scRNA-seq datasets and five simulated datasets, SSRE achieved the superior performance in most cases compared to several state-of-the-art single-cell clustering methods."
