---
name: mcmc-based-hyperparameter-inference
description: Use when Canopy2 requires the 1 x M alpha and beta hyperparameter vectors
  and single-cell expression counts are available for the mutation-host genes (e.g.,
  the featurecounts.qc slot of GBM10_preproc).
when_to_use_negative:
- The goal is to estimate bursting kinetics from the SNV read-count matrices themselves
  — alpha/beta are derived from gene expression and are independent of mutational
  read counts, hence decoupled from SNV carrier status
- The host gene's expression cannot be aligned to the SNV row via biomaRt (no estimable
  kinetics) — the row should be excluded via id.g subsetting rather than forced through
  BPSC
- Only bulk DNA WES read counts (Rb/Xb) are available with no scRNA-seq expression
  matrix — BPSC/SCALE require single-cell expression counts
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0622
tools:
- name: Canopy2
  role: Provides get_burstiness_bpsc() and get_burstiness_scale(), which produce the
    alpha/beta hyperparameters consumed by get_trees()
  repo: https://github.com/annweideman/canopy2
- name: BPSC
  role: Underlying beta-Poisson MCMC machinery invoked by get_burstiness_bpsc() to
    estimate bursting kinetics
- name: SCALE
  role: Moment-estimator methodology underlying get_burstiness_scale() for large datasets
    where BPSC is infeasible
- name: biomaRt
  role: Maps chromosome positions to HGNC symbols (get_gene_expression, build 37)
    so expression rows match SNV rows
- name: R
  role: Implementation language; canopy2 installed via devtools::install_github('annweideman/canopy2')
  repo: https://github.com/annweideman/canopy2
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s12561-024-09466-1
    title: 'Canopy2: Tumor Phylogeny Inference by Bulk DNA and Single-Cell RNA Sequencing'
  source_content_hash: sha256:223b923b8aa22b268f7dc0ca37e3c4b3f5569088389dbcacba665096184bb98e
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
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# mcmc-based-hyperparameter-inference

## Summary

Estimates mutation-specific gene activation (alpha) and deactivation (beta) rates for Canopy2's beta-Poisson bursting model, using BPSC's MCMC sampling (get_burstiness_bpsc) or SCALE's moment estimators (get_burstiness_scale) on single-cell gene expression, decoupled from SNV read counts.

## When to use

Use when Canopy2 requires the 1 x M alpha and beta hyperparameter vectors and single-cell expression counts are available for the mutation-host genes (e.g., the featurecounts.qc slot of GBM10_preproc).

## When NOT to use

- The goal is to estimate bursting kinetics from the SNV read-count matrices themselves — alpha/beta are derived from gene expression and are independent of mutational read counts, hence decoupled from SNV carrier status
- The host gene's expression cannot be aligned to the SNV row via biomaRt (no estimable kinetics) — the row should be excluded via id.g subsetting rather than forced through BPSC
- Only bulk DNA WES read counts (Rb/Xb) are available with no scRNA-seq expression matrix — BPSC/SCALE require single-cell expression counts

## Inputs

- Single-cell gene expression count matrix (e.g., GBM10_preproc featurecounts.qc slot) aligned to mutation-host genes via biomaRt (build 37) HGNC mapping
- M x N alternative (Rs) and total (Xs) read-count matrices whose rows define the mutations needing alpha/beta estimates

## Outputs

- param.out$alpha: 1 x M vector of mutation-specific gene activation rates
- param.out$beta: 1 x M vector of mutation-specific gene deactivation rates
- param.out$id.g: vector of row identifiers (mutations) for which bursting kinetics are estimable
- Per-gene transcription-rate scale estimates returned by BPSC/SCALE

## How to apply

First align the scRNA-seq expression count matrix to the mutation rows of the Rs/Xs read-count matrices by mapping chromosome positions to HGNC symbols with biomaRt (get_BM_fun / get_gene_expression, Ensembl build 37), so gene order matches the SNV rows. Run get_burstiness_bpsc() on the expression matrix; it normalizes library-size factors internally and samples each gene's bursting kinetics from a beta-Poisson model via MCMC. For large datasets where BPSC's MCMC is too computationally intensive, substitute get_burstiness_scale(), which uses SCALE moment estimators. Collect param.out$alpha and param.out$beta as the 1 x M vectors and param.out$id.g as the identifiers of mutations with estimable kinetics, then pass them to get_trees() together with kappa=1, tau=999 (sequencing error kappa/(kappa+tau) = 0.001) and a Klist range for BIC-based subclone selection.

## Related tools

- **Canopy2** (Provides get_burstiness_bpsc() and get_burstiness_scale(), which produce the alpha/beta hyperparameters consumed by get_trees()) — https://github.com/annweideman/canopy2
- **BPSC** (Underlying beta-Poisson MCMC machinery invoked by get_burstiness_bpsc() to estimate bursting kinetics)
- **SCALE** (Moment-estimator methodology underlying get_burstiness_scale() for large datasets where BPSC is infeasible)
- **biomaRt** (Maps chromosome positions to HGNC symbols (get_gene_expression, build 37) so expression rows match SNV rows)
- **R** (Implementation language; canopy2 installed via devtools::install_github('annweideman/canopy2')) — https://github.com/annweideman/canopy2

## Examples

```
param.out<-get_burstiness_bpsc(counts=sims.out$G)
```

## Evaluation signals

- Every returned alpha and beta value is non-NA and non-negative (the paper notes BPSC had improved estimability of non-NA or negative values)
- param.out$id.g marks exactly the mutations whose host-gene bursting kinetics were estimable, and alternative/total read counts can be subset to those rows
- Length of alpha and beta equals the number of mutations M in the Rs/Xs matrices, with row order consistent after the biomaRt HGNC alignment
- Downstream get_trees() accepts the vectors without dimension errors alongside kappa=1, tau=999 (sequencing error 0.1%) and a Klist range

## Limitations

- BPSC's MCMC sampling is computationally intensive on large datasets; get_burstiness_scale() with moment estimators is the fallback
- get_gene_expression() relies on biomaRt::useEnsembl(); a BiocFileCache/dbplyr compatibility bug can throw a 'Failed to collect lazy table' error, fixed by reinstalling BiocFileCache via BiocManager
- Not all genes/mutations yield estimable bursting kinetics; only rows flagged in id.g should feed the beta-Poisson model

## Evidence

- [methods] One can use function get_burstiness_bpsc() or get_burstiness_scale() to estimate alpha and beta.: "One can use function `get_burstiness_bpsc()` or `get_burstiness_scale()` to estimate `alpha` and `beta`."
- [methods] The estimates for alpha and beta are passed to get_trees() which outputs trees, posteriors, and Metropolis-Hastings acceptance rate.: "The estimates for $\alpha$ and $\beta$ are passed to `get_trees()`, which outputs trees, posteriors, and Metropolis-Hastings acceptance rate"
- [readme] These can be estimated using either the BPSC methodology in function get_burstiness_bpsc() (recommended) or the SCALE methodology with function get_burstiness_scale().: [evidence span withheld — anchor status: external_doc]
- [discussion] Estimate gene activation and deactivation rates from germline mutations within same gene body as somatic mutations.: [evidence span withheld — anchor status: not_found]
