---
name: scrna-seq-similarity-learning
description: Use when clustering scRNA-seq data (cells x genes matrix with ground-truth
  labels available for evaluation) and noisy/high-dropout expression makes naive cell-to-cell
  similarity unreliable; need NMI/ARI gains from similarity learning.
when_to_use_negative:
- Large-scale scRNA-seq datasets where runtime matters — the authors note SSRE is
  relatively time-consuming on large-scale data and defer acceleration via parallel
  computing to future work
- Tasks requiring multi-omics integration — multi-view clustering for single-cell
  multi-omics is only proposed as a future trend, not implemented in SSRE
- Analyses needing incorporated biological prior information (gene functions, gene
  regulatory relationships) — only suggested as future work
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_3308
tools:
- name: SSRE
  role: 'Main method: sparse subspace representation of cell-to-cell similarity plus
    adaptive gene selection and similarity enhancement, combined with spectral clustering;
    MATLAB and Python implementations provided'
  repo: https://github.com/CSUBioGroup/SSRE
- name: MATLAB
  role: Implementation environment for running the SSRE and SSR baseline pipelines
    on scRNA-seq datasets
  repo: https://github.com/CSUBioGroup/SSRE
- name: Python
  role: Alternative implementation environment for running the SSRE pipeline
  repo: https://github.com/CSUBioGroup/SSRE
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.gpb.2020.09.004
    title: 'SSRE: Cell Type Detection Based on Sparse Subspace Representation and
      Similarity Enhancement'
  source_content_hash: sha256:2521f58782316cad8c416e8b8dcbe9e2966bc2616596adff691be44305a3e4fb
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
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8888888888888888
n_claims: 9
---

# scrna-seq-similarity-learning

## Summary

Learn and enhance cell-to-cell similarity matrices for single-cell RNA-seq clustering, as in SSRE: sparse subspace representation (SSR) of cell similarity plus a similarity-enhancement strategy combining three classical pairwise similarities, followed by spectral clustering for cell-type detection.

## When to use

Use when clustering scRNA-seq data (cells x genes matrix with ground-truth labels available for evaluation) and noisy/high-dropout expression makes naive cell-to-cell similarity unreliable; need NMI/ARI gains from similarity learning.

## When NOT to use

- Large-scale scRNA-seq datasets where runtime matters — the authors note SSRE is relatively time-consuming on large-scale data and defer acceleration via parallel computing to future work
- Tasks requiring multi-omics integration — multi-view clustering for single-cell multi-omics is only proposed as a future trend, not implemented in SSRE
- Analyses needing incorporated biological prior information (gene functions, gene regulatory relationships) — only suggested as future work

## Inputs

- scRNA-seq gene expression matrix (n cells x p genes), e.g. .mat files with in_X and true_labs fields
- Ground-truth cell type labels for the cells (for NMI/ARI evaluation)
- Choice of pairwise similarities for gene selection and enhancement (default: all three)

## Outputs

- Enhanced cell-to-cell similarity matrix revealing block structure
- Cluster assignments (predicted cell type labels) from spectral clustering
- NMI and ARI scores against ground-truth labels

## How to apply

Load the scRNA-seq expression matrix (n cells x p genes) and ground-truth cell type labels, then run SSRE's two-step procedure: (1) adaptive gene selection to reduce irrelevant features, and (2) similarity enhancement, in which SSRE computes linear representation between cells based on sparse subspace theory to generate a sparse cell-to-cell similarity and incorporates three classical pairwise similarities (all three used by default, though one or two may be chosen). Combine the learned similarity with spectral clustering to assign cell types. For ablation, run the SSR baseline (sparse subspace representation similarity + spectral clustering) on the same data and compare: SSRE outperformed SSR on nine of ten real scRNA-seq datasets in both NMI and ARI. Use the MATLAB or Python implementations from the CSUBioGroup/SSRE repository; compute NMI and ARI against the provided labels to judge correctness.

## Related tools

- **SSRE** (Main method: sparse subspace representation of cell-to-cell similarity plus adaptive gene selection and similarity enhancement, combined with spectral clustering; MATLAB and Python implementations provided) — https://github.com/CSUBioGroup/SSRE
- **MATLAB** (Implementation environment for running the SSRE and SSR baseline pipelines on scRNA-seq datasets) — https://github.com/CSUBioGroup/SSRE
- **Python** (Alternative implementation environment for running the SSRE pipeline) — https://github.com/CSUBioGroup/SSRE

## Examples

```
[NMI, ~, clus_labs, ~] = calc2_nmis(max(true_labs), double(P), true_labs); ARI = RandIndex(clus_labs, true_labs)
```

## Evaluation signals

- Ablation outcome: SSRE should beat the SSR baseline on ~9 of 10 real scRNA-seq datasets in both NMI and ARI; flag any dataset where the outcome diverges
- The learned similarity matrix, combined with spectral clustering, should reveal the block structure of scRNA-seq data reliably
- Computed NMI and ARI against provided cell-type labels should match or approach the paper's reported ablation table values
- SSRE should show superior performance in most cases versus state-of-the-art single-cell clustering methods (e.g. SC3, SIMLR) on ten real and five simulated datasets

## Limitations

- SSRE is relatively time-consuming on large-scale datasets; parallel computing is only proposed as a future acceleration strategy
- Users may choose one or two of the three pairwise similarities, but all three are used by default; results may vary with this choice
- Incorporating gene functions or gene regulatory relationships, and multi-view clustering for multi-omics, remain unimplemented future directions

## Evidence

- [intro] SSRE computes linear representation between cells based on sparse subspace theory: "SSRE computes the linear representation between cells based on sparse subspace theory, and thus generates a sparse representation of cell-to-cell similarity"
- [intro] two-step procedure of adaptive gene selection and similarity enhancement: "In order to reduce the effect of irrelevant features and improve the overall accuracy, SSRE designs a two-step procedure, i.e., 1) adaptive gene selection and 2) similarity enhancement."
- [intro] learned similarities combined with spectral clustering reveal block structure: "The experimental results show that when combined with spectral clustering, the learned similarities by SSRE can reveal the block structure of scRNA-seq data reliably."
- [abstract] MATLAB and Python implementations available at GitHub: "The matlab and python implementations of SSRE are available at https://github.com/CSUBioGroup/SSRE."
- [discussion] users can choose one or two of the three pairwise similarities; all three by default: "users can choose one or two of the three pairwise similarities mentioned in this study to perform gene selection and similarity enhancement procedures, and all three are used by default"
- [discussion] limitation: time-consuming on large-scale datasets; parallel computing as future acceleration: "One limitation of SSRE is relatively time-consuming in large-scale datasets; therefore, parallel computing is a possible strategy to accelerate the framework"
- [readme] README pattern for evaluating clustering with NMI, Purity, and ARI on a .mat dataset: "% Compute ARI
ARI=RandIndex(clus_labs,true_labs)

%%% Final output: Performances (three measures) of MPSSC
[NMI, Purity, ARI]"
