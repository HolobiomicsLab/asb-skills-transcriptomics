---
name: twas-power-estimation-across-parameter-grid
description: Use when you need to compare TWAS power across prediction models or quantify
  how power scales with GWAS/eQTL sample size, eqtl-h2, h2ge, and number of causal
  eQTLs before choosing weights for a real TWAS.
when_to_use_negative:
- You need fine-mapped credible sets of causal genes from existing TWAS statistics
  — use FOCUS/MA-FOCUS instead of simulation-based power analysis.
- You have real GWAS and eQTL weights and only want to compute a single TWAS statistic
  on observed data rather than estimate power under simulated causal models.
- h2ge = 0 null behavior is the only interest and you cannot afford replicate-level
  simulation; empirical power estimation requires many replicates per setting.
edam_operation: http://edamontology.org/operation_3566
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3517
tools:
- name: twas_sim
  role: Simulation framework that samples genomic regions, simulates GWAS summary
    statistics, computes TWAS Z-scores from LD, GWAS Z-scores, and estimated eQTL
    effect sizes
  repo: https://github.com/mancusolab/twas_sim
- name: sim.py
  role: Core simulator invoked per parameter set and replicate with --linear-model,
    --h2ge, --ncausal, --seed, and --IDX flags
  repo: https://github.com/mancusolab/twas_sim
- name: PLINK
  role: Provides PLINK-formatted genotype input; --eqtl-prefix and --test-prefix point
    to PLINK files for eQTL and test LD panels
- name: susieR
  role: Fits the SuSiE prediction model via the external module (external_r.py calling
    external.R) to compare its power against LASSO/Enet/GBLUP
  repo: https://github.com/stephenslab/susieR
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1093/bioinformatics/btad288
    title: twas_sim, a Python-based tool for simulation and power analysis of transcriptome-wide
      association analysis
  source_content_hash: sha256:a8452dd515a494a9981de5d8ae2f9b77169b2bb81faa3b226332f5b372d8dc18
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mancusolab/twas_sim
  tools:
  - PLINK
  - R
  - sim.py
  - susieR
  - twas_sim
  edam_operation: http://edamontology.org/operation_3566
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4444444444444444
n_claims: 9
---

# twas-power-estimation-across-parameter-grid

## Summary

A simulation-based TWAS power analysis workflow using twas_sim's sim.py to estimate how statistical power varies across expression prediction models (LASSO, Elastic Net, GBLUP, SuSiE) and simulation parameters (h2ge, GWAS/eQTL sample sizes, eQTL sparsity) under a causal eQTL-trait model.

## When to use

Use when you need to compare TWAS power across prediction models or quantify how power scales with GWAS/eQTL sample size, eqtl-h2, h2ge, and number of causal eQTLs before choosing weights for a real TWAS.

## When NOT to use

- You need fine-mapped credible sets of causal genes from existing TWAS statistics — use FOCUS/MA-FOCUS instead of simulation-based power analysis.
- You have real GWAS and eQTL weights and only want to compute a single TWAS statistic on observed data rather than estimate power under simulated causal models.
- h2ge = 0 null behavior is the only interest and you cannot afford replicate-level simulation; empirical power estimation requires many replicates per setting.

## Inputs

- 1000G Phase 3 PLINK-formatted reference genotypes (restricted to HapMap3 variants)
- Parameter grid specification (slurm.params-style: GWAS N, eQTL N, eQTL model, eqtl-h2, h2ge 0 to 0.01, linear model, ncausal)
- Per-replicate seeds / --IDX simulation indices
- Optional external prediction module (external_r for SuSiE via susieR)

## Outputs

- Per-replicate OUTPUT.summary.tsv files with twas.p, twas.z, and alpha columns
- power_summary.tsv tabulating power per prediction model as a function of h2ge, sample sizes, and eQTL sparsity
- Power curves comparable to Supplementary Figs S4-S5

## How to apply

Download the 1000G Phase 3 PLINK reference genotypes and define a parameter grid (GWAS N, eQTL N, eQTL model, eqtl-h2, h2ge from 0 to 0.01, linear model, ncausal sparsity) in a slurm.params-style file modeled on example.slurm.sh. For each parameter set, run sim.py with --indep-gwas (and --fast-gwas-sim) to generate GWAS effect sizes under the horizontal-pleiotropy-through-linkage model, using --eqtl-prefix and --test-prefix for separate eQTL/TWAS LD panels and the --ld-ridge 0.1 diagonal offset. Fit eQTL prediction weights by cycling --linear-model through lasso, enet, ridge (GBLUP), and external with --external-module external_r (which calls external.R to fit SuSiE via susieR). Repeat each setting across many replicates using distinct --seed/--IDX values, then parse the per-replicate OUTPUT.summary.tsv files (columns twas.p, twas.z, alpha) and compute power per setting as the fraction of replicates with significant TWAS statistics. Tabulate power for each model as a function of h2ge, GWAS/eQTL sample size, and ncausal, and compare the resulting power curves against the published Supplementary Figs S4-S5.

