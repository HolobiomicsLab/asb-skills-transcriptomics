---
name: beta-poisson-gene-expression-modeling
description: 'Use when preparing Canopy2 inputs from single-cell RNA-seq (e.g., Smart-seq2
  data such as GBM10_preproc): you need mutation-specific alpha/beta bursting rates
  to model stochastic zeros before running get_trees(), and you have a gene expression
  count matrix.'
when_to_use_negative:
- The Rs/Xs read-count matrices already have finalized alpha and beta supplied — the
  estimates simply pass through to get_trees(), so re-running burstiness estimation
  is redundant
- Only bulk DNA WES data (Rb/Xb) is available with no single-cell gene expression
  counts, since the beta-Poisson model is fitted to scRNA-seq expression
- You need the clonal tree itself rather than bursting hyperparameters — proceed directly
  to get_trees() once alpha/beta exist
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: Canopy2
  role: R package exposing get_burstiness_bpsc() and get_burstiness_scale() to estimate
    alpha/beta and id.g for its beta-Poisson zero model
  repo: https://github.com/annweideman/canopy2
- name: BPSC
  role: Recommended beta-Poisson bursting-kinetics estimator called internally by
    get_burstiness_bpsc(); fits via MCMC after internal library-size normalization
  repo: https://github.com/nghiavtr/BPSC
- name: SCALE
  role: Moment-estimator alternative used by get_burstiness_scale() when BPSC is computationally
    prohibitive on large datasets
- name: biomaRt
  role: Maps chromosome positions to HGNC symbols (build 37) via get_BM_fun/get_gene_expression
    so gene expression rows align with SNV rows
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s12561-024-09466-1
    title: 'Canopy2: Tumor Phylogeny Inference by Bulk DNA and Single-Cell RNA Sequencing'
  source_content_hash: sha256:6641b8f887aabeb77ac8fa5e98bf5181f8d052e57df9372f71dc06016685b8bb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/annweideman/canopy2
  tools:
  - BPSC
  - Canopy2
  - R
  - SCALE
  - biomaRt
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# beta-poisson-gene-expression-modeling

## Summary

Estimates mutation-specific transcriptional bursting kinetics — gene activation rate alpha and gene deactivation rate beta — by fitting a beta-Poisson model to single-cell gene expression counts using BPSC (MCMC) or SCALE (moment estimators), as implemented in Canopy2's get_burstiness_bpsc() and get_burstiness_scale(). These 1 x M hyperparameters model the stochastic (bursting-driven) zeros in scRNA-seq and are passed downstream to get_trees().

## When to use

Use when preparing Canopy2 inputs from single-cell RNA-seq (e.g., Smart-seq2 data such as GBM10_preproc): you need mutation-specific alpha/beta bursting rates to model stochastic zeros before running get_trees(), and you have a gene expression count matrix.

## When NOT to use

- The Rs/Xs read-count matrices already have finalized alpha and beta supplied — the estimates simply pass through to get_trees(), so re-running burstiness estimation is redundant
- Only bulk DNA WES data (Rb/Xb) is available with no single-cell gene expression counts, since the beta-Poisson model is fitted to scRNA-seq expression
- You need the clonal tree itself rather than bursting hyperparameters — proceed directly to get_trees() once alpha/beta exist

## Inputs

- Single-cell gene expression count matrix (e.g., featurecounts.qc slot of GBM10_preproc), full-transcript scRNA-seq such as Smart-seq2/Smart-seq3
- M x N alternative (Rs) and total (Xs) read-count matrices whose rows correspond to SNV host genes
- Chromosome positions of mutations and gene annotation resources via biomaRt (Ensembl build 37, HGNC symbols)

## Outputs

- param.out$alpha: 1 x M vector of mutation-specific gene activation rates
- param.out$beta: 1 x M vector of mutation-specific gene deactivation rates
- param.out$id.g: vector of row identifiers for mutations with estimable bursting kinetics
- Alpha/beta vectors ready to be passed to get_trees() alongside Rs, Xs, Rb, Xb, kappa=1, tau=999, and Klist

## How to apply

