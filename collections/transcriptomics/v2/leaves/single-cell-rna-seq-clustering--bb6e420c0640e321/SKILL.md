---
name: single-cell-rna-seq-clustering
description: Use when raw UMI count matrices from multiple individuals (e.g., 10x
  Genomics Chromium droplet data) must be clustered into K cell types while correcting
  batch effects and unbalanced sequencing depth across donors.
when_to_use_negative:
- The goal is rare cell type identification after filtering genes with excessive zeros
  — the paper notes this filtering 'may be problematic for rare cell type identification'
- Input is not raw UMI counts (e.g., already normalized/log-transformed data) — BAMM-SC
  is designed to take raw count data as input
- The analysis goal is trajectory/developmental-stage inference — the paper defers
  trajectory analysis to future work and assumes each cell belongs to one distinct
  cluster
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: BAMM-SC
  role: Primary Bayesian mixture model for multi-individual clustering; also provides
    BAMMSC.predict for supervised label prediction
  repo: https://github.com/lichen-lab/BAMMSC
- name: DIMM-SC
  role: Dirichlet multinomial mixture model invoked for single-individual or pooled
    (option="DIMMSC") analysis within the BAMMSC package
  repo: https://github.com/lichen-lab/BAMMSC
- name: SIMLR
  role: Competing unsupervised scRNA-seq clustering method used as benchmark
- name: CellTree
  role: Competing unsupervised scRNA-seq clustering method used as benchmark
- name: SC3
  role: Competing unsupervised scRNA-seq clustering method used as benchmark
- name: TSCAN
  role: Competing unsupervised scRNA-seq clustering method used as benchmark
- name: scran
  role: Implementation of mutual nearest neighbors (MNN) batch correction used as
    competing benchmark
- name: mclust
  role: Provides adjustedRandIndex to score clustering/prediction accuracy against
    true labels
provenance:
  source_task_ids:
  - task_005
  - task_004
  source_papers:
  - doi: 10.1038/s41467-019-09639-3
    title: A Bayesian mixture model for clustering droplet-based single-cell transcriptomic
      data from population studies
  source_content_hash: sha256:ceec95cb434a48065498a658a8278a993cbc729a5029660150990dcbebe033c8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lichen-lab/BAMMSC
  tools:
  - BAMM-SC
  - CIDR
  - CellTree
  - DCA
  - DEC
  - DIMM-SC
  - Leiden
  - Nvidia Tesla P100
  - Python
  - Python 3.8
  - R
  - SC3
  - SIMLR
  - Scanpy
  - TSCAN
  - mclust
  - scAce
  - scDCC
  - scVI
  - scran
  - scvis
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4666666666666667
n_claims: 15
---

# single-cell-rna-seq-clustering

## Summary

Cluster droplet-based scRNA-seq UMI count data from multiple individuals using BAMM-SC, a Bayesian hierarchical Dirichlet multinomial mixture model that accounts for individual heterogeneity and batch effect, and benchmark it against scRNA-seq-tailored methods (SIMLR, CellTree, SC3, TSCAN, DIMM-SC, scran/MNN).

## When to use

Use when raw UMI count matrices from multiple individuals (e.g., 10x Genomics Chromium droplet data) must be clustered into K cell types while correcting batch effects and unbalanced sequencing depth across donors.

## When NOT to use

- The goal is rare cell type identification after filtering genes with excessive zeros — the paper notes this filtering 'may be problematic for rare cell type identification'
- Input is not raw UMI counts (e.g., already normalized/log-transformed data) — BAMM-SC is designed to take raw count data as input
- The analysis goal is trajectory/developmental-stage inference — the paper defers trajectory analysis to future work and assumes each cell belongs to one distinct cluster

## Inputs

- List of per-individual raw UMI count matrices (genes x cells), same gene set across individuals (a single matrix triggers DIMMSC)
- Predefined number of clusters K (or K selected from AIC/BIC)
- Optionally, true cell labels (unlisted membership vectors) for adjustedRandIndex evaluation
- Optionally, a training proportion p for BAMMSC.predict supervised classification

## Outputs

