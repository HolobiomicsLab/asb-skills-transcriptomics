---
name: similarity-learning-pipeline-execution
description: Use when the goal is to cluster pre-annotated or simulated scRNA-seq
  count matrices (gene-by-cell) by learning an enhanced cell-to-cell similarity matrix
  and to benchmark the clusterings with NMI/ARI against competing methods.
when_to_use_negative:
- Input is already a learned similarity/affinity matrix rather than a raw gene-by-cell
  expression matrix, since SSRE's gene selection step requires expression features
- The task is visualization or differential expression analysis rather than cell-type
  clustering, although the article notes SSRE can be extended to these
- Datasets are very large-scale, since SSRE is reported to be relatively time-consuming
  on large-scale data
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: SSRE
  role: 'Primary similarity-learning method: adaptive gene selection, similarity enhancement
    over three pairwise similarities, and spectral clustering on the learned similarity
    matrix'
  repo: https://github.com/CSUBioGroup/SSRE
- name: Splatter
  role: R package used to simulate five scRNA-seq count matrices with controlled group
    proportions, cell counts, and sparsity levels for benchmarking
- name: MPSSC
  role: Competing spectral clustering method based on multiple doubly stochastic affinity
    matrices with sparse structure, run via clus_fin_update.m on the same datasets
  repo: https://github.com/ishspsy/project
- name: SC3
  role: Competing unsupervised single-cell clustering tool (BioConductor/GitHub) run
    on the same pre-annotated matrices
  repo: https://github.com/hemberg-lab/SC3
- name: SIMLR
  role: Competing similarity-learning-based single-cell clustering method included
    in the ten-method benchmark
  repo: https://github.com/ishspsy/project
provenance:
  source_task_ids:
  - task_002
  - task_001
  source_papers:
  - doi: 10.1016/j.gpb.2020.09.004
    title: 'SSRE: Cell Type Detection Based on Sparse Subspace Representation and
      Similarity Enhancement'
  source_content_hash: sha256:6f7c7c28f184904bd088cd4794a45362e3a65c9c998cd4e6aedc3fdde195bccb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSUBioGroup/SSRE
  tools:
  - MATLAB
  - MPSSC
  - Python
  - SC3
  - SIMLR
  - SSRE
  - Splatter
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# similarity-learning-pipeline-execution

## Summary

Executes a similarity-learning single-cell clustering pipeline (SSRE-style: adaptive gene selection, similarity enhancement over three pairwise similarities, then spectral clustering) on scRNA-seq expression matrices and scores the resulting cluster assignments against known labels with NMI and ARI.

## When to use

Use when the goal is to cluster pre-annotated or simulated scRNA-seq count matrices (gene-by-cell) by learning an enhanced cell-to-cell similarity matrix and to benchmark the clusterings with NMI/ARI against competing methods.

## When NOT to use

- Input is already a learned similarity/affinity matrix rather than a raw gene-by-cell expression matrix, since SSRE's gene selection step requires expression features
- The task is visualization or differential expression analysis rather than cell-type clustering, although the article notes SSRE can be extended to these
- Datasets are very large-scale, since SSRE is reported to be relatively time-consuming on large-scale data

## Inputs

- Gene-by-cell scRNA-seq expression count matrix (e.g. .mat file with in_X, n cells x p genes)
- Ground-truth cell-type label vector (true_labs) per dataset, from pre-annotation or Splatter simulation
- Known number of cell types per dataset
- Splatter simulation parameters for synthetic benchmarks: group.prob=(0.65,0.25,0.1), nCells=1000/1000/1000/500/1500, sparsity=0.61/0.8/0.94/0.94/0.94

## Outputs

- Per-dataset cluster assignment vector from SSRE and each comparison method
- NMI and ARI scores per method per dataset
- Comparison tables mirroring Tables 2 and 3 (real data) and Tables S1 and S2 (simulated data), e.g. tables_s1_s2_reproduction.csv

## How to apply

