---
name: horizontal-pleiotropy-gwas-effect-simulation
description: Use when assessing whether TWAS test statistics at tagging genes are
  inflated by horizontal pleiotropy through linkage, or when building a null ensemble
  of TWAS Z-scores/p-values under a model where GWAS effects are independent of eQTLs.
when_to_use_negative:
- The goal is power analysis under the causal TWAS model where expression mediates
  the trait (alpha != 0) — use the causal simulation mode instead of --indep-gwas,
  since --indep-gwas generates GWAS effects independently of eQTLs by design
- Input is real GWAS summary statistics rather than simulated genotypes/traits — twas_sim
  simulates from PLINK reference genotypes and cannot ingest existing GWAS results
- Evaluating prediction-weight accuracy of eQTL models (LASSO, ENet, GBLUP, SuSiE)
  rather than null-model FWER calibration — the pleiotropy mode conflates tagging-gene
  tests and is not a clean comparison of weight quality
edam_operation: http://edamontology.org/operation_3566
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3517
tools:
- name: twas_sim
  role: Python simulator providing sim.py with the --indep-gwas flag that generates
    GWAS effect sizes independently of eQTLs to model horizontal pleiotropy through
    linkage
  repo: https://github.com/mancusolab/twas_sim
- name: PLINK
  role: Reference genotype format for LD panels; optional --eqtl-prefix and --test-prefix
    supply distinct eQTL and TWAS LD references
- name: conda
  role: Environment management for twas_sim installation (conda env create --file
    environment.yml; conda activate twas_sim)
provenance:
  source_task_ids:
  - task_002
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad288
    title: twas_sim, a Python-based tool for simulation and power analysis of transcriptome-wide
      association analysis
  source_content_hash: sha256:9bc471786d0729741eaedae390d6237eed59704db073a63b192f9f500db8473c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/mancusolab/twas_sim
  tools:
  - PLINK
  - R
  - conda
  - sim.py
  - susieR
  - twas_sim
  edam_operation: http://edamontology.org/operation_3566
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 10
---

# horizontal-pleiotropy-gwas-effect-simulation

## Summary

Simulate GWAS effect sizes independently of eQTL effects using twas_sim's --indep-gwas mode, which models horizontal pleiotropy through linkage and is used to quantify FWER inflation at tagging genes in TWAS.

## When to use

Use when assessing whether TWAS test statistics at tagging genes are inflated by horizontal pleiotropy through linkage, or when building a null ensemble of TWAS Z-scores/p-values under a model where GWAS effects are independent of eQTLs.

## When NOT to use

- The goal is power analysis under the causal TWAS model where expression mediates the trait (alpha != 0) — use the causal simulation mode instead of --indep-gwas, since --indep-gwas generates GWAS effects independently of eQTLs by design
- Input is real GWAS summary statistics rather than simulated genotypes/traits — twas_sim simulates from PLINK reference genotypes and cannot ingest existing GWAS results
- Evaluating prediction-weight accuracy of eQTL models (LASSO, ENet, GBLUP, SuSiE) rather than null-model FWER calibration — the pleiotropy mode conflates tagging-gene tests and is not a clean comparison of weight quality

## Inputs

- 1000G Phase 3 PLINK-formatted reference genotypes (optionally separate --eqtl-prefix and --test-prefix PLINK panels for eQTL and TWAS LD)
- Simulation parameters: GWAS sample size (--ngwas), eQTL sample size (--nqtl), eQTL heritability (--eqtl-h2), phenotypic variance explained (--h2ge), GWAS SNP heritability (--h2g-gwas), number of causal SNPs (--ncausal), LD ridge offset (--ld-ridge)
- Per-replicate seed and simulation index (--seed, --IDX)

## Outputs

- Per-replicate OUTPUT.summary.tsv files containing twas.z, twas.p, and alpha columns at tagging genes
- Aggregated null ensemble of TWAS Z-scores/p-values and an estimated family-wise error rate under the pleiotropy-through-linkage model