- mem: list of cluster membership vectors, one per individual
- loglik: vector of log-likelihood over MCMC iterations
- AIC and BIC values for model checking / K selection
- alpha estimates (3-D matrix [#gene, #cluster, #individual] per iteration)
- pi and sigma2 estimates
- delta: posterior probability estimates per cell across iterations

## How to apply

Assemble a list in which each element is a UMI count matrix (rows = genes, columns = cells) for one individual, with the same genes across individuals; BAMM-SC takes raw counts directly and models gene, cell-type and individual variability in a unified Bayesian hierarchy. Filter out genes with excessive zeros (assuming lowly-expressed genes contribute little to clustering) and select highly variable genes (e.g., the top 1000 per dataset). Choose K from prior knowledge of the tissue or via the returned AIC/BIC, then run BAMMSC(X, K) with default MCMC settings nBurn=400, maxIter.BAMMSC=500, likTol.BAMMSC=1e-10 and collect the mem membership list; for single-indicator or pooled data the package falls back to DIMMSC (maxIter.DIMMSC=400, likTol.DIMMSC=1e-4). For large datasets, use BAMMSC.predict(X, K, p, seed=1) to train on a p-fraction subset per individual and predict held-out labels, then score accuracy with mclust's adjustedRandIndex against true labels. Benchmark against SIMLR, CellTree, SC3, TSCAN, DIMM-SC and scran/MNN on identical inputs, comparing adjusted Rand index plus stability (APN) and tightness (silhouette width) metrics.

## Related tools

- **BAMM-SC** (Primary Bayesian mixture model for multi-individual clustering; also provides BAMMSC.predict for supervised label prediction) — https://github.com/lichen-lab/BAMMSC
- **DIMM-SC** (Dirichlet multinomial mixture model invoked for single-individual or pooled (option="DIMMSC") analysis within the BAMMSC package) — https://github.com/lichen-lab/BAMMSC
- **SIMLR** (Competing unsupervised scRNA-seq clustering method used as benchmark)
- **CellTree** (Competing unsupervised scRNA-seq clustering method used as benchmark)
- **SC3** (Competing unsupervised scRNA-seq clustering method used as benchmark)
- **TSCAN** (Competing unsupervised scRNA-seq clustering method used as benchmark)
- **scran** (Implementation of mutual nearest neighbors (MNN) batch correction used as competing benchmark)
- **mclust** (Provides adjustedRandIndex to score clustering/prediction accuracy against true labels)

## Examples

```
library(BAMMSC); result=BAMMSC(data_BAMMSC,K=4)
```

## Evaluation signals

- adjustedRandIndex(unlist(result$mem), unlist(Z)) is high (BAMM-SC achieved the highest ARI in most simulated scenarios, including the 10% cluster-label disagreement case)
- MCMC log-likelihood trace converges within maxIter.BAMMSC=500 under likTol.BAMMSC=1e-10
- t-SNE of clustered cells shows cell-type grouping rather than batch/donor grouping (the uncorrected data showed samples from the same batch clustering together)
- Stability check: low average proportion of nonoverlap (APN) over repeated runs with 100 randomly removed top-1000 highly variable genes, consistent with Supplementary Tables 7-8
- Tightness check: high average silhouette width of cluster memberships across the human PBMC, mouse lung and human skin datasets

## Limitations

- Filtering genes with excessive zeros may be problematic for rare cell type identification
- Zero-inflation is not explicitly modeled, which may or may not affect clustering accuracy
- The supervised strategy for large datasets is efficient but may potentially miss some rare clusters
- The number of clusters is treated as a continuum, so a range of cluster numbers should be tried rather than a single value; posterior probabilities measure clustering uncertainty, not cell cycle or developmental stage
- Runtimes are substantial (~1.5, 2.5 and 4.5 h for human PBMC, mouse lung and human skin; ~30 min for 10 individuals x 4000 cells); parallel/GPU speed-up is not implemented

## Evidence

- [abstract] BAMM-SC takes raw count data as input and accounts for data heterogeneity and batch effect among multiple individuals in a unified Bayesian hierarchical model: "BAMM-SC takes raw count data as input and accounts for data heterogeneity and batch effect among multiple individuals in a unified Bayesian hierarchical model framework"
- [intro] Competing methods named in the article: "scRNA-seq tailored unsupervised methods, such as SIMLR9, CellTree10, SC311, TSCAN12, and DIMM-SC13"
- [discussion] Gene filtering rationale: "we filtered out genes with excessive zeros from the analysis under the assumption that lowly-expressed genes do not contribute much to clustering"
- [readme] BAMMSC function usage and defaults: "BAMMSC(X, K=4, option="BAMMSC",method_cluster_initial="kmeans", method_alpha_initial="Ronning", maxIter.DIMMSC=400, tol.DIMMSC=1e-4, likTol.DIMMSC=1e-2, nBurn=400, maxIter.BAMMSC=500,"
- [readme] Mem output structure: "mem: a list of vectors. Each vector [#cell] is the clustering membership for all cells in each individual. Length of list is #individuals"