First preprocess each dataset into a gene-by-cell expression matrix with known cell-type labels (e.g. Treutlein, Deng, Ting, Macosko, Yan, Goolam, Song/GSE85908, Engel/GSE74597, Haber/GSE92332, Vento/E-MTAB-6678, or Splatter simulations with group.prob=(0.65,0.25,0.1)). Then run SSRE from the CSUBioGroup/SSRE MATLAB/Python implementation using its two-step procedure — adaptive gene selection followed by similarity enhancement that combines three classical pairwise similarities with the sparse subspace representation — and apply spectral clustering to the learned similarity matrix to obtain cell labels. Use all three pairwise similarities by default (the article notes users may select one or two, but three is the default configuration), and supply the known number of cell types to SSRE and to each comparison method (SSR, SC, SIMLR, MPSSC, Corr, SNN-Cliq, NMF, SC3, dropClust, Seurat). Compute NMI and ARI between each method's cluster assignments and the true labels per dataset, then assemble the scores into comparison tables mirroring Tables 2, 3, S1 and S2. The rationale is that traditional clustering fails on scRNA-seq data due to high noise and dropout rates, and SSRE's sparse structure embedding over multiple similarity measurements reduces the effect of irrelevant features and improves accuracy.

## Related tools

- **SSRE** (Primary similarity-learning method: adaptive gene selection, similarity enhancement over three pairwise similarities, and spectral clustering on the learned similarity matrix) — https://github.com/CSUBioGroup/SSRE
- **Splatter** (R package used to simulate five scRNA-seq count matrices with controlled group proportions, cell counts, and sparsity levels for benchmarking)
- **MPSSC** (Competing spectral clustering method based on multiple doubly stochastic affinity matrices with sparse structure, run via clus_fin_update.m on the same datasets) — https://github.com/ishspsy/project
- **SC3** (Competing unsupervised single-cell clustering tool (BioConductor/GitHub) run on the same pre-annotated matrices) — https://github.com/hemberg-lab/SC3
- **SIMLR** (Competing similarity-learning-based single-cell clustering method included in the ten-method benchmark) — https://github.com/ishspsy/project

## Examples

```
rho=0.2; lam=0.0001; lam2=lam; eta=1; c=0.1; [P] = clus_fin_update(rho, lam, lam2, eta, c, in_X, true_labs);
```

## Evaluation signals

- CSV/table contains NMI and ARI entries for SSRE and every competing method across all ten real and five simulated datasets
- SSRE achieves best or tied-first NMI and ARI on seven of the ten real datasets, ranking second on three (NMI) and two (ARI) datasets, matching Tables 2 and 3 within numerical tolerance
- Reproduced scores on the five Splatter-simulated datasets match supplementary Tables S1 and S2 within a small numerical tolerance, with SSRE superior in the majority of cases
- Learned similarity matrix combined with spectral clustering reveals a reliable block structure of the scRNA-seq data
- Per-dataset cell-type label counts match the known number of cell types supplied to each method

## Limitations

- SSRE is relatively time-consuming on large-scale datasets; parallel computing is proposed but not implemented in this work
- Clustering requires the number of cell types to be supplied, so performance depends on knowing the true number of clusters
- Incorporation of additional biological information (gene functions, gene regulatory relationships) and multi-omics multi-view clustering are only proposed as future work

## Evidence

- [abstract] Tested on ten real scRNA-seq datasets and five simulated datasets, SSRE achieved the superior performance in most cases compared to several state-of-the-art single-cell clustering methods.: "Tested on ten real scRNA-seq datasets and five simulated datasets, SSRE achieved the superior performance in most cases compared to several"
- [intro] SSRE designs a two-step procedure: adaptive gene selection and similarity enhancement to reduce the effect of irrelevant features.: "In order to reduce the effect of irrelevant features and improve the overall accuracy, SSRE designs a two-step procedure, i.e., 1) adaptive gene selection and 2) similarity enhancement."
- [discussion] Users can choose one or two of the three pairwise similarities; all three are used by default.: "users can choose one or two of the three pairwise similarities mentioned in this study to perform gene selection and similarity enhancement procedures, and all three are used by default"
- [discussion] One limitation of SSRE is relatively time-consuming in large-scale datasets; parallel computing is a possible acceleration strategy.: "One limitation of SSRE is relatively time-consuming in large-scale datasets; therefore, parallel computing is a possible strategy to accelerate the framework"
- [readme] README example showing the concrete MATLAB invocation pattern with penalty parameters and NMI/ARI computation on the Deng dataset.: "rho=0.2; lam=0.0001; lam2=lam; eta=1; c=0.1;   


%% Run MPSSC and obtain the target matrix P
[P] = clus_fin_update(rho, lam, lam2, eta, c, in_X, true_labs);"
