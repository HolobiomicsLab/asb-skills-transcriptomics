---
name: bayesian-matrix-factorization-imputation
description: Use when a scRNA-seq count matrix has high dropout (the article notes
  40%-90% missing values) and downstream goals are cell clustering, pseudotime/trajectory
  inference, or visualization that depends on recovering cell-to-cell relationships.
when_to_use_negative:
- The count matrix is already normalized/log-transformed without setting normalized=TRUE
  — Bfimpute would apply CPM and log10(bias 1.01) a second time
- 'The goal is a general-purpose best imputer for any dataset: the article''s discussion
  states ''There is no consensus about which imputation method is the best'' and it
  is ''premature to conclude that Bfimpute will achieve better performance across
  all datasets'''
- False positives in downstream differential expression are unacceptable without tuning
  — the article notes users 'may constraint parameters to maximize true positives
  or minimize false positives'
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: Bfimpute
  role: 'Core imputation tool: Bayesian factorization of the count matrix into latent
    gene (G) and cell (C) matrices via Gibbs-sampling MCMC to recover dropouts'
  repo: github.com/maiziezhoulab/Bfimpute
- name: Spectrum
  role: Spectral clustering of cells to define cell groups within which imputation
    is performed when labels are unavailable (ccluster = c(7,"Spectrum"))
  repo: github.com/maiziezhoulab/Bfimpute
- name: splatter
  role: Simulates scRNA-seq count matrices with known ground truth ('experiment' dropout
    mode, 5-8 cell types) to benchmark imputation against raw data and other methods
- name: scImpute
  role: Comparison imputation method run with default settings on the same matrices
    for clustering and pseudotime benchmarking
- name: DrImpute
  role: Comparison imputation method that averages imputation across multiple clustering
    results
- name: TSCAN
  role: Pseudotime inference tool used to score raw vs. imputed matrices via POS and
    Kendall's rank correlation on Chu, Scialdone, and Petropoulos datasets
provenance:
  source_task_ids:
  - task_001
  - task_004
  - task_008
  - task_007
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100133
    title: A Bayesian factorization method to recover single-cell RNA sequencing data
  source_content_hash: sha256:198e8b5f6a26aeef788aba03e26417cfba4456b993c11ff0e7ce791308fe6f4a
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/maiziezhoulab/Bfimpute
  tools:
  - Bfimpute
  - DrImpute
  - MAGIC
  - Monocle
  - PBLR
  - SAVER
  - SCRABBLE
  - Spectrum
  - TSCAN
  - VIPER
  - kernlab
  - mnormt
  - netNMF-sc
  - parallel
  - rsvd
  - scImpute
  - splatter
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7586206896551724
n_claims: 29
---

# bayesian-matrix-factorization-imputation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Applies Bfimpute, a full Bayesian probabilistic matrix factorization with Gibbs-sampling MCMC, to impute dropout events in scRNA-seq count matrices by factorizing them into latent gene-specific and cell-specific feature matrices, optionally leveraging cell labels, spectral clustering, or gene/cell side information such as bulk RNA-seq.

## When to use

Use when a scRNA-seq count matrix has high dropout (the article notes 40%-90% missing values) and downstream goals are cell clustering, pseudotime/trajectory inference, or visualization that depends on recovering cell-to-cell relationships.

## When NOT to use

- The count matrix is already normalized/log-transformed without setting normalized=TRUE — Bfimpute would apply CPM and log10(bias 1.01) a second time
- The goal is a general-purpose best imputer for any dataset: the article's discussion states 'There is no consensus about which imputation method is the best' and it is 'premature to conclude that Bfimpute will achieve better performance across all datasets'
- False positives in downstream differential expression are unacceptable without tuning — the article notes users 'may constraint parameters to maximize true positives or minimize false positives'

## Inputs

- Raw scRNA-seq count matrix (CSV; rows = genes, columns = cells), e.g., sc_qc.csv with QC
- Optional bulk RNA-seq matrix (bulk_qc.csv) for gene side information S_G
- Optional cell-type label file (cell_type.csv) used when ccluster='labeled'
- Splatter-simulated count matrices (20,000 genes, 500-800 cells, 5-8 cell types, 'experiment' dropout mode) for benchmarking

## Outputs

- Imputed gene expression matrix R_calculate = G^T C (returned directly or via returnGC=TRUE list with G, C, R_calculate)
- Saved result files Bfimpute.csv, Bfimpute.txt, Bfimpute.rds in ./Bfimpute/ (out_type='all')
- Downstream clustering metrics (ARI, Jaccard, NMI, purity) and pseudotime scores (POS, Kendall) versus raw and competing imputations

## How to apply

Load raw counts (genes x rows, cells x columns) and optionally bulk and cell-type files (e.g., sc_qc.csv, bulk_qc.csv, cell_type.csv from Chu et al. 2016). Group cells either with provided labels (ccluster='labeled'), via Spectrum spectral clustering (e.g., c(7,"Spectrum") for the 7-cell-type Chu data), or with a custom function such as kmeans, then normalize with CPM plus log10 transform with bias 1.01 (normalized=FALSE by default). Run the Gibbs sampler over latent matrices G and C with documented defaults D=32, totalepoch=300, burnin=200, sn_init=1, sn_max=10, threshold=0.5, ncores=5, and hyperparameters mu_0=0, beta_0=2, nu_0=D, W_0=I. Optionally supply gene or cell side information (S_G = t(bulk) so bulk columns align with count-matrix gene rows; S_C analogously) to assist imputation. Extract the imputed matrix R_calculate = G^T C by setting returnGC=TRUE, and save with out_type (default 'all' writes Bfimpute.csv/txt/rds to ./Bfimpute/). Judge success by improved same-cell-type Pearson correlations of cells (approximately 0.70 to 0.87 on Chu), better clustering scores (adjusted Rand index, Jaccard index, normalized mutual information, purity) on splatter-simulated data with 5-8 cell types, and higher POS and Kendall's pseudotime scores versus raw data and imputers like scImpute, DrImpute, MAGIC, SAVER, VIPER, and PBLR.

## Related tools

- **Bfimpute** (Core imputation tool: Bayesian factorization of the count matrix into latent gene (G) and cell (C) matrices via Gibbs-sampling MCMC to recover dropouts) — github.com/maiziezhoulab/Bfimpute
- **Spectrum** (Spectral clustering of cells to define cell groups within which imputation is performed when labels are unavailable (ccluster = c(7,"Spectrum"))) — github.com/maiziezhoulab/Bfimpute
- **splatter** (Simulates scRNA-seq count matrices with known ground truth ('experiment' dropout mode, 5-8 cell types) to benchmark imputation against raw data and other methods)
- **scImpute** (Comparison imputation method run with default settings on the same matrices for clustering and pseudotime benchmarking)
- **DrImpute** (Comparison imputation method that averages imputation across multiple clustering results)
- **TSCAN** (Pseudotime inference tool used to score raw vs. imputed matrices via POS and Kendall's rank correlation on Chu, Scialdone, and Petropoulos datasets)

## Examples

```
counts_imputed <- Bfimpute(counts, ccluster = c(7,"Spectrum"), ncores = 5)
```

## Evaluation signals

- On splatter-simulated data (5-8 cell types), Bfimpute should yield the highest ARI, Jaccard index, NMI, and purity versus raw data and scImpute/DrImpute/MAGIC/SAVER/VIPER/PBLR over 10 random seeds, with all metrics in [0,1]
- Pseudotime on Chu with TSCAN should improve from raw POS 0.610 / Kendall 0.317 to approximately 1.000 / 0.902 after imputation
- Same-cell-type Pearson correlations of cells should rise from approximately 0.70 on raw Chu data into the reported 0.70-0.87 improvement range
- Supplied bulk side information (S_G = t(bulk)) should produce no significant performance difference versus no side information, and SCRABBLE/netNMF-sc should not outperform Bfimpute (Figures S1I-S1J)
- Output files Bfimpute.csv/.txt/.rds exist in ./Bfimpute/ and returnGC=TRUE returns a list containing G, C, and R_calculate with dimensions matching the input count matrix

## Limitations

- There is no consensus on the best imputation method; performance varies across criteria, protocols, datasets, and downstream analyses
- It is premature to conclude Bfimpute will outperform across all datasets; generalizability beyond benchmarked data is unconfirmed
- Imputation may increase false positives in downstream analysis; users must tune parameters per application
- Larger D, totalepoch, and burnin increase accuracy but cost more compute time; smaller values trade accuracy for speed

## Evidence

- [intro] Bfimpute uses full Bayesian inference to describe the latent information for genes and cells and carries out a Markov chain Monte Carlo scheme: "Bfimpute uses full Bayesian inference to describe the latent information for genes and cells and carries out a Markov chain Monte Carlo scheme"
- [readme] Bfimpute will perform CPM and log10 transform with bias 1.01: "If `FALSE`, Bfimpute will perform CPM and log10 transform with bias 1.01."
- [readme] counts_imputed <- Bfimpute(counts, ccluster = c(7,"Spectrum"), ncores = 5): "counts_imputed <- Bfimpute(counts, ccluster = c(7,"Spectrum"), ncores = 5)"
- [readme] S_G = t(bulk); counts_imputed <- Bfimpute(counts, ccluster = c(7,"Spectrum"), S_G = S_G, ncores = 5): "S_G = t(bulk)
counts_imputed <- Bfimpute(counts, ccluster = c(7,"Spectrum"), S_G = S_G, ncores = 5)"
- [results] Bfimpute achieved the best clustering scores (adjusted Rand index, Jaccard index, normalized mutual information, purity) versus raw data and other imputation methods: [evidence span withheld — anchor status: not_found]
- [discussion] There is no consensus about which imputation method is the best.: "There is no consensus about which imputation method is the best."
