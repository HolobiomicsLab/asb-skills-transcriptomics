---
name: eqtl-weight-based-grex-association-testing
description: Use when GWAS summary statistics and pre-trained eQTL weights (per-gene
  SNP effect sizes from a reference panel) are available and the goal is to identify
  genes whose genetically regulated expression associates with a trait.
when_to_use_negative:
- Only individual-level reference expression data are available and the goal is FUSION-style
  TWAS with a small reference (OTTERS is designed for summary-level reference data)
- The aim is to infer the direction of gene-phenotype associations — OTTERS cannot
  provide direction, only the sign of per-method TWAS Z-scores
- The question requires trans-eQTL effects — OTTERS only considers cis-eQTL effects
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3517
tools:
- name: OTTERS
  role: Integrated framework that runs Stage II gene-based association testing and
    computes ACAT-O omnibus p values from multiple weight models
  repo: github.com/daiqile96/OTTERS
- name: PLINK 1.9
  role: Extracts genotype/input data per target gene for the association test
- name: TABIX
  role: Extracts indexed genomic regions per target gene from summary data
- name: PRS-CS
  role: Bayesian continuous-shrinkage method providing one of the eQTL weight models
    whose TWAS p values feed the ACAT-O omnibus test
  repo: github.com/getian107/PRScs
- name: lassosum
  role: Frequentist LASSO method providing one of the eQTL weight models for TWAS
    testing
  repo: github.com/tshmak/lassosum
- name: SDPR
  role: Dirichlet Process Regression-based summary-statistics method providing one
    of the eQTL weight models for TWAS testing
  repo: github.com/eldronzhou/SDPR
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-36862-w
    title: 'OTTERS: a powerful TWAS framework leveraging summary-level reference data'
  source_content_hash: sha256:011066a5ebc9e4d97243af4b8d9b403cd332cc527a0265a90db7065387c6e562
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - OTTERS
  - PLINK 1.9
  - PRS-CS
  - Python
  - SDPR
  - TABIX
  - lassosum
  - numpy
  - scipy
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3517
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7692307692307693
n_claims: 13
---

# eqtl-weight-based-grex-association-testing

## Summary

Performs TWAS Stage II gene-based association testing: impute genetically regulated gene expression (GReX) in GWAS data using trained eQTL weights (from P+T, lassosum, SDPR, PRS-CS, or FUSION) and test the imputed GReX–phenotype association, with OTTERS aggregating per-method p values into an ACAT-O omnibus test.

## When to use

Use when GWAS summary statistics and pre-trained eQTL weights (per-gene SNP effect sizes from a reference panel) are available and the goal is to identify genes whose genetically regulated expression associates with a trait.

## When NOT to use

- Only individual-level reference expression data are available and the goal is FUSION-style TWAS with a small reference (OTTERS is designed for summary-level reference data)
- The aim is to infer the direction of gene-phenotype associations — OTTERS cannot provide direction, only the sign of per-method TWAS Z-scores
- The question requires trans-eQTL effects — OTTERS only considers cis-eQTL effects

## Inputs

- GWAS summary statistics (e.g., UK Biobank cardiovascular disease summary-level GWAS)
- Stage I-trained eQTL weights per gene (from P+T, lassosum, SDPR, PRS-CS, or FUSION)
- LD reference panel matching the GWAS ancestry (PLINK bed/bim/fam or summary-level LD)
- Annotation file mapping genes to cis-regions

## Outputs

- Per-gene, per-model TWAS association statistics (Z-scores and p values)
- ACAT-O omnibus gene-level p values
- List of significant risk genes (e.g., genes missed by FUSION in the UKBB application)

## How to apply

