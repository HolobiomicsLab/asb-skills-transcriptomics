---
name: single-cell-rna-seq-dropout-imputation
description: Use when a scRNA-seq count matrix has extensive dropouts (40%-90% missing
  values) and downstream tasks such as cell clustering, t-SNE visualization, differential
  expression, or pseudotime/trajectory inference are degraded by zero inflation.
when_to_use_negative:
- The input matrix is already normalized and log-transformed without setting normalized
  = TRUE, risking double normalization (Bfimpute will redo CPM + log10 bias 1.01 otherwise)
- No benchmark or labeled/ordered data exists and only point estimates of individual
  gene activity are needed, since the article notes prior imputation methods are less
  effective at recovering cell-to-cell relationships but no single method is best
  for every criterion
- A dataset far outside the benchmarked regimes (simulated 5-8 cell types with 20,000
  genes and 500-800 cells; Chu, Scialdone, Petropoulos) is used without tuning, since
  generalizability beyond these datasets is unconfirmed
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2229
tools:
- name: Bfimpute
  role: 'Core imputation method: Bayesian factorization of the count matrix into gene-
    and cell-specific latent matrices with MCMC, run per cell cluster'
  repo: https://github.com/maiziezhoulab/Bfimpute
- name: Spectrum
  role: Spectral clustering used inside Bfimpute (ccluster = c(7, "Spectrum")) to
    group similar cells before within-group imputation
  repo: https://github.com/maiziezhoulab/Bfimpute
- name: scImpute
  role: Comparison imputer benchmarked on the same matrices
- name: DrImpute
  role: Comparison imputer that averages imputation over multiple clustering results
- name: MAGIC
  role: Comparison imputer based on a Markov affinity-based graph
- name: SAVER
  role: Comparison Bayesian imputer that alters all gene expression values
- name: VIPER
  role: Comparison imputer using non-negative sparse regression on local neighborhoods
- name: PBLR
  role: Comparison low-rank matrix-factorization imputer using ADMM
- name: TSCAN
  role: Pseudotime inference tool used to score imputed matrices via POS and Kendall
    correlation
- name: Monocle
  role: Pseudotime inference tool used to score imputed matrices via POS and Kendall
    correlation
- name: splatter
  role: Simulation of scRNA-seq ground-truth matrices with 'experiment' dropout mode
    for benchmarking (5-8 cell types, 20,000 genes, 500-800 cells)
- name: SCRABBLE
  role: Bulk-constrained comparison imputer on the Chu dataset
- name: netNMF-sc
  role: Gene-gene-interaction-based comparison imputer on the Chu dataset
provenance:
  source_task_ids:
  - task_004
  - task_007
  - task_001
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100133
    title: A Bayesian factorization method to recover single-cell RNA sequencing data
  source_content_hash: sha256:eeb3a481d509c4eb81d5a503b64633b81c286539f1c5595056d81d03b3147c3b
schema_version: 0.2.0
metadata:
  license_tier: noncommercial
  tool_license:
    tier: noncommercial
    requires_ack: true
    ref: CC-BY-NC-ND-4.0
    url: https://github.com/maiziezhoulab/Bfimpute
  repo_url: https://github.com/maiziezhoulab/Bfimpute
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
  - netNMF-sc
  - scImpute
  - splatter
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 22
---

# single-cell-rna-seq-dropout-imputation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Impute dropout events in scRNA-seq count matrices using Bfimpute, a Bayesian factorization method that reconstructs latent gene (G) and cell (C) matrices via MCMC, optionally within spectral clusters and with gene/cell side information (e.g., bulk RNA-seq), to improve clustering and pseudotime analyses.

## When to use

Use when a scRNA-seq count matrix has extensive dropouts (40%-90% missing values) and downstream tasks such as cell clustering, t-SNE visualization, differential expression, or pseudotime/trajectory inference are degraded by zero inflation.

## When NOT to use

- The input matrix is already normalized and log-transformed without setting normalized = TRUE, risking double normalization (Bfimpute will redo CPM + log10 bias 1.01 otherwise)
- No benchmark or labeled/ordered data exists and only point estimates of individual gene activity are needed, since the article notes prior imputation methods are less effective at recovering cell-to-cell relationships but no single method is best for every criterion
- A dataset far outside the benchmarked regimes (simulated 5-8 cell types with 20,000 genes and 500-800 cells; Chu, Scialdone, Petropoulos) is used without tuning, since generalizability beyond these datasets is unconfirmed

## Inputs

- Raw scRNA-seq count matrix (genes x cells, e.g., CSV such as sc_qc.csv)
- Optional cell type labels (e.g., cell_type.csv) for ccluster = 'labeled'
- Optional gene side-information matrix S_G (e.g., t(bulk) from bulk_qc.csv, columns aligned to count-matrix genes)
- Optional cell side-information matrix S_C (columns aligned to count-matrix cells)
- Cluster count / clustering choice, e.g., c(7, "Spectrum") for the 7-cell-type Chu dataset