Load the single-cell gene expression count matrix (e.g., the featurecounts.qc slot of GBM10_preproc) and align it to the mutation-host genes of the read-count matrices, mapping chromosome positions to HGNC symbols with biomaRt (get_BM_fun / get_gene_expression, Ensembl build 37) so gene order matches the SNV rows. Call get_burstiness_bpsc(counts=...), which performs library-size factor normalization internally and estimates each gene's activation rate, deactivation rate, and transcription-rate scale per mutation by MCMC sampling from a beta-Poisson model; BPSC is the recommended estimator per the README. For large datasets where BPSC's MCMC is too computationally intensive, substitute get_burstiness_scale(), which derives alpha and beta with SCALE's moment estimators. Collect param.out$alpha, param.out$beta, and the row-identifier vector param.out$id.g marking mutations with estimable kinetics, then subset the alternative (Rs) and total (Xs) read counts to those id.g rows. Rationale: Canopy2 separates zeros into non-cancerous, stochastic (bursting-driven), and technical sources, and the beta-Poisson alpha/beta hyperparameters are estimated from gene expression independent of mutational read counts, so they are decoupled from SNV carrier status.

## Related tools

- **Canopy2** (R package exposing get_burstiness_bpsc() and get_burstiness_scale() to estimate alpha/beta and id.g for its beta-Poisson zero model) — https://github.com/annweideman/canopy2
- **BPSC** (Recommended beta-Poisson bursting-kinetics estimator called internally by get_burstiness_bpsc(); fits via MCMC after internal library-size normalization) — https://github.com/nghiavtr/BPSC
- **SCALE** (Moment-estimator alternative used by get_burstiness_scale() when BPSC is computationally prohibitive on large datasets)
- **biomaRt** (Maps chromosome positions to HGNC symbols (build 37) via get_BM_fun/get_gene_expression so gene expression rows align with SNV rows)

## Examples

```
param.out <- get_burstiness_bpsc(counts = sims.out$G); alpha <- param.out$alpha; beta <- param.out$beta; id.g <- param.out$id.g
```

## Evaluation signals

- Every returned alpha and beta value is non-NA and non-negative (the paper notes BPSC had improved estimability of non-NA or negative values)
- param.out$id.g marks exactly the mutations for which bursting kinetics are estimable, and Rs/Xs rows subset to these identifiers
- Gene expression matrix row order matches the SNV rows of Rs/Xs after biomaRt chromosome-position-to-HGNC mapping
- Alpha and beta vectors have length equal to the number of mutations M, suitable as 1 x M inputs to get_trees()
- get_burstiness_bpsc() applied to count data (e.g., sims.out$G) returns per-gene alpha, beta, and transcription-rate scale without external normalization, confirming internal library-size normalization ran

## Limitations

- BPSC's MCMC sampling is computationally intensive on large datasets; get_burstiness_scale() with SCALE moment estimators is the fallback but is only described as an alternative, not as equivalent in accuracy
- get_gene_expression() internally uses biomaRt::useEnsembl(), which can fail with a db_collect() error due to BiocFileCache/dbplyr incompatibility; fix by reinstalling BiocFileCache via BiocManager::install("BiocFileCache")
- The bursting hyperparameters are estimated from gene expression and are thus decoupled from SNV carrier status — they cannot be used to infer which cells carry a mutation; as an option the rates can be estimated from germline mutations within the same gene body as somatic mutations

## Evidence

- [methods] Function choice for burstiness estimation: "One can use function `get_burstiness_bpsc()` or `get_burstiness_scale()` to estimate `alpha` and `beta`."
- [methods] Downstream use of alpha/beta: "The estimates for $\alpha$ and $\beta$ are passed to `get_trees()`, which outputs trees, posteriors, and Metropolis-Hastings acceptance rate"
- [readme] README definition of alpha/beta inputs: [evidence span withheld — anchor status: external_doc]
- [readme] README recommendation of BPSC over SCALE: [evidence span withheld — anchor status: external_doc]
- [abstract] Stochastic zeros modeled by bursting: "Canopy2 demystifies the sources of zeros in the single-cell data and separates zeros categorized as non-cancerous (cells without mutations), stochastic (mutations not expressed due to bursting)"
- [discussion] Optional germline-based estimation: "should the user choose to estimate the gene activation and deactivation rates"