For each gene, take the Stage I-trained eQTL weights and the GWAS summary statistics, impute GReX, and compute a gene-based association Z-score/p value (OTTERS's testing.py with --models=P0.001,P0.05,lassosum,SDPR,PRScs covers all five weight sources). Because the optimal training method depends on the unknown expression genetic architecture (P+T better for sparse, Bayesian methods like PRS-CS/SDPR better for dense), combine the per-method TWAS p values with ACAT-O to obtain a single omnibus gene-level p value. OTTERS integrates TABIX and PLINK 1.9 to extract input data per target gene and supports parallel computation across genes. Run on GWAS summary data with matching LD reference (e.g., eQTLGen blood reference, n=31,684) as demonstrated on UK Biobank cardiovascular disease GWAS. Verify calibration under null simulations before interpreting real-data hits.

## Related tools

- **OTTERS** (Integrated framework that runs Stage II gene-based association testing and computes ACAT-O omnibus p values from multiple weight models) — github.com/daiqile96/OTTERS
- **PLINK 1.9** (Extracts genotype/input data per target gene for the association test)
- **TABIX** (Extracts indexed genomic regions per target gene from summary data)
- **PRS-CS** (Bayesian continuous-shrinkage method providing one of the eQTL weight models whose TWAS p values feed the ACAT-O omnibus test) — github.com/getian107/PRScs
- **lassosum** (Frequentist LASSO method providing one of the eQTL weight models for TWAS testing) — github.com/tshmak/lassosum
- **SDPR** (Dirichlet Process Regression-based summary-statistics method providing one of the eQTL weight models for TWAS testing) — github.com/eldronzhou/SDPR

## Examples

```
python OTTERS/testing.py --models=P0.001,P0.05,lassosum,SDPR,PRScs --gwas=gwas_summary.txt --weights=eqtl_weights/ --out=twas_results.txt
```

## Evaluation signals

- Null-simulation calibration: empirical type I error rates controlled at thresholds 10^-4 to 2.5x10^-6 (only slight inflation at 10^-2), matching Supplementary Table S1
- Q-Q plot of null TWAS p values tracks the identity line, consistent with Supplementary Fig. S1
- Each of the five models (P0.001, P0.05, lassosum, SDPR, PRScs) yields a valid per-gene TWAS p value, and ACAT-O returns a single omnibus p value per gene
- Real-data sanity check: OTTERS on eQTLGen + UKBB cardiovascular GWAS should reveal risk genes missed by FUSION using GTEx V6

## Limitations

- Training GReX imputation models by all five methods costs ~20 min per gene, which is computationally challenging for ~20K genome-wide genes and multiple tissues
- OTTERS cannot provide the direction of identified gene-phenotype associations; users must refer to the sign of the TWAS Z-score per training method
- Available eQTL summary statistics are mainly from individuals of European descent, limiting cross-population transferability
- Only cis-eQTL effects are considered; trans-eQTL incorporation is out of scope

## Evidence

- [intro] Stage II of TWAS proceeds by using these trained eQTL effect sizes to impute GReX within an independent GWAS of a complex human disease or trait: "Stage II of TWAS proceeds by using these trained eQTL effect sizes to impute GReX within an independent GWAS of a complex human disease or trait"
- [intro] OTTERS first constructs individual TWAS tests and p values using eQTL weights trained by each of the PRS techniques outlined above, and then calculates an omnibus test p value: "OTTERS first constructs individual TWAS tests and p values using eQTL weights trained by each of the PRS techniques outlined above, and then calculates an omnibus test p value using the aggregated"
- [discussion] (2) Conduct TWAS (i.e., testing gene-trait association in Stage II) using both individual-level and summary-level GWAS data with the estimated eQTL weights: "(2) Conduct TWAS (i.e., testing gene-trait association in Stage II) using both individual-level and summary-level GWAS data with the estimated eQTL weights"
- [readme] integrate TABIX and PLINK 1.9 tools to extract input data per target gene more efficiently: "integrate [TABIX](http://www.htslib.org/doc/tabix.html) and [PLINK 1.9](https://www.cog-genomics.org/plink) tools to extract input data per target gene more efficiently"
- [other] Run OTTERS testing.py (--models=P0.001,P0.05,lassosum,SDPR,PRScs) with the Stage I-trained eQTL weights on each simulated null GWAS: [evidence span withheld — anchor status: not_found]
- [results] OTTERS was well calibrated in the tails of the TWAS p-value distribution under null simulations, as shown by quantile-quantile (Q-Q) plots: [evidence span withheld — anchor status: not_found]
