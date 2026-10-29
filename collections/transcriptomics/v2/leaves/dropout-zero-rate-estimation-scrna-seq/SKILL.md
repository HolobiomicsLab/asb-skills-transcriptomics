---
name: dropout-zero-rate-estimation-scrna-seq
description: Use when validating that a Bayesian scRNA-seq model's dropout (zero-inflation)
  component captures observed zero-count rates, or when quantifying how much model
  misfit a negative-binomial-only model incurs under non-UMI protocols.
when_to_use_negative:
- The input is UMI count data where zero rates are driven by sequencing depth rather
  than protocol-level dropout, so the non-UMI misfit signal (SMART-seq2-type batches)
  will not be diagnostic
- The goal is batch effects correction or cell-type clustering itself rather than
  model-fit validation of the zero-inflation component
- A completely confounded experimental design applies (batch 1 measures cell types
  1 and 2, batch 2 measures cell types 3 and 4), for which no method including BUSseq
  is applicable
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_2269
tools:
- name: BUSseq
  role: Fits the zero-inflated Bayesian hierarchical model (full and nzf variants)
    via BUSseq_MCMC with Drop_ind controlling the dropout component
  repo: https://github.com/songfd2018/BUSseq-Rpackage
- name: BUSseq-1.1_implementation
  role: Reproduction code and OneDrive-hosted raw count data for the mouse hematopoietic
    and human pancreas studies used in Table 1
  repo: https://github.com/songfd2018/BUSseq-1.1_implementation
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-020-16905-2
    title: Flexible experimental designs for valid single-cell RNA-sequencing experiments
      allowing batch effects correction
  source_content_hash: sha256:027573ec7e5dffc81701661d32e46d5cff1a8e46610d22549758b19d2155d0b5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/songfd2018/BUSseq-Rpackage
  tools:
  - BUSseq
  - BUSseq-1.1_implementation
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# dropout-zero-rate-estimation-scrna-seq

## Summary

Reproduce a posterior predictive check of per-batch zero rates by fitting the full zero-inflated BUSseq model and the reduced BUSseq-nzf model (dropout coefficient fixed at zero) to real scRNA-seq read counts, then comparing posterior-mean zero rates from simulated replicates against the observed zero rates (Table 1).

## When to use

Use when validating that a Bayesian scRNA-seq model's dropout (zero-inflation) component captures observed zero-count rates, or when quantifying how much model misfit a negative-binomial-only model incurs under non-UMI protocols.

## When NOT to use

- The input is UMI count data where zero rates are driven by sequencing depth rather than protocol-level dropout, so the non-UMI misfit signal (SMART-seq2-type batches) will not be diagnostic
- The goal is batch effects correction or cell-type clustering itself rather than model-fit validation of the zero-inflation component
- A completely confounded experimental design applies (batch 1 measures cell types 1 and 2, batch 2 measures cell types 3 and 4), for which no method including BUSseq is applicable

## Inputs

- Raw read count matrices (genes x cells) of the mouse hematopoietic study (MARS-seq) from RawCountData in MouseHematopoietic
- Raw read count matrices of the human pancreas study from RawCountData in HumanPancreas
- Dimension files (cells, genes, batches, per-batch cell counts) and metadata files (batch and cell type labels)

## Outputs

- table1_zero_rates.csv with per-batch observed zero rates vs posterior-mean zero rates under full BUSseq and BUSseq-nzf
- Posterior samples of cell-type mean expression, overdispersion, and dropout parameters from each BUSseq_MCMC fit
- 4000 simulated posterior predictive replicated count datasets per model per batch

## How to apply

Load the hematopoietic (MARS-seq) and human pancreas read count matrices and fit the full BUSseq model with BUSseq_MCMC, enabling the zero-inflated dropout component via Drop_ind, with n.iterations=4000 including burn-in and n.cores for parallel chains. Refit both studies with the reduced BUSseq-nzf model in which the dropout coefficient is fixed at zero. Extract post-burn-in posterior samples of the cell-type mean expression, overdispersion, and dropout parameters from each fit, then simulate 4000 replicated count datasets per batch from each model's posterior predictive distribution. Compute per-batch zero rates of the raw data and posterior-mean zero rates across simulations for each model, and assemble the Table 1 comparison (e.g., hematopoietic MARS-seq: 0.892 observed vs 0.887 BUSseq vs 0.874 BUSseq-nzf) written to table1_zero_rates.csv.

## Related tools

- **BUSseq** (Fits the zero-inflated Bayesian hierarchical model (full and nzf variants) via BUSseq_MCMC with Drop_ind controlling the dropout component) — https://github.com/songfd2018/BUSseq-Rpackage
- **BUSseq-1.1_implementation** (Reproduction code and OneDrive-hosted raw count data for the mouse hematopoietic and human pancreas studies used in Table 1) — https://github.com/songfd2018/BUSseq-1.1_implementation

## Examples

```
Rscript -e 'BUSseq_MCMC(ObservedData, n.celltypes=K, seed=123, n.cores=4, n.iterations=4000, n.burnin=2000, Drop_ind=TRUE)'
```

## Evaluation signals

- Posterior-mean zero rates from full BUSseq must closely match observed zero rates in every batch (e.g., 0.887 vs 0.892 for hematopoietic MARS-seq)
- BUSseq-nzf must visibly underestimate zero rates under the non-UMI protocol (0.874 vs 0.892 for hematopoietic MARS-seq), demonstrating the dropout-model misfit
- Both models must be run with n.iterations=4000 with burn-in, and simulations must use 4000 posterior predictive replicates per batch
- Output CSV must contain, per batch, three aligned zero-rate values: observed, full BUSseq, and BUSseq-nzf
- MCMC chains should be run in parallel with a fixed seed so results are reproducible

## Limitations

- The largest BUSseq-nzf discrepancies occur on batches assayed by non-UMI protocols such as SMART-seq2, so the diagnostic contrast is weak or absent on UMI-based batches
- MCMC algorithms carry a heavy computational load, though most steps of the BUSseq MCMC are parallelizable (Supplementary Note 12)
- The C++ source codes support Linux operating systems (tested on Ubuntu 18.04 and CentOS 7), constraining where the standalone implementation can run

## Evidence

- [methods] MCMC inference: "`BUSseq_MCMC(ObservedData, n.celltypes, seed, n.cores, n.iterations, n.burnin, n.unchanged, n.output, working_dir, Drop_ind, fdr, hyper_pi, hyper_gamma0, hyper_gamma1, hyper_alpha, tau1sq, hyper_p,`"
- [intro] zero-inflated hierarchical model: "we propose Batch effects correction with Unknown Subtypes for scRNA-seq data (BUSseq), an interpretable hierarchical model that simultaneously corrects batch effects, clusters cell types, and takes"
- [readme] reproduction data: [evidence span withheld — anchor status: external_doc]
