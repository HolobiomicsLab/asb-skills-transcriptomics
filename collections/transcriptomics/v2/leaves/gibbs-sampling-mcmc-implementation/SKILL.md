---
name: gibbs-sampling-mcmc-implementation
description: Use when scRNA-seq count matrices contain pervasive dropouts (40%-90%
  missing values) and cell-to-cell relationships must be recovered via Bayesian matrix
  factorization with MCMC, optionally leveraging cell labels or bulk side information.
when_to_use_negative:
- 'The counts matrix is already normalized: set normalized=TRUE or Bfimpute will re-run
  CPM and log10 transform with bias 1.01 (log-transform after normalization is still
  recommended)'
- 'A single best imputation is needed universally: the paper notes there is no consensus
  about which imputation method is the best and performance varies by dataset and
  downstream analysis'
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_2229
tools:
- name: Bfimpute
  role: Implements the Bayesian Gibbs-sampling MCMC factorization that trains latent
    G and C matrices and imputes dropout events
  repo: github.com/maiziezhoulab/Bfimpute
- name: Spectrum
  role: Spectral clustering to group cells into 7 clusters before within-group imputation
  repo: github.com/maiziezhoulab/Bfimpute
- name: kernlab
  role: Provides the specc spectral clustering alternative for cell grouping
  repo: github.com/maiziezhoulab/Bfimpute
- name: parallel
  role: Parallel computation across ncores=5 during the MCMC epochs
- name: mnormt
  role: Multivariate normal sampling used in the Bayesian inference scheme
- name: rsvd
  role: Randomized SVD dependency for the Bfimpute R package
provenance:
  source_task_ids:
  - task_008
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100133
    title: A Bayesian factorization method to recover single-cell RNA sequencing data
  source_content_hash: sha256:4d1c53d491c75d362ddccb7929bea030c0cc999dcd7a0d3d3de5c9d17c042382
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
  - Spectrum
  - kernlab
  - mnormt
  - parallel
  - rsvd
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_2229
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6153846153846154
n_claims: 13
---

# gibbs-sampling-mcmc-implementation

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Bfimpute's computational core: full Bayesian inference over latent gene (G) and cell (C) matrices via a Gibbs-sampling Markov chain Monte Carlo scheme, which factorizes the scRNA-seq count matrix into gene-specific and cell-specific feature matrices to impute dropout events.

## When to use

Use when scRNA-seq count matrices contain pervasive dropouts (40%-90% missing values) and cell-to-cell relationships must be recovered via Bayesian matrix factorization with MCMC, optionally leveraging cell labels or bulk side information.

## When NOT to use

- The counts matrix is already normalized: set normalized=TRUE or Bfimpute will re-run CPM and log10 transform with bias 1.01 (log-transform after normalization is still recommended)
- A single best imputation is needed universally: the paper notes there is no consensus about which imputation method is the best and performance varies by dataset and downstream analysis

## Inputs

- QC'd scRNA-seq raw count matrix (genes as rows, cells as columns; sc_qc.csv)
- Optional bulk RNA-seq matrix (bulk_qc.csv) used as gene side information S_G
- Optional cell type labels (cell_type.csv) when ccluster="labeled"
- Optional cell side information matrix S_C with D rows matching cell order

## Outputs

- Imputed gene expression matrix R_calculate = G^T C
- Latent gene matrix G and cell matrix C from the final epoch (when returnGC=TRUE)
- Saved result files as csv, txt, and rds (out_type="all") in ./Bfimpute/

## How to apply

Load the QC'd Chu et al. (2016) counts (sc_qc.csv), optional bulk matrix (bulk_qc.csv), and 7 cell-type labels (cell_type.csv) via read.csv with row.names=1. Normalize raw counts with CPM followed by log10 transform with bias 1.01 (normalized=FALSE), grouping cells either with provided labels (ccluster="labeled") or Spectrum/specc spectral clustering with k=7. Run the Gibbs sampling cycle over latent matrices G and C with hyperpriors mu_0=0, beta_0=2, nu_0=D, W_0=I (identity), using D=32, totalepoch=300, burnin=200, sn_init=1, sn_max=10, dropout threshold=0.5, and ncores=5. Set returnGC=TRUE to extract the final-epoch G and C matrices and the imputed matrix R_calculate = G^T C, and save results via out_type ("all" writes csv, txt, and rds) to the default out_dir "./Bfimpute/". Rationale: larger D, totalepoch, and burnin increase imputation accuracy at higher runtime cost, and cell labels or gene/cell side information (S_G from transposed bulk, S_C) can be incorporated to improve cell-to-cell relationship recovery.

## Related tools

- **Bfimpute** (Implements the Bayesian Gibbs-sampling MCMC factorization that trains latent G and C matrices and imputes dropout events) — github.com/maiziezhoulab/Bfimpute
- **Spectrum** (Spectral clustering to group cells into 7 clusters before within-group imputation) — github.com/maiziezhoulab/Bfimpute
- **kernlab** (Provides the specc spectral clustering alternative for cell grouping) — github.com/maiziezhoulab/Bfimpute
- **parallel** (Parallel computation across ncores=5 during the MCMC epochs)
- **mnormt** (Multivariate normal sampling used in the Bayesian inference scheme)
- **rsvd** (Randomized SVD dependency for the Bfimpute R package)

## Examples

```
counts_imputed <- Bfimpute(counts, ccluster = "labeled", label = cell_type, ncores = 5, returnGC = TRUE)
```

## Evaluation signals

- Same-cell-type Pearson correlations of cells should rise from roughly 0.70 on the raw Chu et al. matrix into the reported improvement range up to approximately 0.87 on the imputed matrix
- With returnGC=TRUE, the returned list must contain R_calculate, G, and C, and R_calculate must equal G^T C
- Convergence check: burnin=200 of totalepoch=300 epochs are used only as running Markov chain, so post-burn-in samples should be used for the final estimate
- Output files written to ./Bfimpute/ with default name 'Bfimpute' in csv, txt, and rds formats when out_type="all"
- The imputed matrix preserves the dimensions and gene/cell ordering of the input count matrix (factorization into gene-specific and cell-specific feature matrices)

## Limitations

- Generalizability beyond benchmarked datasets is unconfirmed: it is premature to conclude that Bfimpute will achieve better performance across all datasets and improve downstream analysis
- Depending on the application, users may need to constrain parameters to maximize true positives or minimize false positives, as imputation may increase false positives in downstream analysis
- Larger D, totalepoch, and burnin increase imputation accuracy but take more time to run; smaller parameters save time at the cost of accuracy
- Dropout probabilities above the threshold=0.5 determine which entries are imputed, so threshold tuning affects which values are altered

## Evidence