## Outputs

- Imputed gene expression matrix (R_calculate) saved as Bfimpute.csv, Bfimpute.txt, and Bfimpute.rds in ./Bfimpute/
- Optionally, the latent matrices G and C (returnGC = TRUE)
- Downstream artifacts: cluster metrics (ARI, Jaccard, NMI, purity), t-SNE embeddings, PCA/correlation plots, and pseudotime POS/Kendall scores

## How to apply

Load the gene-by-cell raw count matrix (e.g., sc_qc.csv from the Chu et al. 2016 dataset) and run Bfimpute with ccluster = c(7, "Spectrum") when the number of cell types is known, letting the tool perform CPM normalization and log10 transform with bias 1.01 (normalized = FALSE). Use defaults D = 32 (latent factor dimension), totalepoch = 300, burnin = 200, threshold = 0.5 (dropout probability cutoff), sn_max = 10, and ncores = 5; alternatively pass ccluster = "labeled" with known cell_type labels, a custom clustering function (e.g., kmeans), or a gene side-information matrix S_G = t(bulk) whose columns align with count-matrix rows. The factorization leverages similar cells within clusters so cell-to-cell relationships are recovered better than global imputers (scImpute, DrImpute, MAGIC, SAVER, VIPER, PBLR), and full Bayesian MCMC makes side information easy to incorporate. Collect the returned imputed matrix (default outputs Bfimpute.csv, Bfimpute.txt, Bfimpute.rds in ./Bfimpute/) and feed it to downstream clustering, TSCAN/Monocle pseudotime, or PCA workflows; benchmark against raw data and competing imputers using ARI, Jaccard index, NMI, purity, POS, and Kendall rank correlation against known labels or developmental orderings.

## Related tools

- **Bfimpute** (Core imputation method: Bayesian factorization of the count matrix into gene- and cell-specific latent matrices with MCMC, run per cell cluster) — https://github.com/maiziezhoulab/Bfimpute
- **Spectrum** (Spectral clustering used inside Bfimpute (ccluster = c(7, "Spectrum")) to group similar cells before within-group imputation) — https://github.com/maiziezhoulab/Bfimpute
- **scImpute** (Comparison imputer benchmarked on the same matrices)
- **DrImpute** (Comparison imputer that averages imputation over multiple clustering results)
- **MAGIC** (Comparison imputer based on a Markov affinity-based graph)
- **SAVER** (Comparison Bayesian imputer that alters all gene expression values)
- **VIPER** (Comparison imputer using non-negative sparse regression on local neighborhoods)
- **PBLR** (Comparison low-rank matrix-factorization imputer using ADMM)
- **TSCAN** (Pseudotime inference tool used to score imputed matrices via POS and Kendall correlation)
- **Monocle** (Pseudotime inference tool used to score imputed matrices via POS and Kendall correlation)
- **splatter** (Simulation of scRNA-seq ground-truth matrices with 'experiment' dropout mode for benchmarking (5-8 cell types, 20,000 genes, 500-800 cells))
- **SCRABBLE** (Bulk-constrained comparison imputer on the Chu dataset)
- **netNMF-sc** (Gene-gene-interaction-based comparison imputer on the Chu dataset)

## Examples

```
devtools::install_github("maiziezhoulab/Bfimpute"); library(Bfimpute); counts = read.csv("sc_qc.csv", row.names = 1, header = T); counts_imputed <- Bfimpute(counts, ccluster = c(7,"Spectrum"), ncores = 5)
```

## Evaluation signals

- On simulated splatter data (5-8 cell types), clustering metrics (ARI, Jaccard index, NMI, purity) computed over 10 random seeds should all fall in [0,1] and Bfimpute should exceed raw data and other imputers (Figure 2B pattern)
- Pseudotime benchmarking: Chu TSCAN scores should improve from POS 0.610 / Kendall 0.317 on raw data to POS 1.000 / Kendall 0.902 after Bfimpute imputation, matching Table 1
- Side-information control: adding S_G = t(bulk) to Bfimpute on the Chu dataset should produce no significant performance difference, and SCRABBLE/netNMF-sc should not outperform Bfimpute (Figures S1I-S1J)
- Output files Bfimpute.csv, Bfimpute.txt, and Bfimpute.rds exist in ./Bfimpute/ and the returned matrix has the same gene x cell dimensions as the input counts
- PCA/t-SNE projections of the imputed matrix should show more compact, well-separated clusters aligned with the provided or known cell-type labels

## Limitations

- No consensus exists on which imputation method is best; performance varies across evaluation criteria, experimental protocols, datasets, and downstream analyses
- It is premature to conclude Bfimpute will achieve better performance across all datasets; generalizability beyond benchmarked datasets is unconfirmed
- Imputation can increase false positives in downstream analyses; users may need to constraint parameters (e.g., threshold) to maximize true positives or minimize false positives
- Larger D, totalepoch, and burnin increase accuracy but proportionally increase runtime

## Evidence
