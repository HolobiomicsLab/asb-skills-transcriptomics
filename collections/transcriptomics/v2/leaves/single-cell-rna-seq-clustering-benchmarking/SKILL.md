---
name: single-cell-rna-seq-clustering-benchmarking
description: Use when you must quantitatively compare scRNA-seq clustering methods
  against ground-truth cell-type annotations on datasets with 15-30k cells, using
  ARI/AMI/NMI and number-of-cluster accuracy metrics.
when_to_use_negative:
- Input is a dataset without ground-truth cell-type annotations - ARI/AMI/NMI comparison
  and log-modulus scoring cannot be computed
- Datasets are so large that SIMLR cannot run (documented out-of-memory errors on
  Tabula Sapiens datasets); the large-scale comparison only covers the remaining five
  methods
- The goal is de novo clustering of a single dataset rather than method-vs-method
  benchmarking against reference labels
edam_operation: http://edamontology.org/operation_3644
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0091
tools:
- name: CASCC
  role: Co-expression-assisted clustering method under evaluation; run via CASCC::run.CASCC
    with featureSelection, predictClusterK (Ward.D2/silhouette), and kmeansCenterModify
  repo: https://github.com/LingyiC/CASCC
- name: Seurat 4.3.0
  role: Pinned dependency version; baseline clustering method (Seurat V4) and used
    for DEG discovery on CASCC results
- name: cafr
  role: Dependency providing the adaptive attractor algorithm underlying CASCC feature
    selection
  repo: https://github.com/weiyi-bitw/cafr
- name: aricode
  role: Computes ARI, AMI, and NMI between predicted cluster labels and ground-truth
    cell-type annotations for every dataset-method pair
- name: NbClust
  role: Listed among benchmark environment tools for the Tabula Sapiens comparison
- name: R
  role: Benchmark runtime environment, pinned to R 4.0.5 with documented sessionInfo
    for reproducibility
provenance:
  source_task_ids:
  - task_001
  - task_004
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btae283
    title: 'CASCC: a co-expression-assisted single-cell RNA-seq data clustering method'
  source_content_hash: sha256:41a904942a689076bc1f5621530289e5a0ea4fe93e1e16ebe80a7954a14bd076
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/LingyiC/CASCC
  tools:
  - CASCC
  - CIDR
  - Monocle
  - NbClust
  - PhenoGraph
  - PyTorch
  - Python
  - R
  - RaceID
  - Rphenograph
  - SC3
  - SHARP
  - SINCERA
  - Seurat
  - Seurat 4.3.0
  - aricode
  - cafr
  - dplyr
  - faiss-cpu
  - pcaReduce
  - scDHA
  - scEFSC
  - scGCL
  - wMetaC
  edam_operation: http://edamontology.org/operation_3644
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6875
n_claims: 16
---

# single-cell-rna-seq-clustering-benchmarking

## Summary

A benchmarking skill for evaluating scRNA-seq clustering methods (CASCC vs. CIDR, RaceID3, SC3, Seurat V4, SIMLR, TSCAN) on datasets with ground-truth cell-type labels, computing ARI/AMI/NMI with aricode and comparing estimated vs. true cluster numbers via absolute log-modulus and deviation scores.

## When to use

Use when you must quantitatively compare scRNA-seq clustering methods against ground-truth cell-type annotations on datasets with 15-30k cells, using ARI/AMI/NMI and number-of-cluster accuracy metrics.

## When NOT to use

- Input is a dataset without ground-truth cell-type annotations - ARI/AMI/NMI comparison and log-modulus scoring cannot be computed
- Datasets are so large that SIMLR cannot run (documented out-of-memory errors on Tabula Sapiens datasets); the large-scale comparison only covers the remaining five methods
- The goal is de novo clustering of a single dataset rather than method-vs-method benchmarking against reference labels

## Inputs

- Normalized, log-transformed scRNA-seq count matrices from the 15 Krzak et al. 2019 evaluation datasets with ground-truth cell-type labels (LingyiC/CASCC_benchmark repo)
- 19 Tabula Sapiens organ scRNA-seq datasets (5,000-30,000 cells) with ground-truth cell-type annotations
- Cluster label assignments from each method (CASCC plus CIDR, RaceID3, SC3, Seurat V4, SIMLR, TSCAN)
- True cell-type annotation tables per dataset (K_true)

## Outputs

- Per-dataset, per-method ARI/AMI/NMI score table (benchmark_ari_ami_nmi_scores.csv)
- Per-dataset ARI table for the Tabula Sapiens large-scale comparison (tabula_sapiens_ari_table.csv)
- 7 x 15 table of generated-vs-true cluster numbers with absolute log-modulus and deviation scores (supplementary_table_s5.csv)