## Related tools

- **twas_sim** (Simulation framework that samples genomic regions, simulates GWAS summary statistics, computes TWAS Z-scores from LD, GWAS Z-scores, and estimated eQTL effect sizes) — https://github.com/mancusolab/twas_sim
- **sim.py** (Core simulator invoked per parameter set and replicate with --linear-model, --h2ge, --ncausal, --seed, and --IDX flags) — https://github.com/mancusolab/twas_sim
- **PLINK** (Provides PLINK-formatted genotype input; --eqtl-prefix and --test-prefix point to PLINK files for eQTL and test LD panels)
- **susieR** (Fits the SuSiE prediction model via the external module (external_r.py calling external.R) to compare its power against LASSO/Enet/GBLUP) — https://github.com/stephenslab/susieR

## Examples

```
python sim.py prefix --indep-gwas --fast-gwas-sim --ngwas 100000 --nqtl 500 --eqtl-h2 0.1 --h2ge 0.001 --ncausal 1 --linear-model enet --ld-ridge 0.1 --seed 1 --IDX 1 -o sim_enet
```

## Evaluation signals

- Power increases monotonically with h2ge, GWAS/eQTL sample size, and eQTL sparsity, consistent with Supplementary Figs S4-S5.
- Null setting (h2ge = 0) yields empirical power near the nominal significance level.
- Replicates with distinct --seed/--IDX values produce per-setting power estimates with plausible sampling variability; each replicate yields a valid OUTPUT.summary.tsv with twas.p, twas.z, alpha columns.
- Relative model ranking reproduces the reported ordering: Elastic Net (0.66) > GBLUP (0.64) > LASSO (0.62) > SuSiE (0.44) under causal (alpha != 0) simulations.
- Setting --h2ge 0 versus nonzero values discriminates null from causal behavior as expected under the horizontal-pleiotropy-through-linkage simulation model.

## Limitations

- SuSiE external weights showed markedly lower power (0.44) than LASSO/Enet/GBLUP in this simulation framework, so absolute power levels may depend on the external module implementation.
- Power estimates are only valid within the simulator's assumptions (1000G/HapMap3-restricted regions, LD ridge offset 0.1, specified sparsity models); extrapolating to other genotype panels or architectures is not validated.
- The default h2ge range 0 (null) to 0.01 ('huge effect') bounds the explored effect-size regime; effects outside this range are untested.

## Evidence

- [abstract] twas_sim enables simplified performance evaluation and power analysis for TWAS methods: "we present twas_sim, a computationally scalable and easily extendable tool for simplified performance evaluation and power analysis for TWAS methods"
- [readme] sim.py generates a single TWAS statistic: "The script [example.sh](https://github.com/mancusolab/twas_sim/blob/master/example.sh) generates a single TWAS statistic using the simulator [sim.py]"
- [readme] Linear models supported for expression prediction: "**Linear model:** `twas_sim` supports predicting gene expressions using Elastic Net, LASSO, GBLUP, and true eQTL effect sizes. The dynamic import feature enables twas_sim to easily include external"
- [readme] TWAS statistic computation: "**TWAS:** twas_sim computes TWAS test statistics using LD, GWAS Z-scores, and estimated eQTL effect sizes."
- [readme] Reference genotype recommendation: "We recommend to use data from [1000G](https://data.broadinstitute.org/alkesgroup/LDSCORE/1000G_Phase3_plinkfiles.tgz) as reference genotypes"
- [readme] Variant filtering: "removing genetic variants that are non-biallelic, MAF < 1%, HWE < 1e-5, and variant missingness > 10%"
- [readme] Reasonable h2ge range: "H2GE=0.001 # variance explained in complex trait; 0 (null) to 0.01 (huge effect) are reasonable values"
- [readme] Horizontal pleiotropy modeling: "**Horizontal pleiotropy:** twas_sim accounts for the situation when nearby tagging genes are also tested in TWAS in addition to the causal TWAS model."
