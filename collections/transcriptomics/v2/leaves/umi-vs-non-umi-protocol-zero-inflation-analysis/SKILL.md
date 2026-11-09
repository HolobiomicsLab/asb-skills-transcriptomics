---
name: umi-vs-non-umi-protocol-zero-inflation-analysis
description: Use when validating that a Bayesian scRNA-seq count model reproduces
  observed batch-level zero rates, or when assessing whether an explicit zero-inflated
  dropout component is justified versus a plain negative-binomial fit.
when_to_use_negative:
- The goal is batch-effect-corrected expression for downstream clustering or DE rather
  than model validation of zero rates — use corrected_read_counts/imputed_read_counts
  outputs instead
- Data are UMI-based with low intrinsic zero rates where the dropout component is
  unnecessary; the discrepancy documented in Table 1 is largest for non-UMI protocols
  such as SMART-seq2
- A different generative model (not BUSseq/BUSseq-nzf) is being evaluated — this check
  depends on extracting BUSseq_MCMC posterior parameters
edam_operation: http://edamontology.org/operation_3658
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0121
tools:
- name: BUSseq
  role: Fit the full zero-inflated Bayesian hierarchical model via BUSseq_MCMC (Drop_ind
    = TRUE) and the reduced BUSseq-nzf variant (dropout coefficient fixed at zero);
    its posterior samples supply the parameters for simulating the 4000 posterior
    predictive replicates
  repo: https://github.com/songfd2018/BUSseq-Rpackage
- name: BUSseq-1.1_implementation
  role: Reproduction code repository providing access to the mouse hematopoietic and
    human pancreas raw count data used for the Table 1 zero-rate comparison
  repo: https://github.com/songfd2018/BUSseq-1.1_implementation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-16905-2
    title: Flexible experimental designs for valid single-cell RNA-sequencing experiments
      allowing batch effects correction
  source_content_hash: sha256:874557ba93a35e8743b0ceb0767c6b014124c7f35b1d1e232f286708ee1e6a81
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/songfd2018/BUSseq-Rpackage
  tools:
  - BUSseq
  - BUSseq-1.1_implementation
  edam_operation: http://edamontology.org/operation_3658
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0121
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9
n_claims: 10
---

# umi-vs-non-umi-protocol-zero-inflation-analysis

## Summary

A posterior predictive check of zero-count rates that contrasts the full zero-inflated BUSseq model against the reduced BUSseq-nzf model (dropout coefficient fixed at zero), demonstrating that explicit dropout modeling is needed to reproduce observed zero rates, especially under non-UMI protocols such as SMART-seq2.

## When to use

Use when validating that a Bayesian scRNA-seq count model reproduces observed batch-level zero rates, or when assessing whether an explicit zero-inflated dropout component is justified versus a plain negative-binomial fit.

## When NOT to use

- The goal is batch-effect-corrected expression for downstream clustering or DE rather than model validation of zero rates — use corrected_read_counts/imputed_read_counts outputs instead
- Data are UMI-based with low intrinsic zero rates where the dropout component is unnecessary; the discrepancy documented in Table 1 is largest for non-UMI protocols such as SMART-seq2
- A different generative model (not BUSseq/BUSseq-nzf) is being evaluated — this check depends on extracting BUSseq_MCMC posterior parameters

## Inputs

- Raw read count matrices (G genes x N cells) of the mouse hematopoietic study (MARS-seq batches) and the human pancreas study, e.g. the RawCountData folders from songfd2018/BUSseq-1.1_implementation
- Post-burn-in MCMC posterior samples of cell-type mean expression, overdispersion, and dropout parameters from full BUSseq and BUSseq-nzf BUSseq_MCMC fits
- Batch and cell-type metadata (dimension file listing N, G, B, and per-batch cell numbers, as in dim_*.txt)

## Outputs

- table1_zero_rates.csv with per-batch observed zero rates versus posterior-mean zero rates under full BUSseq and BUSseq-nzf (e.g., hematopoietic MARS-seq: 0.892 observed vs 0.887 BUSseq vs 0.874 BUSseq-nzf)
- 4000 posterior predictive replicated count datasets per fitted model per batch

