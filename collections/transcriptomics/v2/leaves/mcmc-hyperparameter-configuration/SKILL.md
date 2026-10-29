---
name: mcmc-hyperparameter-configuration
description: Use when running Bfimpute on scRNA-seq count matrices and you must set
  or tune the MCMC factorization settings (D, totalepoch, burnin, threshold, sn_init/sn_max,
  priors) to reproduce published benchmarks or trade accuracy vs. compute time.
when_to_use_negative:
- If you need a universally best imputation setting — the article's discussion states
  no consensus exists about the best imputation method and it is premature to conclude
  Bfimpute wins across all datasets
- If the count matrix is already CPM-normalized and log-transformed — set normalized=TRUE
  instead of relying on the default transform path, otherwise the chain receives double-transformed
  input
- When the goal is a fast exploratory imputation where accuracy can be traded for
  time — do not keep accuracy-oriented defaults (D=32, totalepoch=300, burnin=200)
  uncritically, per the README's accuracy/runtime trade-off note
edam_operation: http://edamontology.org/operation_3947
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: Bfimpute
  role: Bayesian factorization imputation tool whose MCMC scheme consumes the hyperparameters
    (D, totalepoch, burnin, threshold, priors) configured by this skill
  repo: github.com/maiziezhoulab/Bfimpute
- name: devtools
  role: Used to install Bfimpute from GitHub before configuring its hyperparameters
  repo: github.com/maiziezhoulab/Bfimpute
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1016/j.crmeth.2021.100133
    title: A Bayesian factorization method to recover single-cell RNA sequencing data
  source_content_hash: sha256:29b63c8737d58c452357684d95188e533f4ce7519df1c50aeb94d0ff264c2da2
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
  - PBLR
  - devtools
  - scImpute
  edam_operation: http://edamontology.org/operation_3947
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
evidence_trimmed: true
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# mcmc-hyperparameter-configuration

> **License: noncommercial** — confirm your use is a permitted (noncommercial) purpose before applying; commercial use requires a separate license (see `metadata.tool_license`). <!-- asb-license-banner -->
## Summary

Configure the Markov chain Monte Carlo (MCMC) hyperparameters of Bfimpute's full Bayesian factorization imputation for scRNA-seq dropouts — latent dimension D, totalepoch, burnin, dropout-probability threshold, and Normal-Inverse-Wishart priors — balancing imputation accuracy against runtime.

## When to use

Use when running Bfimpute on scRNA-seq count matrices and you must set or tune the MCMC factorization settings (D, totalepoch, burnin, threshold, sn_init/sn_max, priors) to reproduce published benchmarks or trade accuracy vs. compute time.

## When NOT to use

- If you need a universally best imputation setting — the article's discussion states no consensus exists about the best imputation method and it is premature to conclude Bfimpute wins across all datasets
- If the count matrix is already CPM-normalized and log-transformed — set normalized=TRUE instead of relying on the default transform path, otherwise the chain receives double-transformed input
- When the goal is a fast exploratory imputation where accuracy can be traded for time — do not keep accuracy-oriented defaults (D=32, totalepoch=300, burnin=200) uncritically, per the README's accuracy/runtime trade-off note

## Inputs

- QC'd scRNA-seq expression count matrix (sc_qc.csv; rows = genes, columns = cells)
- Optional bulk matrix for gene side information (bulk_qc.csv) passed as S_G
- Cell type labels (cell_type.csv) when running in labeled mode

## Outputs

- Imputed gene expression matrix (R_calculate; optionally with G and C latent matrices when returnGC=TRUE)
- Saved imputed results as csv, txt, and/or rds files in out_dir (default './Bfimpute/')

## How to apply

Run Bfimpute with its default hyperparameters — totalepoch=300, burnin=200, latent dimension D=32, dropout threshold=0.5, ncores=5 — which is the configuration used in the labeled benchmark (Bfimpute with ccluster="labeled", label=cell_type). Keep counts unnormalized (normalized=FALSE) so Bfimpute itself performs CPM and log10 transform with bias 1.01 before the MCMC chain. The chain's Normal-Inverse-Wishart priors are fixed at mu_0=0, beta_0=2, nu_0=D, and W_0=I; adaptive precision runs from sn_init=1 up to sn_max=10. Decision point: larger D, totalepoch, and burnin increase imputation accuracy but proportionally increase runtime, so shrink them for exploratory runs and grow them for final benchmark outputs (e.g., Figure 6 silhouette/Pearson tables). Treat the defaults as the baseline; only deviate after confirming downstream silhouette and same/different-cell-type Pearson correlations on a validation set.

## Related tools

- **Bfimpute** (Bayesian factorization imputation tool whose MCMC scheme consumes the hyperparameters (D, totalepoch, burnin, threshold, priors) configured by this skill) — github.com/maiziezhoulab/Bfimpute
- **devtools** (Used to install Bfimpute from GitHub before configuring its hyperparameters) — github.com/maiziezhoulab/Bfimpute

## Examples

```
counts_imputed <- Bfimpute(counts, ccluster = "labeled", label = cell_type, D = 32, totalepoch = 300, burnin = 200, threshold = 0.5, ncores = 5)
```

## Evaluation signals

- Confirm the call used the benchmark defaults: totalepoch=300, burnin=200, D=32, threshold=0.5, ncores=5 (matching the labeled-mode Figure 6 benchmark configuration)
- Verify output files were written in the requested out_type (csv/txt/rds/all/none) to out_dir with out_name 'Bfimpute'
- Sanity-check that the returned R_calculate matrix has the same genes x cells dimensions as the input counts matrix
- Recompute silhouette scores with true cell type labels and same/different-cell-type Pearson correlations on the imputed matrix; the labeled benchmark expects Bfimpute to attain the best silhouette in all but one dataset
- If normalized=FALSE was left default, confirm CPM + log10 (bias 1.01) transform was applied inside the tool rather than pre-applied externally

## Limitations

- Larger D, totalepoch, and burnin increase accuracy but increase runtime proportionally; smaller values save time at an accuracy cost
- It is premature to conclude Bfimpute will achieve better performance across all datasets and improve downstream analysis — generalizability beyond benchmarked datasets is unconfirmed
- Depending on the application, users may need to constrain parameters to maximize true positives or minimize false positives; increased false positives in downstream analysis are possible
- The priors are fixed in the implementation (mu_0=0, beta_0=2, nu_0=D, W_0=I) and are not user-tunable

## Evidence