## How to apply

Load each evaluation dataset with ground-truth cell-type labels from the LingyiC/CASCC_benchmark repository and preprocess as normalized, log-transformed expression data (inputDataType 'well-normalized'). Run CASCC via CASCC::run.CASCC, which performs co-expression-based feature selection (adaptive attractor scanning), cluster-number prediction (Ward.D2 hierarchical clustering with silhouette index), and k-means clustering with attractor-modified centers, extracting res$mainType.output$clusteringResults. Run the baselines (CIDR, RaceID3, SC3, Seurat V4, SIMLR, TSCAN) on the same feature inputs, excluding SIMLR from large-scale Tabula Sapiens runs (5,000-30,000 cells) where it fails with out-of-memory errors. Score every method-dataset pair with the aricode package to get ARI, AMI, and NMI against the reference labels. Separately, compute number-of-cell-types accuracy as |log2(K_gen/K_true)| and deviation (K_gen - K_true) for each method-dataset pair, then aggregate into per-dataset, per-method tables (e.g. benchmark_ari_ami_nmi_scores.csv, tabula_sapiens_ari_table.csv, supplementary_table_s5.csv) under the documented session info (R 4.0.5, Seurat 4.3.0, cafr).

## Related tools

- **CASCC** (Co-expression-assisted clustering method under evaluation; run via CASCC::run.CASCC with featureSelection, predictClusterK (Ward.D2/silhouette), and kmeansCenterModify) — https://github.com/LingyiC/CASCC
- **Seurat 4.3.0** (Pinned dependency version; baseline clustering method (Seurat V4) and used for DEG discovery on CASCC results)
- **cafr** (Dependency providing the adaptive attractor algorithm underlying CASCC feature selection) — https://github.com/weiyi-bitw/cafr
- **aricode** (Computes ARI, AMI, and NMI between predicted cluster labels and ground-truth cell-type annotations for every dataset-method pair)
- **NbClust** (Listed among benchmark environment tools for the Tabula Sapiens comparison)
- **R** (Benchmark runtime environment, pinned to R 4.0.5 with documented sessionInfo for reproducibility)

## Examples

```
res <- CASCC::run.CASCC(Data_PDAC_peng_2k); aricode::ARI(res$mainType.output$clusteringResults, ground_truth_labels)
```

## Evaluation signals

- Computed ARI/AMI/NMI values for CASCC and each baseline reproduce the per-dataset clustering scores in Fig. 1b and Supplementary Table S3
- CASCC's mean ARI across the 15 datasets exceeds SC3 (second-best) by ~34%
- Tabula Sapiens ARI table reproduces the relative per-dataset method comparison of the paper's Table S1/Fig. 1c, with SIMLR absent due to documented OOM failure
- Number-of-clusters table shows CASCC with the smallest absolute log-modulus and deviation scores (Supplementary Table S5), and SC3 overestimating K on large datasets
- Session info matches the paper's documented environment: R 4.0.5, Seurat 4.3.0, cafr installed from weiyi-bitw/cafr

## Limitations

- SIMLR fails with out-of-memory errors on large-scale (5,000-30,000 cell) Tabula Sapiens datasets and is excluded from the large-scale comparison
- SC3 overestimates the number of clusters on large benchmark datasets
- Results depend on pinned versions (R 4.0.5, Seurat 4.3.0) recorded in the paper's sessionInfo; other versions may not reproduce scores

## Evidence

- [supplementary] On the 15 real scRNA-seq evaluation datasets, CASCC improved mean ARI scores by 34% over the second-best performer (SC3): [evidence span withheld — anchor status: not_found]
- [supplementary] For each dataset, run the six baseline methods CIDR, RaceID3, SC3, Seurat V4, SIMLR, and TSCAN to obtain cluster label assignments.: [evidence span withheld — anchor status: not_found]
- [supplementary] SIMLR was excluded from large-scale comparison due to out-of-memory errors: [evidence span withheld — anchor status: not_found]
- [supplementary] Compute the absolute log-modulus score |log2(K_gen/K_true)| and the deviation score: [evidence span withheld — anchor status: not_found]
- [readme] CASCC, a clustering method designed to improve biological accuracy using gene co-expression features identified using an unsupervised adaptive attractor algorithm.: "CASCC, a clustering method designed to improve biological accuracy using gene co-expression features identified using an unsupervised adaptive attractor algorithm."
- [supplementary] verify that the computed ARI/AMI/NMI values for CASCC and each baseline reproduce the per-dataset clustering scores reported in Fig. 1b: [evidence span withheld — anchor status: not_found]
