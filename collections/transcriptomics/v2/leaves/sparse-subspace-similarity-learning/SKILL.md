---
name: sparse-subspace-similarity-learning
description: Use when clustering single-cell RNA-seq expression matrices to identify
  cell types where high noise and dropout rates degrade naive pairwise similarity,
  or when tuning/validating SSRE's rho/lambda penalty ratio.
when_to_use_negative:
- Very large-scale scRNA-seq datasets where runtime is critical — the authors state
  SSRE is relatively time-consuming at scale and parallel acceleration is future work
- Single-cell multi-omics integration tasks — multi-view clustering for multi-omics
  is only proposed as a possible future trend, not implemented
- Analyses requiring incorporated biological knowledge such as gene functions or gene
  regulatory relationships — proposed only as future work in the paper
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: SSRE
  role: 'Primary method: sparse subspace representation plus similarity enhancement
    with spectral clustering for cell type detection'
  repo: https://github.com/CSUBioGroup/SSRE
- name: MATLAB
  role: MATLAB implementation of SSRE used to run the method and parameter sweeps
    on scRNA-seq datasets
  repo: https://github.com/CSUBioGroup/SSRE
- name: Python
  role: Python implementation of SSRE as an alternative runtime for the clustering
    pipeline
  repo: https://github.com/CSUBioGroup/SSRE
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.gpb.2020.09.004
    title: 'SSRE: Cell Type Detection Based on Sparse Subspace Representation and
      Similarity Enhancement'
  source_content_hash: sha256:1850be235dc33f8c309424b7d8a9429b167b8dd9d66e3fa1dc8b895ff34e2584
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
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 11
---

# sparse-subspace-similarity-learning

## Summary

SSRE identifies cell types from scRNA-seq data by computing sparse subspace representations of cell-to-cell similarity, then enhancing that similarity via adaptive gene selection and a similarity-enhancement step before spectral clustering. Its penalty coefficient ratio rho/lambda is tunable, with stable performance in the 6-20 interval and default rho/lambda = 10.

## When to use

Use when clustering single-cell RNA-seq expression matrices to identify cell types where high noise and dropout rates degrade naive pairwise similarity, or when tuning/validating SSRE's rho/lambda penalty ratio.

## When NOT to use

- Very large-scale scRNA-seq datasets where runtime is critical — the authors state SSRE is relatively time-consuming at scale and parallel acceleration is future work
- Single-cell multi-omics integration tasks — multi-view clustering for multi-omics is only proposed as a possible future trend, not implemented
- Analyses requiring incorporated biological knowledge such as gene functions or gene regulatory relationships — proposed only as future work in the paper

## Inputs

- scRNA-seq gene expression matrix (n cells x p genes, e.g. .mat files with in_X and true_labs)
- Ground-truth cell type label vector for NMI/ARI evaluation
- Ten real scRNA-seq benchmark datasets and five simulated datasets as used in the paper

## Outputs

- Sparse cell-to-cell similarity matrix from sparse subspace representation
- Cluster assignments (predicted cell type labels) per dataset
- NMI and ARI scores per (dataset, rho/lambda) pair
- Parameter-sweep table (e.g. rho_lambda_sensitivity.csv) and stability curves of NMI/ARI versus rho/lambda

## How to apply

Run SSRE (MATLAB or Python implementation from CSUBioGroup/SSRE) on a cells-by-genes scRNA-seq expression matrix: first an adaptive gene selection step reduces irrelevant features, then sparse subspace representation generates a sparse cell-to-cell similarity matrix that is enhanced by combining three classical pairwise similarities (all three used by default, though one or two can be chosen). The enhanced similarity is fed to spectral clustering to reveal block structure of the data. Compare resulting cluster assignments to ground-truth cell type labels using NMI and ARI. To validate parameter stability, sweep rho/lambda over {2, 4, 6, ..., 30} with all other parameters at defaults and confirm stable NMI/ARI in the interval 6-20, matching the paper's default rho/lambda = 10.

## Related tools

- **SSRE** (Primary method: sparse subspace representation plus similarity enhancement with spectral clustering for cell type detection) — https://github.com/CSUBioGroup/SSRE
- **MATLAB** (MATLAB implementation of SSRE used to run the method and parameter sweeps on scRNA-seq datasets) — https://github.com/CSUBioGroup/SSRE
- **Python** (Python implementation of SSRE as an alternative runtime for the clustering pipeline) — https://github.com/CSUBioGroup/SSRE

## Evaluation signals

- NMI and ARI stability curves versus rho/lambda should be flat/stable for ratios in the interval 6-20 across all ten datasets
- Performance at the default rho/lambda = 10 should match or exceed surrounding ratios and reproduce the paper's Figures 2 and S1
- SSRE should outperform several state-of-the-art single-cell clustering methods in most cases on ten real and five simulated datasets
- Predicted cluster assignments compared to true labels must yield finite NMI and ARI values, with ARI computed via RandIndex-style comparison
- Learned similarity, combined with spectral clustering, should reveal the block structure of the scRNA-seq data

## Limitations

- SSRE is relatively time-consuming on large-scale datasets; parallel computing is suggested but not implemented in this work
- Default uses all three pairwise similarities; users must explicitly choose one or two if subsets are desired
- Parameter behavior is only characterized for rho/lambda in {2, 4, ..., 30}; stability outside the 6-20 interval is not guaranteed

## Evidence

- [abstract] SSRE implementation and availability: "The matlab and python implementations of SSRE are available at https://github.com/CSUBioGroup/SSRE."
- [abstract] Benchmark scope and performance: "Tested on ten real scRNA-seq datasets and five simulated datasets, SSRE achieved the superior performance in most cases compared to several state-of-the-art single-cell clustering methods."
- [intro] Sparse subspace similarity mechanism: "SSRE computes the linear representation between cells based on sparse subspace theory, and thus generates a sparse representation of cell-to-cell similarity"
- [intro] Block structure validation signal: "when combined with spectral clustering, the learned similarities by SSRE can reveal the block structure of scRNA-seq data reliably"
- [discussion] Scalability limitation: "One limitation of SSRE is relatively time-consuming in large-scale datasets; therefore, parallel computing is a possible strategy to accelerate the framework"
- [discussion] Similarity choice flexibility: "users can choose one or two of the three pairwise similarities mentioned in this study to perform gene selection and similarity enhancement procedures, and all three are used by default"
- [readme] MATLAB evaluation workflow with NMI/ARI: [evidence span withheld — anchor status: not_found]