## How to apply

Set up the twas_sim conda environment and download the 1000 Genomes Phase 3 PLINK reference genotypes, restricting to biallelic HapMap3 variants with MAF >= 1%, HWE >= 1e-5, and missingness <= 10%. Invoke sim.py with --indep-gwas so GWAS effect sizes are generated independently of eQTL effects (the horizontal-pleiotropy-through-linkage model), typically with --fast-gwas-sim, --h2g-gwas 0.01, --ngwas 100000, --nqtl 500, --eqtl-h2 0.1, --h2ge 0.001, --linear-model enet, --ncausal 1, and --ld-ridge 0.1 for the diagonal LD offset. Optionally pass --eqtl-prefix and --test-prefix so GWAS, eQTL estimation, and TWAS testing use distinct LD reference panels. Repeat across many genomic regions (sampled uniformly at random) and many --seed/--IDX replicates, then aggregate twas.z and twas.p from each OUTPUT.summary.tsv and estimate the family-wise error rate as the fraction of replicates with a significant TWAS result, targeting the reported elevated FWER of P = 0.02 (Supplementary Fig S8).

## Related tools

- **twas_sim** (Python simulator providing sim.py with the --indep-gwas flag that generates GWAS effect sizes independently of eQTLs to model horizontal pleiotropy through linkage) — https://github.com/mancusolab/twas_sim
- **PLINK** (Reference genotype format for LD panels; optional --eqtl-prefix and --test-prefix supply distinct eQTL and TWAS LD references)
- **conda** (Environment management for twas_sim installation (conda env create --file environment.yml; conda activate twas_sim))

## Examples

```
python sim.py <prefix> --indep-gwas --fast-gwas-sim --h2g-gwas 0.01 --ngwas 100000 --nqtl 500 --eqtl-h2 0.1 --h2ge 0.001 --linear-model enet --ncausal 1 --ld-ridge 0.1 --seed <IDX> --output <prefix>
```

## Evaluation signals

- Aggregated family-wise error rate from --indep-gwas runs is elevated above the nominal significance level and consistent with the reported P = 0.02 (Supplementary Fig S8)
- Each replicate's OUTPUT.summary.tsv contains populated twas.z and twas.p fields at tagging genes
- Genotype inputs pass the standard filters: biallelic HapMap3 variants with MAF >= 1%, HWE >= 1e-5, and missingness <= 10%
- Null setting h2ge = 0 yields empirical type-I error near the nominal level in causal-model control runs, contrasting with the inflated FWER under --indep-gwas

## Limitations

- Inflated FWER arises specifically from linkage between eQTLs of the causal gene and tagging genes; it is a property of the pleiotropy-through-linkage model, not evidence of estimator failure
- Simulation depends on the quality of the LD reference (1000G Phase 3 PLINK files) and the --ld-ridge 0.1 diagonal offset used to regularize the LD matrix
- Accurate FWER estimation requires many replicates across distinct genomic regions and seeds; few replicates give noisy error-rate estimates

## Evidence

- [results] When GWAS effect sizes are simulated independently of eQTLs, TWAS testing at tagging genes yields significantly inflated test statistics: [evidence span withheld — anchor status: not_found]
- [readme] Generate GWAS effect-sizes independently from eQTLs: "--indep-gwas          Generate GWAS effect-sizes independently from eQTLs."
- [readme] Horizontal pleiotropy modeling in twas_sim: "**Horizontal pleiotropy:** twas_sim accounts for the situation when nearby tagging genes are also tested in TWAS in addition to the causal TWAS model."
- [methods] Variant quality filtering for reference genotypes: "removing genetic variants that are non-biallelic, MAF < 1%, HWE < 1e-5, and variant missingness > 10%"
- [methods] TWAS statistics computation inputs: "**TWAS:** twas_sim computes TWAS test statistics using LD, GWAS Z-scores, and estimated eQTL effect sizes."
