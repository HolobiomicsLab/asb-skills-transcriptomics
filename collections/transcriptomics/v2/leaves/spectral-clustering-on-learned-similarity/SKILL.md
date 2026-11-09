---
name: spectral-clustering-on-learned-similarity
description: Use when you have a gene-by-cell scRNA-seq expression matrix with known
  number of cell types and need cell-type clusters whose quality depends on an engineered/learned
  similarity matrix rather than raw expression distances.
when_to_use_negative:
- 'The data are large-scale and runtime matters: SSRE/similarity-learning spectral
  clustering is acknowledged as relatively time-consuming on large-scale datasets'
- The number of cell types is unknown and cannot be supplied, since spectral clustering
  requires the target cluster count
- You only need off-the-shelf clustering on raw expression without similarity learning
  — simpler pipelines (e.g. Seurat) may suffice
edam_operation: http://edamontology.org/operation_3935
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SSRE
  role: Produces the learned similarity matrix (adaptive gene selection + similarity
    enhancement) that is then clustered by spectral clustering
  repo: https://github.com/CSUBioGroup/SSRE
- name: MPSSC
  role: Alternative similarity-learning spectral clustering framework using multiple
    doubly stochastic affinity matrices and ADMM; provides runnable MATLAB pipeline
    and example data
  repo: https://github.com/ishspsy/project
- name: MATLAB
  role: Implementation language for SSRE and MPSSC pipelines
- name: SC3
  role: Comparison method (BioConductor tool for unsupervised clustering of scRNA-seq
    cells) benchmarked against the learned-similarity approach
  repo: https://github.com/hemberg-lab/SC3
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.gpb.2020.09.004
    title: 'SSRE: Cell Type Detection Based on Sparse Subspace Representation and
      Similarity Enhancement'
  source_content_hash: sha256:d9ee2c67217b46bb4d0e78193c178009a0c93cb936ec8d2b24a5cf024a26f75b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSUBioGroup/SSRE
  tools:
  - MATLAB
  - MPSSC
  - Python
  - SC3
  - SSRE
  edam_operation: http://edamontology.org/operation_3935
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# spectral-clustering-on-learned-similarity

## Summary

Run spectral clustering on a learned cell-to-cell similarity matrix (e.g. produced by SSRE's two-step procedure of adaptive gene selection and similarity enhancement, or MPSSC's doubly stochastic affinity matrices) to identify cell types from scRNA-seq data and reveal the block structure reliably.

## When to use

Use when you have a gene-by-cell scRNA-seq expression matrix with known number of cell types and need cell-type clusters whose quality depends on an engineered/learned similarity matrix rather than raw expression distances.

## When NOT to use

- The data are large-scale and runtime matters: SSRE/similarity-learning spectral clustering is acknowledged as relatively time-consuming on large-scale datasets
- The number of cell types is unknown and cannot be supplied, since spectral clustering requires the target cluster count
- You only need off-the-shelf clustering on raw expression without similarity learning — simpler pipelines (e.g. Seurat) may suffice

## Inputs

- Gene-by-cell scRNA-seq expression matrix with ground-truth cell-type labels (e.g. in_X as n-by-p matrix plus true_labs, as in Data_Deng.mat)
- Known number of cell types (used to set k for spectral clustering)
- Optionally a precomputed learned similarity matrix from SSRE's gene selection + similarity enhancement or MPSSC's ADMM steps

## Outputs

- Cluster assignments for each cell (clus_labs) over the learned similarity matrix
- Clustering agreement metrics: NMI and ARI (and optionally purity) against pre-annotated labels, aggregated into a dataset x method table (e.g. 10 datasets x 11 methods)
- The learned/enhanced similarity matrix whose block structure reflects cell-type structure

## How to apply

First construct the similarity matrix: with SSRE, perform adaptive gene selection to reduce irrelevant features, then apply similarity enhancement that combines the sparse subspace representation of cell-to-cell similarity with three classical pairwise similarities (users may choose one or two, but all three are used by default). Then feed the learned similarity matrix to spectral clustering to obtain cluster assignments. With MPSSC, construct multiple doubly stochastic affinity matrices via Gaussian kernels, impose a sparse structure on the target matrix with ADMM (rho=0.2, lam=0.0001, lam2=lam, eta=1, c=0.1 in the Deng example), then spectral clustering. Evaluate agreement with pre-annotated cell-type labels using NMI and ARI (optionally purity), as in the 10-dataset x 11-method benchmark tables.