## How to apply

Fit the full BUSseq model to the raw read count matrices (mouse hematopoietic MARS-seq and human pancreas studies) using BUSseq_MCMC with Drop_ind enabled to activate the zero-inflated dropout component, running n.iterations=4000 with burn-in and n.cores for parallel chains, then refit both studies with the reduced BUSseq-nzf model in which the dropout coefficient is fixed at zero. From each fit, extract post-burn-in posterior samples of the cell-type mean expression, overdispersion, and dropout parameters. Simulate 4000 replicated count datasets per batch from the posterior predictive distribution of each model, compute the per-batch zero rate of the raw data, and average zero rates across the 4000 simulations to get posterior-mean zero rates. Assemble the Table 1 comparison of observed versus model-implied zero rates per batch. The rationale is that dropout events inflate zeros beyond negative-binomial expectations, so a model ignoring dropout (BUSseq-nzf) should visibly underestimate zero rates under non-UMI protocols.

## Related tools

- **BUSseq** (Fit the full zero-inflated Bayesian hierarchical model via BUSseq_MCMC (Drop_ind = TRUE) and the reduced BUSseq-nzf variant (dropout coefficient fixed at zero); its posterior samples supply the parameters for simulating the 4000 posterior predictive replicates) — https://github.com/songfd2018/BUSseq-Rpackage
- **BUSseq-1.1_implementation** (Reproduction code repository providing access to the mouse hematopoietic and human pancreas raw count data used for the Table 1 zero-rate comparison) — https://github.com/songfd2018/BUSseq-1.1_implementation

## Examples

```
BUSseq_MCMC(ObservedData, n.celltypes=K, seed=123, n.cores=4, n.iterations=4000, n.burnin=1000, working_dir="./", Drop_ind=TRUE)
```

## Evaluation signals

- Posterior-mean zero rates from the full BUSseq model closely match observed zero rates in every batch (e.g., 0.887 vs 0.892 for the hematopoietic MARS-seq batch)
- BUSseq-nzf visibly underestimates observed zero rates under the non-UMI protocol (0.874 vs 0.892 for hematopoietic MARS-seq), and its largest discrepancies occur in batches assayed by non-UMI protocols such as SMART-seq2
- Each posterior predictive comparison is based on 4000 replicated count datasets simulated per batch from post-burn-in samples
- The output table1_zero_rates.csv contains one observed value and two model values (full BUSseq and BUSseq-nzf) per batch for both studies
- Both model fits used n.iterations=4000 with burn-in, with the full fit run with Drop_ind enabled and the nzf fit with the dropout coefficient fixed at zero

## Limitations

- MCMC algorithms are computationally heavy, so fitting full and reduced models plus 4000 posterior predictive simulations is expensive, although most MCMC steps of BUSseq are parallelizable
- The check is specific to BUSseq-style zero-inflated negative-binomial models and its dropout parameterization; results do not transfer directly to other count models
- The largest BUSseq-nzf misfit is expected on non-UMI batches; on UMI-protocol batches the two models may be nearly indistinguishable, limiting the check's discriminative power there

## Evidence

- [methods] BUSseq_MCMC signature with Drop_ind and iteration parameters: "`BUSseq_MCMC(ObservedData, n.celltypes, seed, n.cores, n.iterations, n.burnin, n.unchanged, n.output, working_dir, Drop_ind, fdr, hyper_pi, hyper_gamma0, hyper_gamma1, hyper_alpha, tau1sq, hyper_p,"
- [readme] BUSseq handles dropout events in its model: "BUSseq is able to simultaneously correct batch effects, clusters cell types, and takes care of the count data nature, the overdispersion, the dropout events, and the cell-specific sequencing depth of"
- [abstract] BUSseq imputes missing data caused by dropout events: [evidence span withheld — anchor status: not_found]
- [discussion] MCMC parallelizability supports the heavy posterior simulation workload: "most steps of the MCMC algorithm for BUSseq are parallelizable (Supplementary Note 12)."
- [readme] Data availability for the reproduction studies: [evidence span withheld — anchor status: external_doc]
