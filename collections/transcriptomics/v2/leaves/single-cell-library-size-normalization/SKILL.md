---
name: single-cell-library-size-normalization
description: Use when raw single-cell gene expression counts (e.g., the featurecounts.qc
  slot of GBM10_preproc) have varying per-cell totals and must be depth-corrected
  before gene-level parameter estimation such as BPSC/SCALE bursting kinetics.
when_to_use_negative:
- Input is already normalized expression (e.g., TPM/CPM-like values) rather than raw
  counts — BPSC's internal library-size factor normalization assumes raw count data
- The task only involves bulk DNA WES read counts (Rb/Xb); bursting kinetics are estimated
  from single-cell gene expression only and are decoupled from SNV carrier status
- MCMC-based BPSC is infeasible due to dataset size — use the SCALE moment-estimator
  route (get_burstiness_scale) instead of forcing BPSC
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: canopy2
  role: R package whose get_burstiness_bpsc() and get_burstiness_scale() functions
    perform the library-size normalization and downstream bursting-kinetics estimation
  repo: https://github.com/annweideman/canopy2
- name: BPSC
  role: Beta-Poisson MCMC engine invoked internally by get_burstiness_bpsc(); installed
    via devtools::install_github('nghiavtr/BPSC')
- name: SCALE
  role: Moment-estimator methodology used by get_burstiness_scale() as the faster
    alternative for large datasets
- name: biomaRt
  role: Maps chromosome positions to HGNC symbols so expression gene order matches
    SNV rows
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1007/s12561-024-09466-1
    title: 'Canopy2: Tumor Phylogeny Inference by Bulk DNA and Single-Cell RNA Sequencing'
  source_content_hash: sha256:6173a6ce07056d70aba927599c1dd9124594d463fc61921c47099b346209bbd1
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
  - canopy2
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# Single-cell library-size normalization

## Summary

This skill normalizes single-cell RNA-seq count matrices by library-size factor so that gene-level rate estimation (here, beta-Poisson bursting kinetics alpha and beta in Canopy2) is not biased by sequencing depth differences across cells.

## When to use

Use when raw single-cell gene expression counts (e.g., the featurecounts.qc slot of GBM10_preproc) have varying per-cell totals and must be depth-corrected before gene-level parameter estimation such as BPSC/SCALE bursting kinetics.

## When NOT to use

- Input is already normalized expression (e.g., TPM/CPM-like values) rather than raw counts — BPSC's internal library-size factor normalization assumes raw count data
- The task only involves bulk DNA WES read counts (Rb/Xb); bursting kinetics are estimated from single-cell gene expression only and are decoupled from SNV carrier status
- MCMC-based BPSC is infeasible due to dataset size — use the SCALE moment-estimator route (get_burstiness_scale) instead of forcing BPSC

## Inputs

- Raw single-cell gene expression count matrix (e.g., the featurecounts.qc slot of GBM10_preproc)
- Mutation list / chromosome positions of SNV rows whose order the expression matrix must match
- biomaRt Ensembl annotation (build 37) for chromosome-position to HGNC symbol mapping

## Outputs

- Normalized-then-fit per-mutation gene activation rate vector (param.out$alpha)
- Per-mutation gene deactivation rate vector (param.out$beta)
- Vector of row identifiers with estimable bursting kinetics (param.out$id.g)

## How to apply

Load the raw single-cell count matrix and align its genes to the mutation-host genes of the read-count matrices, mapping chromosome positions to HGNC symbols with biomaRt (get_BM_fun / get_gene_expression, Ensembl build 37) so gene order matches the SNV rows. Rather than normalizing manually, run canopy2's get_burstiness_bpsc(), which performs library-size factor normalization internally and then fits each gene with a beta-Poisson model via MCMC sampling. For large datasets where BPSC's MCMC is too computationally intensive, substitute get_burstiness_scale(), which estimates the same alpha (activation rate) and beta (deactivation rate) using moment estimators from the SCALE methodology. Collect param.out$alpha, param.out$beta, and param.out$id.g marking the mutations whose host genes had estimable bursting kinetics.

## Related tools

- **canopy2** (R package whose get_burstiness_bpsc() and get_burstiness_scale() functions perform the library-size normalization and downstream bursting-kinetics estimation) — https://github.com/annweideman/canopy2
- **BPSC** (Beta-Poisson MCMC engine invoked internally by get_burstiness_bpsc(); installed via devtools::install_github('nghiavtr/BPSC'))
- **SCALE** (Moment-estimator methodology used by get_burstiness_scale() as the faster alternative for large datasets)
- **biomaRt** (Maps chromosome positions to HGNC symbols so expression gene order matches SNV rows)

## Examples

```
param.out <- get_burstiness_bpsc(counts = sims.out$G)
```

## Evaluation signals

- Every returned alpha and beta value is non-NA and non-negative (the paper notes BPSC had improved estimability of non-NA or negative values)
- id.g marks exactly the mutations whose host genes have estimable bursting kinetics
- Gene order of the expression matrix matches the SNV row order after biomaRt HGNC mapping (build 37)
- alpha and beta are 1 x M vectors with one entry per mutation, as required by canopy2's get_trees() inputs
- Running get_burstiness_scale() on the same input yields comparable alpha/beta estimates to BPSC for genes both methods can estimate

## Limitations

- BPSC's MCMC sampling is computationally intensive for large datasets; get_burstiness_scale() is the recommended fallback
- get_gene_expression() can fail with a BiocFileCache/dbplyr compatibility bug; reinstalling BiocFileCache via BiocManager::install('BiocFileCache') is the documented fix
- Estimation quality depends on the correctness of the chromosome-position to HGNC-symbol mapping and annotation version (build 37 in the paper)

## Evidence

- [methods] it performs library-size factor normalization internally and estimates each gene's bursting kinetics by MCMC sampling from a beta-Poisson model: [evidence span withheld — anchor status: not_found]
- [methods] One can use function `get_burstiness_bpsc()` or `get_burstiness_scale()` to estimate `alpha` and `beta`.: "One can use function `get_burstiness_bpsc()` or `get_burstiness_scale()` to estimate `alpha` and `beta`."
- [readme] `alpha` and `beta`:  1 x M (number of mutations) vectors of mutation specific gene activation and gene deactivation rates.: [evidence span withheld — anchor status: external_doc]
- [readme] This can be estimated using either the BPSC methodology in function `get_burstiness_bpsc()` (recommended) or the SCALE methodology: [evidence span withheld — anchor status: external_doc]