## Related tools

- **SSRE** (Produces the learned similarity matrix (adaptive gene selection + similarity enhancement) that is then clustered by spectral clustering) — https://github.com/CSUBioGroup/SSRE
- **MPSSC** (Alternative similarity-learning spectral clustering framework using multiple doubly stochastic affinity matrices and ADMM; provides runnable MATLAB pipeline and example data) — https://github.com/ishspsy/project
- **MATLAB** (Implementation language for SSRE and MPSSC pipelines)
- **SC3** (Comparison method (BioConductor tool for unsupervised clustering of scRNA-seq cells) benchmarked against the learned-similarity approach) — https://github.com/hemberg-lab/SC3

## Examples

```
load('Data_Deng.mat'); rho=0.2; lam=0.0001; lam2=lam; eta=1; c=0.1; [P]=clus_fin_update(rho,lam,lam2,eta,c,in_X,true_labs); [NMI,~,clus_labs,~]=calc2_nmis(max(true_labs),double(P),true_labs); ARI=RandIndex(clus_labs,true_labs);
```

## Evaluation signals

- Reproduced NMI and ARI for SSRE and each comparison method match the entries of the 10-dataset x 11-method Tables 2 and 3 within a small numerical tolerance
- The learned similarity matrix shows clear block structure by cell type when visualized (blocks correspond to pre-annotated cell types)
- SSRE achieves the best or tied-first NMI and ARI on seven of the ten real datasets and ranks second on three (NMI) and two (ARI) datasets
- For MPSSC, running the example yields NMI, Purity, and ARI outputs for the Deng dataset matching the published figures using rho=0.2, lam=0.0001, eta=1, c=0.1
- Sanity check: passing ground-truth labels through calc2_nmis and RandIndex yields NMI=1 and ARI=1 on any dataset

## Limitations

- SSRE is relatively time-consuming on large-scale datasets; parallel computing is proposed but not implemented
- Incorporating additional biological information (gene functions, gene regulatory relationships) is only proposed as future work
- MPSSC's large-scale scRNA-seq experiments required a computing cluster with 6 CPUs and 800 GB of memory; most runs were on 2.7 GHz / 8 GB machines
- Existing similarity computations are sensitive to noise in high-dimensional data, which is the motivation for gene selection and similarity enhancement

## Evidence

- [intro] The experimental results show that when combined with spectral clustering, the learned similarities by SSRE can reveal the block structure of scRNA-seq data reliably.: "The experimental results show that when combined with spectral clustering, the learned similarities by SSRE can reveal the block structure of scRNA-seq data reliably."
- [intro] SSRE designs a two-step procedure, i.e., 1) adaptive gene selection and 2) similarity enhancement.: "In order to reduce the effect of irrelevant features and improve the overall accuracy, SSRE designs a two-step procedure, i.e., 1) adaptive gene selection and 2) similarity enhancement."
- [abstract] Tested on ten real scRNA-seq datasets and five simulated datasets, SSRE achieved the superior performance in most cases compared to several state-of-the-art single-cell clustering methods.: "Tested on ten real scRNA-seq datasets and five simulated datasets, SSRE achieved the superior performance in most cases compared to several state-of-the-art single-cell clustering methods."
- [discussion] One limitation of SSRE is relatively time-consuming in large-scale datasets; therefore, parallel computing is a possible strategy to accelerate the framework [60].: "One limitation of SSRE is relatively time-consuming in large-scale datasets; therefore, parallel computing is a possible strategy to accelerate the framework [60]."
- [readme] [NMI, ~,clus_labs,~]=calc2_nmis(max(true_labs), double(P),true_labs);: [evidence span withheld — anchor status: not_found]
- [readme] [P] = clus_fin_update(rho, lam, lam2, eta, c, in_X, true_labs);: [evidence span withheld — anchor status: not_found]
