---
name: bursting-kinetics-parameter-estimation
description: Use when fitting Canopy2's beta-Poisson bursting model and mutation-specific
  gene activation/deactivation rates (alpha, beta) must be estimated from single-cell
  gene expression counts (e.g. featurecounts.qc of GBM10_preproc) before running get_trees().
when_to_use_negative:
- Alpha and beta are already supplied as 1 x M vectors (e.g. from a prior run or simulation
  settings like alpha=0.1, beta=1) — re-estimation is unnecessary
- No single-cell gene expression matrix exists for the mutation-host genes (e.g. bulk
  DNA-only input), since both BPSC and SCALE require gene expression counts
- The dataset is very large and a full MCMC-based fit is infeasible without falling
  back to get_burstiness_scale() — do not default to BPSC when runtime is prohibitive
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_0091
- http://edamontology.org/topic_3307
tools:
- name: Canopy2
  role: Host package providing get_burstiness_bpsc() and get_burstiness_scale(), which
    estimate the alpha/beta vectors passed to get_trees()
  repo: https://github.com/annweideman/canopy2
- name: BPSC
  role: Beta-Poisson model fit by MCMC used internally by get_burstiness_bpsc() for
    the recommended bursting-kinetics estimation
  repo: https://github.com/nghiavtr/BPSC
- name: SCALE
  role: Moment-estimator methodology used internally by get_burstiness_scale() as
    a faster alternative for large datasets
- name: biomaRt
  role: Maps chromosome positions of SNVs to HGNC gene symbols (build 37) via get_gene_expression()/get_BM_fun
    so gene expression matches SNV rows
provenance:
  source_task_ids:
  - task_004
  - task_002
  source_papers:
  - doi: 10.1007/s12561-024-09466-1
    title: 'Canopy2: Tumor Phylogeny Inference by Bulk DNA and Single-Cell RNA Sequencing'
  source_content_hash: sha256:ff09bbfe5338faf3d8c4c88cd391670693f103b8a3039f111f3331172a40dc38
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/annweideman/canopy2
  tools:
  - BPSC
  - Canopy
  - Canopy2
  - Cardelino
  - R
  - SCALE
  - biomaRt
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_0091
  - http://edamontology.org/topic_3307
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# bursting-kinetics-parameter-estimation

## Summary

Estimates mutation-specific gene activation and deactivation rates (alpha, beta) that parameterize transcriptional bursting in Canopy2's beta-Poisson model, using BPSC (MCMC) or SCALE (moment estimators) fitted to single-cell gene expression counts, independent of the mutational read counts.

## When to use

Use when fitting Canopy2's beta-Poisson bursting model and mutation-specific gene activation/deactivation rates (alpha, beta) must be estimated from single-cell gene expression counts (e.g. featurecounts.qc of GBM10_preproc) before running get_trees().

## When NOT to use

- Alpha and beta are already supplied as 1 x M vectors (e.g. from a prior run or simulation settings like alpha=0.1, beta=1) — re-estimation is unnecessary
- No single-cell gene expression matrix exists for the mutation-host genes (e.g. bulk DNA-only input), since both BPSC and SCALE require gene expression counts
- The dataset is very large and a full MCMC-based fit is infeasible without falling back to get_burstiness_scale() — do not default to BPSC when runtime is prohibitive

## Inputs

- Single-cell gene expression count matrix (e.g. featurecounts.qq/featurecounts.qc slot of GBM10_preproc), genes x cells
- Mutation-host-gene mapping linking SNV rows (chromosome positions) to HGNC gene symbols via biomaRt (build 37)
- M x N alternative read count matrix (Rs) and total read count matrix (Xs) from scRNA-seq for downstream subsetting to id.g

## Outputs

- param.out$alpha: 1 x M vector of mutation-specific gene activation rates
- param.out$beta: 1 x M vector of mutation-specific gene deactivation rates
- param.out$id.g: vector of row identifiers for mutations whose bursting kinetics are estimable
- Subsetted Rs/Xs matrices restricted to mutations in id.g

## How to apply

Align the single-cell gene expression count matrix (e.g. the featurecounts.qq/featurecounts.qc slot of GBM10_preproc) to the mutation-host genes of the SNV read-count matrices by mapping chromosome positions to HGNC symbols with biomaRt (get_BM_fun/get_gene_expression, Ensembl build 37) so gene order matches the SNV rows. Call get_burstiness_bpsc() on the expression matrix: it normalizes for library-size factors internally and estimates each gene's bursting kinetics via MCMC sampling from a beta-Poisson model, recommended as the default. For large datasets where BPSC is too computationally intensive, substitute get_burstiness_scale(), which estimates alpha and beta with SCALE moment estimators. Collect param.out$alpha and param.out$beta (1 x M vectors, one entry per mutation) and param.out$id.g marking the mutations whose host-gene burstiness is estimable, then subset the alternative (Rs) and total (Xs) read counts to those mutations before passing the parameters to get_trees(). The paper suggests estimating these rates from germline mutations within the same gene body as the somatic mutations, which decouples the bursting estimates from SNV carrier status.

## Related tools

- **Canopy2** (Host package providing get_burstiness_bpsc() and get_burstiness_scale(), which estimate the alpha/beta vectors passed to get_trees()) — https://github.com/annweideman/canopy2
- **BPSC** (Beta-Poisson model fit by MCMC used internally by get_burstiness_bpsc() for the recommended bursting-kinetics estimation) — https://github.com/nghiavtr/BPSC
- **SCALE** (Moment-estimator methodology used internally by get_burstiness_scale() as a faster alternative for large datasets)
- **biomaRt** (Maps chromosome positions of SNVs to HGNC gene symbols (build 37) via get_gene_expression()/get_BM_fun so gene expression matches SNV rows)

## Examples

```
param.out<-get_burstiness_bpsc(counts=sims.out$G)
```

## Evaluation signals

- Every returned alpha and beta value is non-NA and non-negative
- param.out$id.g marks exactly the mutations whose host-gene burstiness is estimable, and Rs/Xs are subset to those rows
- Gene expression matrix order matches the SNV rows after biomaRt HGNC-symbol alignment (no NA symbol mappings from build 37)
- Estimated alpha/beta are 1 x M vectors (one entry per mutation) and downstream get_trees() accepts them without dimension errors
- Library-size normalization was applied internally (BPSC performs this during the beta-Poisson fit)

## Limitations

- BPSC is computationally intensive (MCMC sampling) on large datasets; get_burstiness_scale() with moment estimators is the fallback
- The paper notes BPSC had improved estimability of non-NA or negative values — some genes/mutations may yield non-estimable kinetics and are dropped via id.g
- Rates are decoupled from SNV carrier status (estimated from gene expression, e.g. germline mutations in the same gene body), so they reflect host-gene bursting, not the somatic SNV itself
- get_gene_expression() can fail with a dbplyr/BiocFileCache incompatibility bug; the README recommends reinstalling BiocFileCache via BiocManager

## Evidence

- [methods] One can use function `get_burstiness_bpsc()` or `get_burstiness_scale()` to estimate `alpha` and `beta`.: "One can use function `get_burstiness_bpsc()` or `get_burstiness_scale()` to estimate `alpha` and `beta`."
- [methods] BPSC parameter passing to get_trees: "The estimates for $\alpha$ and $\beta$ are passed to `get_trees()`, which outputs trees, posteriors, and Metropolis-Hastings acceptance rate"
- [readme] README alpha/beta description: [evidence span withheld — anchor status: external_doc]
- [readme] README input dimensionality: [evidence span withheld — anchor status: external_doc]
