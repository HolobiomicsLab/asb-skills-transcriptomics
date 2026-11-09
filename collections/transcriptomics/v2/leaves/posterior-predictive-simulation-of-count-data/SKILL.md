---
name: posterior-predictive-simulation-of-count-data
description: Use when you have posterior samples from a count-based scRNA-seq model
  (e.g., BUSseq fits of hematopoietic/pancreas read counts) and need to test whether
  the model reproduces observed zero-count rates, or whether dropout events must be
  explicitly modeled.
when_to_use_negative:
- Data are UMI counts with near-zero dropout so a zero-inflated dropout component
  is unnecessary and BUSseq-nzf suffices
- The fitted model does not provide posterior samples of the negative-binomial mean,
  overdispersion, and dropout parameters needed to simulate replicates
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_2269
tools:
- name: BUSseq
  role: Fits the Bayesian hierarchical model via BUSseq_MCMC with Drop_ind toggling
    the zero-inflated dropout component; posterior samples are used for posterior
    predictive simulation of replicated counts
  repo: https://github.com/songfd2018/BUSseq
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-16905-2
    title: Flexible experimental designs for valid single-cell RNA-sequencing experiments
      allowing batch effects correction
  source_content_hash: sha256:e26adacf0be7d242f7ef942b385e6f5cff51cb965323eafbb456002c489165ec
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/songfd2018/BUSseq
  tools:
  - BUSseq
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# posterior-predictive-simulation-of-count-data

## Summary

Simulate replicated read-count datasets from the posterior predictive distribution of a fitted Bayesian hierarchical model (full BUSseq with zero-inflated dropout component vs. reduced BUSseq-nzf) and compare per-batch zero rates against observed zero rates, as a posterior predictive check that dropout is modeled adequately.

## When to use

Use when you have posterior samples from a count-based scRNA-seq model (e.g., BUSseq fits of hematopoietic/pancreas read counts) and need to test whether the model reproduces observed zero-count rates, or whether dropout events must be explicitly modeled.

## When NOT to use

- Data are UMI counts with near-zero dropout so a zero-inflated dropout component is unnecessary and BUSseq-nzf suffices
- The fitted model does not provide posterior samples of the negative-binomial mean, overdispersion, and dropout parameters needed to simulate replicates

## Inputs

- Raw read count matrices of the mouse hematopoietic study and human pancreas study (G genes x N cells, with per-batch cell counts in a dim file)
- Post-burn-in posterior samples of cell-type mean expression, overdispersion, and dropout parameters from each BUSseq_MCMC fit (full BUSseq and BUSseq-nzf)
- Per-cell batch labels

## Outputs

- 4000 replicated posterior predictive count datasets per model per batch
- Per-batch observed zero rates and posterior-mean zero rates for full BUSseq and BUSseq-nzf
- table1_zero_rates.csv comparing observed vs. model-implied zero rates

## How to apply

Fit the full BUSseq model with BUSseq_MCMC (Drop_ind enabling the zero-inflated dropout component, n.iterations=4000 with burn-in, n.cores for parallel chains) and refit the reduced BUSseq-nzf model in which the dropout coefficient is fixed at zero. Extract post-burn-in posterior samples of cell-type mean expression, overdispersion, and dropout parameters from each fit. Simulate 4000 replicated count datasets per batch from the posterior predictive distribution of each model, compute per-batch observed zero rates of the raw data and posterior-mean zero rates across simulations, then assemble the comparison table (e.g., hematopoietic MARS-seq: 0.892 observed vs 0.887 BUSseq vs 0.874 BUSseq-nzf). The rationale is that a model ignoring dropout events should visibly underestimate zero rates, especially in batches assayed by non-UMI protocols such as SMART-seq2.

## Related tools

- **BUSseq** (Fits the Bayesian hierarchical model via BUSseq_MCMC with Drop_ind toggling the zero-inflated dropout component; posterior samples are used for posterior predictive simulation of replicated counts) — https://github.com/songfd2018/BUSseq

## Examples

```
BUSseq_MCMC(ObservedData, n.celltypes=K, seed, n.cores=4, n.iterations=4000, n.burnin, working_dir, Drop_ind=TRUE)
```

## Evaluation signals

- Posterior-mean zero rates from full BUSseq must closely match observed zero rates in every batch (e.g., 0.887 vs 0.892 for hematopoietic MARS-seq)
- BUSseq-nzf must visibly underestimate zero rates under non-UMI protocols (e.g., 0.874 vs 0.892 on hematopoietic MARS-seq)
- Simulation count matches 4000 replicates per fitted model per batch
- Output table1_zero_rates.csv contains one row per batch with observed, BUSseq, and BUSseq-nzf zero-rate columns
- MCMC chains used n.iterations=4000 with burn-in and reproducible seed settings in BUSseq_MCMC

## Limitations

- MCMC inference is computationally heavy, though most steps of the MCMC algorithm are parallelizable (n.cores) and a GPU implementation (BUSseq_gpu) exists
- The C++ source codes support Linux operating systems only (tested on Ubuntu 18.04 and CentOS 7)
- Largest model misfit for the non-zero-inflated model occurs specifically on non-UMI (SMART-seq2) batches, so the check is most discriminative there

## Evidence

- [methods] BUSseq_MCMC(ObservedData, n.celltypes, seed, n.cores, n.iterations, n.burnin, n.unchanged, n.output, working_dir, Drop_ind, fdr, hyper_pi, hyper_gamma0, hyper_gamma1, hyper_alpha, tau1sq, hyper_p,: "BUSseq_MCMC(ObservedData, n.celltypes, seed, n.cores, n.iterations, n.burnin, n.unchanged, n.output, working_dir, Drop_ind, fdr, hyper_pi, hyper_gamma0, hyper_gamma1, hyper_alpha, tau1sq, hyper_p,"
- [discussion] most steps of the MCMC algorithm for BUSseq are parallelizable (Supplementary Note 12).: "most steps of the MCMC algorithm for BUSseq are parallelizable (Supplementary Note 12)."
- [readme] BUSseq is able to simultaneously correct batch effects, clusters cell types, and takes care of the count data nature, the overdispersion, the dropout events, and the cell-specific sequencing depth of scRNA-seq data.: "BUSseq is able to simultaneously correct batch effects, clusters cell types, and takes care of the count data nature, the overdispersion, the dropout events, and the cell-specific sequencing depth of"
- [readme] Please download the raw count data of the mouse hematopoietic study and the human pancreas study: [evidence span withheld — anchor status: external_doc]
