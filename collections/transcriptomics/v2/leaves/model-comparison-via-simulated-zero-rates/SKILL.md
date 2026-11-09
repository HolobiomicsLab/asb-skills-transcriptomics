---
name: model-comparison-via-simulated-zero-rates
description: Use when evaluating whether a Bayesian count model's zero-inflation (dropout)
  component is needed, by checking if posterior-simulated zero rates reproduce observed
  zero-count rates per batch.
when_to_use_negative:
- Data already normalized or corrected (e.g., corrected_read_counts output), since
  the check must run on raw counts where zero rates are meaningful
- Data assumed free of dropout (e.g., UMI-only data with low zero rates), where the
  dropout component contrast is not informative
- When the goal is clustering or batch correction itself rather than assessing model
  fit of zero inflation
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_2269
tools:
- name: BUSseq
  role: Bayesian hierarchical model fitted via BUSseq_MCMC to obtain posterior samples
    used for posterior predictive simulation; refit with dropout coefficient fixed
    at zero to form the BUSseq-nzf comparison model
  repo: https://github.com/songfd2018/BUSseq
- name: BUSseq-Rpackage
  role: R package providing BUSseq_MCMC with parameters such as Drop_ind, n.iterations,
    n.burnin, and n.cores used to run both model variants
  repo: https://github.com/songfd2018/BUSseq-Rpackage
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-16905-2
    title: Flexible experimental designs for valid single-cell RNA-sequencing experiments
      allowing batch effects correction
  source_content_hash: sha256:4929e82561b7bfea6b6383c29c386da1634864aef868f3dc69e124c2d0e2105b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/songfd2018/BUSseq
  tools:
  - BUSseq
  - BUSseq-Rpackage
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# model-comparison-via-simulated-zero-rates

## Summary

A posterior predictive check that compares competing scRNA-seq count models (full BUSseq with zero-inflated dropout vs. the reduced BUSseq-nzf) by simulating replicated datasets from each posterior and comparing per-batch zero rates against the observed data.

## When to use

Use when evaluating whether a Bayesian count model's zero-inflation (dropout) component is needed, by checking if posterior-simulated zero rates reproduce observed zero-count rates per batch.

## When NOT to use

- Data already normalized or corrected (e.g., corrected_read_counts output), since the check must run on raw counts where zero rates are meaningful
- Data assumed free of dropout (e.g., UMI-only data with low zero rates), where the dropout component contrast is not informative
- When the goal is clustering or batch correction itself rather than assessing model fit of zero inflation

## Inputs

- Raw read count matrices (genes x cells) of the mouse hematopoietic study and human pancreas study (RawCountData folders)
- Dimension and metadata files (batch labels, cell counts per batch) in the BUSseq input format (count_data, dim, metadata .txt files)
- Post-burn-in posterior MCMC samples of cell-type mean expression, overdispersion, and dropout parameters from full BUSseq and BUSseq-nzf fits

## Outputs

- Per-batch observed zero rates of the raw data
- Posterior-mean zero rates across 4000 posterior predictive simulations for each model (full BUSseq and BUSseq-nzf) per batch
- Comparison table (e.g., table1_zero_rates.csv) of observed vs. model-implied zero rates

## How to apply

Fit the full BUSseq model with BUSseq_MCMC (Drop_ind enabling the zero-inflated dropout component, n.iterations=4000 with burn-in, n.cores for parallel chains) on the read count matrices, then refit the reduced BUSseq-nzf model in which the dropout coefficient is fixed at zero. Extract post-burn-in posterior samples of cell-type mean expression, overdispersion, and dropout parameters from each fit. Simulate 4000 replicated count datasets from the posterior predictive distribution of each fitted model per batch and compute per-batch zero rates of the raw data plus posterior-mean zero rates across simulations. Assemble the comparison (e.g., hematopoietic MARS-seq: 0.892 observed vs 0.887 BUSseq vs 0.874 BUSseq-nzf) into a table such as table1_zero_rates.csv. The rationale is that a model with a proper dropout component should closely track observed zero rates, especially under non-UMI protocols like SMART-seq2 where dropouts are frequent.

## Related tools

- **BUSseq** (Bayesian hierarchical model fitted via BUSseq_MCMC to obtain posterior samples used for posterior predictive simulation; refit with dropout coefficient fixed at zero to form the BUSseq-nzf comparison model) — https://github.com/songfd2018/BUSseq
- **BUSseq-Rpackage** (R package providing BUSseq_MCMC with parameters such as Drop_ind, n.iterations, n.burnin, and n.cores used to run both model variants) — https://github.com/songfd2018/BUSseq-Rpackage

## Examples

```
fit <- BUSseq_MCMC(ObservedData = counts, n.celltypes = K, seed = 1, n.iterations = 4000, n.burnin = 2000, n.cores = 4, working_dir = "./", Drop_ind = TRUE)
```

## Evaluation signals

- Posterior-mean zero rates from full BUSseq must closely match observed zero rates in every batch (e.g., 0.887 vs 0.892 for hematopoietic MARS-seq)
- BUSseq-nzf must visibly underestimate observed zero rates under non-UMI protocols (e.g., 0.874 vs 0.892 on hematopoietic MARS-seq), demonstrating dropout-model misfit
- Exactly 4000 replicated datasets simulated per fitted model per batch, using post-burn-in posterior samples
- Both model variants must be fit on identical count matrices with the only difference being the dropout coefficient fixed at zero in BUSseq-nzf
- The resulting table (table1_zero_rates.csv) must contain one row per batch with observed, BUSseq, and BUSseq-nzf zero-rate columns

## Limitations

- MCMC algorithms are well-known for heavy computation load, so simulating 4000 posterior predictive replicates per model is computationally expensive
- The C++/compiled source codes support Linux operating systems only (tested on Ubuntu 18.04 and CentOS 7)
- The largest BUSseq-nzf discrepancies occur on batches assayed by non-UMI protocols such as SMART-seq2, so the contrast is less diagnostic for UMI-based batches

## Evidence

- [methods] BUSseq_MCMC signature with zero-inflation toggle: "`BUSseq_MCMC(ObservedData, n.celltypes, seed, n.cores, n.iterations, n.burnin, n.unchanged, n.output, working_dir, Drop_ind, fdr, hyper_pi, hyper_gamma0, hyper_gamma1, hyper_alpha, tau1sq, hyper_p,`"
- [abstract] BUSseq models dropout events: [evidence span withheld — anchor status: not_found]
- [discussion] MCMC computational cost caveat: "Although MCMC algorithms are well-known for heavy computation load, fortunately, the computational complexity of BUSseq is"
- [readme] Downloading the hematopoietic and pancreas raw count data: [evidence span withheld — anchor status: external_doc]
- [readme] MCMC iteration and burn-in settings: [evidence span withheld — anchor status: not_found]
