---
name: simulation-scenario-parameterization
description: Use when validating that a TWAS or gene-based test controls type I error
  before applying it to real GWAS data, or when designing simulation grids over p_causal
  and h_e2 for null or power calibrations.
when_to_use_negative:
- The goal is power comparison under alternative (causal) architectures rather than
  null calibration — effects must then be injected instead of simulating pure noise
- The reference LD panel does not match the ancestry/population of the data being
  tested, violating the calibration premise
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_0203
tools:
- name: OTTERS
  role: Runs TWAS Stage II testing.py with --models=P0.001,P0.05,lassosum,SDPR,PRScs
    on simulated null GWAS to produce null gene-based and ACAT-O omnibus p values
  repo: https://github.com/daiqile96/OTTERS
- name: PLINK
  role: Supplies and extracts the LD reference panel genotypes (binary bed/bim/fam)
    from which null phenotypes are simulated
- name: numpy
  role: Simulates null quantitative phenotypes from N(0,1) under fixed p_causal and
    h_e2
- name: scipy
  role: Statistical support for simulation and p-value handling
- name: PRS-CS
  role: One of the five eQTL weight-training models included in the null-replicate
    testing sweep
  repo: https://github.com/getian107/PRScs
- name: lassosum
  role: One of the five eQTL weight-training models included in the null-replicate
    testing sweep
  repo: https://github.com/tshmak/lassosum
- name: SDPR
  role: One of the five eQTL weight-training models included in the null-replicate
    testing sweep
  repo: https://github.com/eldronzhou/SDPR
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-36862-w
    title: 'OTTERS: a powerful TWAS framework leveraging summary-level reference data'
  source_content_hash: sha256:230b196219566c1073ada4def1c054b7197ea3581865938269bd256f34dc5f7e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/daiqile96/OTTERS
  tools:
  - OTTERS
  - PLINK
  - PRS-CS
  - Python
  - SDPR
  - lassosum
  - numpy
  - scipy
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# simulation-scenario-parameterization

## Summary

Define null-simulation scenarios (causal-variant proportion, heritability, number of phenotypes/replicates) and derive empirical type I error rates at fixed p-value thresholds to assess whether a gene-based association test such as the OTTERS ACAT-O omnibus TWAS is well calibrated.

## When to use

Use when validating that a TWAS or gene-based test controls type I error before applying it to real GWAS data, or when designing simulation grids over p_causal and h_e2 for null or power calibrations.

## When NOT to use

- The goal is power comparison under alternative (causal) architectures rather than null calibration — effects must then be injected instead of simulating pure noise
- The reference LD panel does not match the ancestry/population of the data being tested, violating the calibration premise

## Inputs

- LD reference panel genotypes in PLINK binary format (bed/bim/fam)
- Annotation file defining gene boundaries, used with OTTERS
- Simulation parameters: number of phenotypes (2000), p_causal (0.001), h_e2 (0.1), target replicate count (10^6 null TWAS replicates)
- Stage I-trained eQTL weights from P+T, lassosum, SDPR, PRS-CS

## Outputs

- type1_error_table.csv with empirical type I error rates per model at thresholds 10^-2, 10^-4, 2.5x10^-6
- qq_plot.png of observed vs expected null TWAS p values
- Null gene-based association p values (per-model and ACAT-O omnibus) per replicate

## How to apply

Fix a simulation grid over genetic-architecture parameters — in the OTTERS calibration, causal-variant proportion p_causal=0.001 and heritability h_e2=0.1 with 2000 quantitative phenotypes drawn from N(0,1), totaling 10^6 null TWAS replicates. Generate each null phenotype so no SNP has a true effect, using the LD reference panel genotypes (PLINK bed/bim/fam) so LD structure is preserved. Run the full pipeline under test — here OTTERS testing.py with --models=P0.001,P0.05,lassosum,SDPR,PRScs on Stage I-trained eQTL weights — on every null GWAS to obtain null gene-level and ACAT-O omnibus p values. Aggregate null p values and compute empirical type I error at prespecified thresholds (10^-2, 10^-4, 2.5x10^-6); well-calibrated tests should match the nominal rate within Monte Carlo error, especially in the tail. Visualize calibration with a Q-Q plot of observed vs expected -log10(p), which should track the identity line as in Supplementary Fig. S1.

## Related tools

- **OTTERS** (Runs TWAS Stage II testing.py with --models=P0.001,P0.05,lassosum,SDPR,PRScs on simulated null GWAS to produce null gene-based and ACAT-O omnibus p values) — https://github.com/daiqile96/OTTERS
- **PLINK** (Supplies and extracts the LD reference panel genotypes (binary bed/bim/fam) from which null phenotypes are simulated)
- **numpy** (Simulates null quantitative phenotypes from N(0,1) under fixed p_causal and h_e2)
- **scipy** (Statistical support for simulation and p-value handling)
- **PRS-CS** (One of the five eQTL weight-training models included in the null-replicate testing sweep) — https://github.com/getian107/PRScs
- **lassosum** (One of the five eQTL weight-training models included in the null-replicate testing sweep) — https://github.com/tshmak/lassosum
- **SDPR** (One of the five eQTL weight-training models included in the null-replicate testing sweep) — https://github.com/eldronzhou/SDPR

## Examples

```
python testing.py --gwas_null_pheno1.txt --weights eqtl_weights --models=P0.001,P0.05,lassosum,SDPR,PRScs --out null_twas_pvalues.txt
```

## Evaluation signals

- Empirical type I error rates are well controlled at thresholds 10^-4 to 2.5x10^-6, matching Supplementary Table S1
- Only slight inflation is tolerated at the loosest threshold (10^-2), consistent with the paper's reported behavior
- Q-Q plot of null p values tracks the identity line across the distribution, reproducing Supplementary Fig. S1
- The total number of null replicates reaches ~10^6 TWAS tests (2000 phenotypes per scenario), sufficient for tail-threshold resolution at 2.5x10^-6
- All five training models (P0.001, P0.05, lassosum, SDPR, PRS-CS) and the ACAT-O omnibus p value appear in the type I error table

## Limitations

- Empirical rates at 2.5x10^-6 have high Monte Carlo error unless the number of null replicates is very large; fewer replicates make tail estimates unstable
- Calibration shown is for quantitative phenotypes simulated under a single architecture point (p_causal=0.001, h_e2=0.1); other architectures require re-parameterization
- OTTERS is limited to cis-eQTL effects and mainly European-descent eQTL summary data, so calibration conclusions do not automatically transfer to trans-effects or other ancestries

## Evidence

- [full_text] Null simulations with p_causal=0.001 and heritability h_e2=0.1, generating 10^6 null TWAS replicates in total from 2000 quantitative phenotypes: [evidence span withheld — anchor status: not_found]
- [full_text] OTTERS testing.py run with all five models to obtain null ACAT-O omnibus p values: [evidence span withheld — anchor status: not_found]
- [full_text] Empirical type I error computed at thresholds 10^-2 to 2.5x10^-6 and compared against Supplementary Table S1: [evidence span withheld — anchor status: not_found]
- [abstract] The paper claims calibration and power support for OTTERS from simulations: "We show that OTTERS is a practical and powerful TWAS tool by both simulations and application studies."
