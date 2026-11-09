---
name: twas-null-calibration-analysis
description: Use when you need to verify that simulated TWAS Z-scores or p-values
  are consistent with the null (KS p > 0.05 vs Uniform(0,1)), with FWER ~0.05 and
  genomic inflation lambda ~1.
when_to_use_negative:
- Estimating TWAS power under alternative models (h2ge > 0) — use power-analysis rather
  than null calibration, where p-value uniformity is expected to break by design.
- Assessing calibration under LD misspecification (mismatched GWAS/eQTL/TWAS panels
  via --eqtl-prefix/--test-prefix) — that requires the matched-LD null as the correctly
  specified baseline comparison instead.
- Calibrating statistics from real observed GWAS data rather than twas_sim-simulated
  summary statistics, since the null expectation here derives from simulated loci
  with alpha = 0.
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_3053
tools:
- name: twas_sim (sim.py)
  role: Simulator that generates null TWAS test statistics from real genotype data;
    run with --h2ge 0 and --fast-gwas-sim per seeded replicate
  repo: https://github.com/mancusolab/twas_sim
- name: PLINK
  role: Provides the PLINK-formatted reference genotype files used as the LD input
    for simulation
- name: conda
  role: Installs the twas_sim execution environment from environment.yml
- name: Python
  role: Performs KS tests against Uniform(0,1), FWER estimation, and genomic inflation
    factor computation on collected twas.p / twas.z values
provenance:
  source_task_ids:
  - task_003
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad288
    title: twas_sim, a Python-based tool for simulation and power analysis of transcriptome-wide
      association analysis
  source_content_hash: sha256:fe00a8ae7503a0510c951bb288bbf76391be16c1295db7247ea0f88563cbda4e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mancusolab/twas_sim
  tools:
  - PLINK
  - Python
  - conda
  - sim.py
  - twas_sim
  - twas_sim (sim.py)
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_3053
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4666666666666667
n_claims: 15
---

# twas-null-calibration-analysis

## Summary

Evaluates whether TWAS test statistics produced by the twas_sim simulator are unbiased and well-calibrated under null simulations (alpha = 0), using Kolmogorov-Smirnov tests on TWAS Z-scores/p-values, family-wise error rate (FWER), and genomic inflation factors across linear prediction models.

## When to use

Use when you need to verify that simulated TWAS Z-scores or p-values are consistent with the null (KS p > 0.05 vs Uniform(0,1)), with FWER ~0.05 and genomic inflation lambda ~1.

## When NOT to use

- Estimating TWAS power under alternative models (h2ge > 0) — use power-analysis rather than null calibration, where p-value uniformity is expected to break by design.
- Assessing calibration under LD misspecification (mismatched GWAS/eQTL/TWAS panels via --eqtl-prefix/--test-prefix) — that requires the matched-LD null as the correctly specified baseline comparison instead.
- Calibrating statistics from real observed GWAS data rather than twas_sim-simulated summary statistics, since the null expectation here derives from simulated loci with alpha = 0.

## Inputs

- PLINK-formatted 1000 Genomes Phase 3 reference genotypes (recommended: 1000G Phase 3 PLINK files from data.broadinstitute.org/alkesgroup/LDSCORE/1000G_Phase3_plinkfiles.tgz)
- twas_sim conda environment (environment.yml)
- sim.py simulator with parameters --h2ge 0 (null), --ngwas, --nqtl, --ncausal, --eqtl-h2, --linear-model, --seed, --output

## Outputs

- Per-replicate OUTPUT.summary.tsv files containing twas.z and twas.p
- twas_null_calibration.tsv with per-model KS statistic/p-value, FWER, and genomic inflation factor

## How to apply

Install the simulator from mancusolab/twas_sim via `conda env create --file environment.yml` and use PLINK-formatted 1000 Genomes Phase 3 reference genotypes as the LD panel. Run sim.py in batch across many loci/replicates under the null by setting `--h2ge 0`, using `--fast-gwas-sim` so GWAS summary statistics are drawn directly from the multivariate normal parameterized by LD, a fixed `--seed` per replicate, and `--linear-model enet`; repeat for lasso, ridge, and trueqtl to compare across prediction models. Collect twas.z and twas.p from each replicate's OUTPUT.summary.tsv report, then perform Kolmogorov-Smirnov tests of the null TWAS p-values against Uniform(0,1) (equivalently, TWAS Z-scores against the standard normal) for each linear model. Estimate the family-wise error rate as the fraction of replicates with TWAS p < 0.05 and the genomic inflation factor from mean TWAS/SNP chi-square statistics. Write the per-model KS statistic/p-value, FWER, and inflation factor into twas_null_calibration.tsv and compare against the paper's reported values (Supplementary Table S3, Figs S2-S3).

## Related tools

- **twas_sim (sim.py)** (Simulator that generates null TWAS test statistics from real genotype data; run with --h2ge 0 and --fast-gwas-sim per seeded replicate) — https://github.com/mancusolab/twas_sim
- **PLINK** (Provides the PLINK-formatted reference genotype files used as the LD input for simulation)
- **conda** (Installs the twas_sim execution environment from environment.yml)
- **Python** (Performs KS tests against Uniform(0,1), FWER estimation, and genomic inflation factor computation on collected twas.p / twas.z values)

## Examples

```
python sim.py <prefix> --h2ge 0 --fast-gwas-sim --ngwas 100000 --nqtl 500 --ncausal 1 --eqtl-h2 0.1 --linear-model enet --seed <IDX> --output <prefix>
```

## Evaluation signals

- KS test of TWAS p-values against Uniform(0,1) should not reject uniformity (KS p > 0.05) for each linear model; the paper reports KS P = 0.26 for Elastic Net
- Empirical FWER (fraction of replicates with TWAS p < 0.05) should be approximately 0.05
- Genomic inflation lambda estimated from mean TWAS/SNP chi-square statistics should be approximately 1
- Null TWAS Z-scores should be consistent with the standard normal distribution
- Per-model KS/lambda/FWER values should align with the direction and approximate magnitude of Supplementary Table S3 and Figs S2-S3

## Limitations

- Calibration is only meaningful for the supported linear prediction models (lasso, enet, ridge, trueqtl, external); results differ by model, and the paper notes 'similar patterns observed for other linear models' rather than identical values
- The null result assumes a correctly specified (matched) LD reference; introducing different LD panels across GWAS, eQTL, and TWAS testing requires a separate misspecification analysis
- Variant filtering (non-biallelic, MAF < 1%, HWE < 1e-5, missingness > 10%, HapMap3 restriction) is applied by the simulator and affects the LD structure underlying the null statistics

## Evidence

- [abstract] twas_sim enables simplified performance evaluation and power analysis for TWAS methods: "we present twas_sim, a computationally scalable and easily extendable tool for simplified performance evaluation and power analysis for TWAS methods"
- [methods] TWAS test statistics are computed from LD, GWAS Z-scores, and eQTL effect sizes: "twas_sim computes TWAS test statistics using LD, GWAS Z-scores, and estimated eQTL effect sizes."
- [readme] Fast GWAS simulation draws summary statistics from the LD-parameterized multivariate normal: "Fast GWAS simulates GWAS summary statistics directly using the multivariate normal"
- [methods] Reference genotype filtering applied before simulation: "removing genetic variants that are non-biallelic, MAF < 1%, HWE < 1e-5, and variant missingness > 10%"
- [readme] Recommended 1000G Phase 3 PLINK reference genotypes: "We recommend to use data from [1000G](https://data.broadinstitute.org/alkesgroup/LDSCORE/1000G_Phase3_plinkfiles.tgz) as reference genotypes"
