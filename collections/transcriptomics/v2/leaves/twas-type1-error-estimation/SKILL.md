---
name: twas-type1-error-estimation
description: Use when you need to verify that a gene-based association (TWAS) test
  controls false positives under the null, before applying it to real GWAS summary
  statistics.
when_to_use_negative:
- You want to evaluate statistical power or effect-size detection, not null calibration
  — use power simulations with causal variants instead
- Your gene-phenotype test is not a TWAS omnibus test; the ACAT-O aggregation and
  its null distribution will not apply
edam_operation: http://edamontology.org/operation_2945
edam_topics:
- http://edamontology.org/topic_3050
- http://edamontology.org/topic_0203
tools:
- name: OTTERS
  role: Runs the Stage I weight training (P+T, lassosum, SDPR, PRScs) and Stage II
    gene-based association testing on simulated null GWAS data to produce null TWAS
    and ACAT-O omnibus p values
  repo: https://github.com/daiqile96/OTTERS
- name: PRScs
  role: One of the five eQTL weight-training models whose null TWAS p values feed
    the omnibus calibration check
  repo: https://github.com/getian107/PRScs
- name: lassosum
  role: Frequentist LASSO eQTL weight-training model included among the five models
    tested under the null
  repo: https://github.com/tshmak/lassosum
- name: SDPR
  role: Bayesian DPR-model-based eQTL weight-training model included among the five
    models tested under the null
  repo: https://github.com/eldronzhou/SDPR
- name: PLINK
  role: Provides the binary bed/bim/fam LD reference genotype format loaded for simulation
    and extraction
- name: numpy/scipy
  role: Simulate the 2000 N(0,1) null phenotypes with p_causal=0.001 and h_e2=0.1
- name: pandas
  role: Aggregate null p values and compute per-threshold empirical type I error rates
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-36862-w
    title: 'OTTERS: a powerful TWAS framework leveraging summary-level reference data'
  source_content_hash: sha256:b5892543a9bd2f189781a13c81974d99c10262d15075b725c22fac9dd57bf541
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/daiqile96/OTTERS
  tools:
  - OTTERS
  - PLINK
  - PRScs
  - Python
  - SDPR
  - lassosum
  - numpy
  - numpy/scipy
  - pandas
  - scipy
  edam_operation: http://edamontology.org/operation_2945
  edam_topics:
  - http://edamontology.org/topic_3050
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7894736842105263
n_claims: 19
---

# twas-type1-error-estimation

## Summary

Empirically calibrates the type I error of an omnibus TWAS test (OTTERS/ACAT-O) by simulating null phenotypes, running the full Stage I→Stage II pipeline, and inspecting Q-Q plots and tail empirical false-positive rates.

## When to use

Use when you need to verify that a gene-based association (TWAS) test controls false positives under the null, before applying it to real GWAS summary statistics.

## When NOT to use

- You want to evaluate statistical power or effect-size detection, not null calibration — use power simulations with causal variants instead
- Your gene-phenotype test is not a TWAS omnibus test; the ACAT-O aggregation and its null distribution will not apply

## Inputs

- LD reference panel genotypes in PLINK binary bed/bim/fam format
- Annotation file consumed by OTTERS
- Stage I-trained eQTL weights from P+T, lassosum, SDPR, PRScs models
- Simulated null quantitative phenotypes drawn from N(0,1)

## Outputs

- Null gene-based TWAS p values, including ACAT-O omnibus p values
- Empirical type I error table (type1_error_table.csv) at thresholds 10^-2 to 2.5x10^-6
- Q-Q plot of observed vs expected null p values (qq_plot.png)

## How to apply

Load the LD reference panel genotype data as PLINK binary bed/bim/fam files together with the annotation file in OTTERS. Simulate 2000 quantitative null phenotypes from N(0,1) with numpy/scipy under causal-variant proportion p_causal=0.001 and heritability h_e2=0.1, yielding 10^6 null TWAS replicates. Run OTTERS testing.py with eQTL weights trained by all five Stage I models (--models=P0.001,P0.05,lassosum,SDPR,PRScs) on each null GWAS to obtain null gene-based p values including ACAT-O omnibus p values. Aggregate null p values with pandas and compute empirical type I error rates at thresholds such as 10^-2, 10^-4, and 2.5x10^-6; construct a Q-Q plot of observed vs expected p values.

## Related tools

- **OTTERS** (Runs the Stage I weight training (P+T, lassosum, SDPR, PRScs) and Stage II gene-based association testing on simulated null GWAS data to produce null TWAS and ACAT-O omnibus p values) — https://github.com/daiqile96/OTTERS
- **PRScs** (One of the five eQTL weight-training models whose null TWAS p values feed the omnibus calibration check) — https://github.com/getian107/PRScs
- **lassosum** (Frequentist LASSO eQTL weight-training model included among the five models tested under the null) — https://github.com/tshmak/lassosum
- **SDPR** (Bayesian DPR-model-based eQTL weight-training model included among the five models tested under the null) — https://github.com/eldronzhou/SDPR
- **PLINK** (Provides the binary bed/bim/fam LD reference genotype format loaded for simulation and extraction)
- **numpy/scipy** (Simulate the 2000 N(0,1) null phenotypes with p_causal=0.001 and h_e2=0.1)
- **pandas** (Aggregate null p values and compute per-threshold empirical type I error rates)

## Examples

```
python testing.py --models=P0.001,P0.05,lassosum,SDPR,PRScs --gwas=sim_null_gwas --out=type1_error_table.csv
```

## Evaluation signals

- Empirical type I error rates are well controlled at thresholds 10^-4 to 2.5x10^-6, matching Supplementary Table S1
- Only slight inflation is tolerated at the 10^-2 threshold
- Q-Q plot of observed vs expected null p values tracks the identity line, consistent with Supplementary Fig. S1
- All five training models (P0.001, P0.05, lassosum, SDPR, PRScs) produce null p values and the ACAT-O omnibus p values are computed
- Total replicate count reaches 10^6 null TWAS replicates (2000 phenotypes per gene)

## Limitations

- Slight type I error inflation can appear at the loosest threshold (10^-2)
- Validation is only as representative as the simulated architecture; results assume p_causal=0.001 and h_e2=0.2-style settings
- OTTERS's omnibus test does not provide association direction; only per-method TWAS Z-score signs can be consulted

## Evidence

- [abstract] abstract claim: "We show that OTTERS is a practical and powerful TWAS tool by both simulations and application studies."
- [intro] omnibus test definition: "OTTERS first constructs individual TWAS tests and p values using eQTL weights trained by each of the PRS techniques outlined above, and then calculates an omnibus test p"
- [discussion] near-optimal null performance: "this test had near-optimal performance across all simulation scenarios considered"
- [readme] stage II testing step: "perform TWAS Stage II: gene-based association test using GWAS summary statistics."
