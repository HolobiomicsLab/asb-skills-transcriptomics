---
name: single-cell-rnaseq-cluster-count-benchmarking
description: Use when a spectral-clustering similarity matrix exists for a scRNA-seq
  dataset and you must determine or benchmark the number of clusters without using
  ground-truth labels for fitting.
when_to_use_negative:
- When ground-truth cluster numbers can be used directly for parameter setting — this
  skill estimates cluster count without labels and is for benchmarking estimation
  reliability
- When the input is raw gene expression rather than a learned cell-to-cell similarity
  matrix — SSRE's two-step gene-selection and similarity-enhancement procedure must
  run first
- For very large-scale datasets where SSRE is too slow without parallelization — the
  article notes SSRE is relatively time-consuming on large-scale data
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3170
tools:
- name: SSRE
  role: Provides the learned similarity matrices whose Laplacian spectra feed the
    eigengap cluster-number estimation; MATLAB and Python implementations available
  repo: https://github.com/CSUBioGroup/SSRE
- name: MPSSC (ishspsy/project)
  role: MATLAB framework for spectral clustering with learned similarity matrices
    (MPSSC/PSSC variants), used to obtain similarity structures for eigengap-based
    cluster number comparison
  repo: https://github.com/ishspsy/project
- name: SC3
  role: R/Bioconductor tool for unsupervised clustering of single-cell RNA-Seq data,
    part of the clustering comparison landscape
  repo: https://github.com/hemberg-lab/SC3
- name: MATLAB
  role: Implementation environment for SSRE and for the eigengap computation on similarity
    matrices
  repo: https://github.com/CSUBioGroup/SSRE
- name: Python
  role: Implementation environment for SSRE used to reproduce similarity learning
    and eigengap estimation
  repo: https://github.com/CSUBioGroup/SSRE
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.gpb.2020.09.004
    title: 'SSRE: Cell Type Detection Based on Sparse Subspace Representation and
      Similarity Enhancement'
  source_content_hash: sha256:92b87bdb907cab32e6552e668ed83631b225c3a1f401d5f4623da94c97580790
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSUBioGroup/SSRE
  tools:
  - MATLAB
  - MPSSC (ishspsy/project)
  - Python
  - SC3
  - SSRE
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# single-cell-rnaseq-cluster-count-benchmarking

## Summary

Estimates the number of cell types in scRNA-seq data via the eigengap heuristic applied to the Laplacian eigenvalue spectrum of learned similarity matrices (SSRE, SSR, and comparison methods), then benchmarks those estimates against pre-annotated cluster numbers, as consolidated in supplementary Table S3.

## When to use

Use when a spectral-clustering similarity matrix exists for a scRNA-seq dataset and you must determine or benchmark the number of clusters without using ground-truth labels for fitting.

## When NOT to use

- When ground-truth cluster numbers can be used directly for parameter setting — this skill estimates cluster count without labels and is for benchmarking estimation reliability
- When the input is raw gene expression rather than a learned cell-to-cell similarity matrix — SSRE's two-step gene-selection and similarity-enhancement procedure must run first
- For very large-scale datasets where SSRE is too slow without parallelization — the article notes SSRE is relatively time-consuming on large-scale data

## Inputs

- Learned similarity matrices (SSRE, SSR, and four similarity-learning comparison methods) for each of the ten real scRNA-seq datasets
- Pre-annotated cell-type counts / ground-truth labels from benchmark metadata for each dataset

## Outputs

- Per-dataset, per-method eigengap-estimated cluster number alongside the pre-annotated cluster number
- Consolidated comparison table: table_s3_cluster_number_estimation.csv, laid out like supplementary Table S3

## How to apply

For each learned similarity matrix (SSRE, the SSR variant, and the four similarity-learning comparison methods on each of the ten real scRNA-seq datasets), compute the spectral decomposition of the normalized graph Laplacian to obtain ordered eigenvalues, then estimate the cluster number as the index of the largest eigengap in the eigenvalue sequence. SSRE learns cell-to-cell similarity by computing linear representations under sparse subspace theory, applying adaptive gene selection and similarity enhancement, and combining three pairwise similarities (all three by default); this learned similarity is combined with spectral clustering to reveal block structure. Retrieve the pre-annotated cell-type counts for each dataset from benchmark metadata, tabulate eigengap-estimated versus pre-annotated cluster numbers per dataset and method, and write table_s3_cluster_number_estimation.csv matching the layout of supplementary Table S3. Validation compares SSRE against SSR and the four comparison methods across all ten real datasets.

## Related tools

- **SSRE** (Provides the learned similarity matrices whose Laplacian spectra feed the eigengap cluster-number estimation; MATLAB and Python implementations available) — https://github.com/CSUBioGroup/SSRE
- **MPSSC (ishspsy/project)** (MATLAB framework for spectral clustering with learned similarity matrices (MPSSC/PSSC variants), used to obtain similarity structures for eigengap-based cluster number comparison) — https://github.com/ishspsy/project
- **SC3** (R/Bioconductor tool for unsupervised clustering of single-cell RNA-Seq data, part of the clustering comparison landscape) — https://github.com/hemberg-lab/SC3
- **MATLAB** (Implementation environment for SSRE and for the eigengap computation on similarity matrices) — https://github.com/CSUBioGroup/SSRE
- **Python** (Implementation environment for SSRE used to reproduce similarity learning and eigengap estimation) — https://github.com/CSUBioGroup/SSRE

## Examples

```
import numpy as np; vals = np.linalg.eigvalsh(L); k = int(np.argmax(np.diff(vals[:20]))) + 1
```

## Evaluation signals

- Estimated-versus-pre-annotated cluster counts in table_s3_cluster_number_estimation.csv must match supplementary Table S3 values for SSRE, SSR, and the four comparison methods across all ten real datasets
- Each dataset must yield exactly one estimated cluster number per method, equal to the index of the largest eigengap in the ordered Laplacian eigenvalue sequence
- Output table must contain all ten real scRNA-seq datasets and all six methods (SSRE, SSR, plus four similarity-learning comparisons)
- Pre-annotated cluster numbers must agree with the benchmark metadata for each dataset
- Downstream sanity check: spectral clustering on the learned similarity using the estimated cluster number should reliably reveal the block structure of the scRNA-seq data

## Limitations

- SSRE is relatively time-consuming on large-scale datasets; parallel computing is suggested as a future acceleration strategy but was not implemented
- Eigengap estimates may deviate from pre-annotated cluster numbers, which is why reliability must be assessed against benchmark metadata rather than assumed
- Users can choose one or two of the three pairwise similarities for gene selection and similarity enhancement, so results depend on the similarity configuration (all three are used by default)

## Evidence

- [supplementary] eigengap estimates compared with pre-annotated numbers via Table S3: [evidence span withheld — anchor status: not_found]
- [intro] spectral clustering on learned similarities reveals block structure: "The experimental results show that when combined with spectral clustering, the learned similarities by SSRE can reveal the block structure of scRNA-seq data reliably."
- [abstract] SSRE MATLAB and Python implementations: "The matlab and python implementations of SSRE are available at https://github.com/CSUBioGroup/SSRE."
- [abstract] benchmark on ten real and five simulated datasets: "Tested on ten real scRNA-seq datasets and five simulated datasets, SSRE achieved the superior performance in most cases"
- [readme] MPSSC constructs normalized graph Laplacian from similarity matrices: [evidence span withheld — anchor status: not_found]
