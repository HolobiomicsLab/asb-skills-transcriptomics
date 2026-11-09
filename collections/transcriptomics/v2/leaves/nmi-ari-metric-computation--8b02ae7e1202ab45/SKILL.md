---
name: nmi-ari-metric-computation
description: Use when comparing scRNA-seq clustering results (e.g., SSRE/SSR/spectral-clustering
  assignments) against ground-truth cell-type labels and a reproducible NMI/ARI benchmark
  table is needed.
when_to_use_negative:
- The dataset lacks pre-annotated ground-truth cell-type labels — NMI/ARI agreement
  with true labels cannot be computed
- You need runtime benchmarking or other metrics (e.g., Purity) as the primary target
  — NMI/ARI alone measures label agreement, and SSRE is already noted as time-consuming
  on large-scale datasets
- Evaluating simulated-data scenarios for which the published ten real-dataset benchmark
  tables (Tables 2 and 3) provide no reference values
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_3474
tools:
- name: MATLAB
  role: Environment in which SSRE and MPSSC clustering runs and NMI/ARI computation
    functions (calc2_nmis, RandIndex) are executed
  repo: https://github.com/CSUBioGroup/SSRE
- name: Python
  role: Alternative implementation language of SSRE used to run the pipeline and compute
    NMI/ARI against provided cell type labels
  repo: https://github.com/CSUBioGroup/SSRE
- name: MPSSC
  role: Comparison method whose clus_labs output is scored with calc2_nmis and RandIndex
    in the MATLAB example
  repo: https://github.com/ishspsy/project
- name: SC3
  role: Comparison clustering method evaluated by the same NMI/ARI protocol on the
    ten datasets
  repo: https://github.com/hemberg-lab/SC3
provenance:
  source_task_ids:
  - task_003
  - task_001
  source_papers:
  - doi: 10.1016/j.gpb.2020.09.004
    title: 'SSRE: Cell Type Detection Based on Sparse Subspace Representation and
      Similarity Enhancement'
  source_content_hash: sha256:d2280ecf76e8df3a983456a502922808cd9c38147e57d4cf3f1f0ae1a5d574bf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSUBioGroup/SSRE
  tools:
  - MATLAB
  - MPSSC
  - Python
  - SC3
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_3474
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# nmi-ari-metric-computation

## Summary

Compute Normalized Mutual Information (NMI) and Adjusted Rand Index (ARI) between cell-cluster assignments and pre-annotated cell-type labels to quantify clustering agreement, as used to benchmark SSRE against SSR and state-of-the-art methods on ten real scRNA-seq datasets.

## When to use

Use when comparing scRNA-seq clustering results (e.g., SSRE/SSR/spectral-clustering assignments) against ground-truth cell-type labels and a reproducible NMI/ARI benchmark table is needed.

## When NOT to use

- The dataset lacks pre-annotated ground-truth cell-type labels — NMI/ARI agreement with true labels cannot be computed
- You need runtime benchmarking or other metrics (e.g., Purity) as the primary target — NMI/ARI alone measures label agreement, and SSRE is already noted as time-consuming on large-scale datasets
- Evaluating simulated-data scenarios for which the published ten real-dataset benchmark tables (Tables 2 and 3) provide no reference values

## Inputs

- Cluster label assignments produced by a scRNA-seq clustering method (e.g., clus_labs or target matrix P from MPSSC/SSRE spectral clustering)
- Ground-truth cell-type label vector (true_labs) from a pre-annotated scRNA-seq dataset
- MATLAB/Python implementations of NMI (calc2_nmis) and ARI (RandIndex) measures

## Outputs

- NMI value per dataset-method pair
- ARI value per dataset-method pair
- Aggregated benchmark table (10 datasets x 11 methods) mirroring Tables 2 and 3, and an SSR-vs-SSRE ablation win count

## How to apply

Load each preprocessed gene-by-cell scRNA-seq matrix together with its pre-annotated cell-type labels (e.g., Treutlein, Deng, Ting, Macosko, Yan, Goolam, Song/GSE85908, Engel/GSE74597, Haber/GSE92332, Vento/E-MTAB-6678). Run each clustering method (SSRE, SSR, SC, SIMLR, MPSSC, Corr, SNN-Cliq, NMF, SC3, dropClust, Seurat) with the known number of cell types, then compare predicted cluster labels to true labels using NMI and ARI. In the MATLAB workflows referenced by the SSRE/MPSSC repositories, NMI is computed with calc2_nmis(max(true_labs), double(P), true_labs) and ARI with RandIndex(clus_labs, true_labs). Tabulate NMI and ARI per dataset and method (mirroring the 10-dataset x 11-method layout of Tables 2 and 3) and, for the ablation, count datasets where enhanced SSRE beats the original SSR similarity. Validate by checking reproduced values match the published tables within a small numerical tolerance; for the ablation, expect SSRE to outperform SSR on nine of ten datasets on both metrics.

## Related tools

- **MATLAB** (Environment in which SSRE and MPSSC clustering runs and NMI/ARI computation functions (calc2_nmis, RandIndex) are executed) — https://github.com/CSUBioGroup/SSRE
- **Python** (Alternative implementation language of SSRE used to run the pipeline and compute NMI/ARI against provided cell type labels) — https://github.com/CSUBioGroup/SSRE
- **MPSSC** (Comparison method whose clus_labs output is scored with calc2_nmis and RandIndex in the MATLAB example) — https://github.com/ishspsy/project
- **SC3** (Comparison clustering method evaluated by the same NMI/ARI protocol on the ten datasets) — https://github.com/hemberg-lab/SC3

## Examples

```
[NMI, ~,clus_labs,~]=calc2_nmis(max(true_labs), double(P),true_labs); ARI=RandIndex(clus_labs,true_labs)
```

## Evaluation signals

- Reproduced NMI/ARI values for SSRE and each comparison method match the corresponding entries of Tables 2 and 3 within a small numerical tolerance
- In the SSR-vs-SSRE ablation, SSRE outperforms SSR on nine of the ten real datasets on both NMI and ARI; flag any divergent dataset
- Both NMI and ARI fall within [0, 1] and ARI of a correct clustering against its own labels equals 1
- Metric functions are called with the known number of cell types and the ground-truth label vector, not the cluster-count estimate
- The aggregated table reproduces the paper's qualitative result: SSRE achieves best or tied-first NMI/ARI on seven datasets and ranks second on three (NMI) and two (ARI) datasets

## Limitations

- SSRE is relatively time-consuming on large-scale datasets, limiting throughput of the benchmark loop
- NMI and ARI do not cover other quality measures used in this literature (e.g., Purity, computed alongside NMI and ARI in the MPSSC example)
- Metric agreement depends on preprocessing (adaptive gene selection plus similarity enhancement); comparing scores across differently preprocessed matrices is not meaningful

## Evidence

- [abstract] abstract: "Tested on ten real scRNA-seq datasets and five simulated datasets, SSRE achieved the superior performance in most cases compared to several state-of-the-art single-cell clustering methods."
- [discussion] discussion: "The systematic performance evaluations on multiple scRNA-seq datasets showed that SSRE achieves superior performance among all competing methods."
- [intro] intro: "The experimental results show that when combined with spectral clustering, the learned similarities by SSRE can reveal the block structure of scRNA-seq data reliably."
- [readme] readme: [evidence span withheld — anchor status: not_found]
- [readme] readme (ARI): [evidence span withheld — anchor status: not_found]
