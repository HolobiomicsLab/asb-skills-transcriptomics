---
name: null-phenotype-simulation
description: Use when validating that a gene-based association or omnibus test (e.g.,
  OTTERS ACAT-O) is well calibrated under the null before applying it to real GWAS
  summary data.
when_to_use_negative:
- The goal is power or effect-size evaluation rather than null calibration — simulation
  should include causal effects instead of N(0,1) null phenotypes
- No LD reference panel or trained eQTL weights are available, so Stage II testing.py
  cannot be run on the simulated null GWAS
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3517
tools:
- name: OTTERS
  role: Runs testing.py to compute null gene-based TWAS p values and ACAT-O omnibus
    p values on the simulated phenotypes
  repo: https://github.com/daiqile96/OTTERS
- name: PLINK
  role: Provides the binary bed/bim/fam LD reference panel genotypes and per-gene
    extraction used to build null GWAS replicates
- name: PRS-CS
  role: One of the Stage I weight-training models whose null p values enter the omnibus
    calibration check
  repo: https://github.com/getian107/PRScs
- name: lassosum
  role: Stage I weight-training method included in the --models list for null testing
  repo: https://github.com/tshmak/lassosum
- name: SDPR
  role: Stage I weight-training method included in the --models list for null testing
  repo: https://github.com/eldronzhou/SDPR
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-36862-w
    title: 'OTTERS: a powerful TWAS framework leveraging summary-level reference data'
  source_content_hash: sha256:2b5365c9dd7f087f8fb7e7f4462244ccf77c8a2c774003103043064df1c0570a
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8571428571428571
n_claims: 14
---

# null-phenotype-simulation

## Summary

Simulate quantitative null GWAS phenotypes from N(0,1) under a specified causal-variant proportion and heritability, run the full TWAS pipeline on the null data, and quantify empirical type I error rates and Q-Q calibration of the omnibus p-value distribution.

## When to use

Use when validating that a gene-based association or omnibus test (e.g., OTTERS ACAT-O) is well calibrated under the null before applying it to real GWAS summary data.

## When NOT to use

- The goal is power or effect-size evaluation rather than null calibration — simulation should include causal effects instead of N(0,1) null phenotypes
- No LD reference panel or trained eQTL weights are available, so Stage II testing.py cannot be run on the simulated null GWAS

## Inputs

- LD reference panel genotypes in PLINK binary bed/bim/fam format
- Gene annotation file consumed by OTTERS
- Stage I-trained eQTL weights from P+T (P0.001, P0.05), lassosum, SDPR, and PRS-CS models

## Outputs

- Empirical type I error table (type1_error_table.csv) at thresholds 10^-2, 10^-4, 2.5x10^-6
- Q-Q plot of observed vs expected null TWAS p values (qq_plot.png)

## How to apply

Load the LD reference panel genotypes in PLINK binary bed/bim/fam format together with the gene annotation file, then use numpy/scipy to draw 2000 quantitative phenotypes from N(0,1) under causal-variant proportion p_causal=0.001 and heritability h_e2=0.1, generating ~10^6 null TWAS replicates. Run OTTERS testing.py (e.g., --models=P0.001,P0.05,lassosum,SDPR,PRScs) with Stage I-trained eQTL weights on each simulated null GWAS to obtain per-gene null p values including the ACAT-O omnibus p. Aggregate null p values with pandas and compute empirical type I error rates per threshold (e.g., 10^-2, 10^-4, 2.5x10^-6). Construct a Q-Q plot of observed vs expected null p values; calibration is confirmed when empirical rates match Supplementary Table S1 expectations and the Q-Q curve tracks the identity line in the tails.

## Related tools

- **OTTERS** (Runs testing.py to compute null gene-based TWAS p values and ACAT-O omnibus p values on the simulated phenotypes) — https://github.com/daiqile96/OTTERS
- **PLINK** (Provides the binary bed/bim/fam LD reference panel genotypes and per-gene extraction used to build null GWAS replicates)
- **PRS-CS** (One of the Stage I weight-training models whose null p values enter the omnibus calibration check) — https://github.com/getian107/PRScs
- **lassosum** (Stage I weight-training method included in the --models list for null testing) — https://github.com/tshmak/lassosum
- **SDPR** (Stage I weight-training method included in the --models list for null testing) — https://github.com/eldronzhou/SDPR

## Examples

```
python testing.py --models=P0.001,P0.05,lassosum,SDPR,PRScs --eQTL_weights stage1_weights/ --gwas null_pheno_gwas/ --out null_twas_pvalues/
```

## Evaluation signals

- Empirical type I error rates are well controlled at thresholds 10^-4 to 2.5x10^-6, matching Supplementary Table S1
- Only slight inflation is observed at the loose 10^-2 threshold
- Q-Q plot of null TWAS p values tracks the identity line, consistent with Supplementary Fig. S1
- Replicate count reaches ~10^6 null TWAS tests across the 2000 simulated phenotypes

## Limitations

- Slight type I error inflation is expected at the 10^-2 threshold, so calibration claims should focus on the tails of the p-value distribution
- OTTERS output does not indicate the direction of gene-phenotype associations; only per-method TWAS Z-score signs can be inspected
- Cross-population transferability is limited because available eQTL summary statistics (e.g., eQTLGen) are mainly from individuals of European descent

## Evidence

- [results] OTTERS was well calibrated in the tails of the TWAS p-value distribution under null simulations, as shown by quantile-quantile (Q-Q) plots in Supplementary Fig. S1: [evidence span withheld — anchor status: not_found]
- [methods] simulate 2000 quantitative phenotypes drawn from N(0,1) under the causal-variant proportion p_causal=0.001 and heritability h_e2=0.1: [evidence span withheld — anchor status: not_found]
- [methods] Run OTTERS testing.py with the Stage I-trained eQTL weights on each simulated null GWAS to obtain null gene-based association p values, including the ACAT-O omnibus p values: [evidence span withheld — anchor status: not_found]
- [methods] Load the LD reference panel genotype data (PLINK binary bed/bim/fam files) and the annotation file with OTTERS: [evidence span withheld — anchor status: not_found]
