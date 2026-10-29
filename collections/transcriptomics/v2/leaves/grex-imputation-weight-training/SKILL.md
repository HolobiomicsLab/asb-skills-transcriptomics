---
name: grex-imputation-weight-training
description: 'Use when the goal is TWAS Stage I: estimating per-gene eQTL weights
  from a reference panel, especially when only summary-level eQTL data (e.g., eQTLGen,
  GTEx V8) are available instead of individual-level genotype/expression data.'
when_to_use_negative:
- The analysis needs trans-eQTL effects or cross-population transferability to non-European
  reference data — OTTERS currently considers only cis-eQTL effects and available
  summary references are mainly European
- The downstream result must report the direction of the gene-phenotype association
  from the omnibus test — OTTERS cannot provide this; users must refer to per-method
  TWAS Z-scores
- Reference eQTL summary statistics for the target tissue are unavailable and no individual-level
  genotype/expression data exist as an alternative reference
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_3053
tools:
- name: OTTERS
  role: Integrated TWAS Stage I framework that runs LD-clumping, formats inputs, and
    trains eQTL weights with all four PRS methods in parallel
  repo: https://github.com/daiqile96/OTTERS
- name: lassosum
  role: Frequentist LASSO regression-based PRS method wrapped by OTTERS to estimate
    eQTL weights, with pseudo-validation
  repo: https://github.com/tshmak/lassosum
- name: SDPR
  role: Summary-statistics Dirichlet Process Regression method wrapped by OTTERS to
    train eQTL weights
  repo: https://github.com/eldronzhou/SDPR
- name: PRS-CS
  role: Bayesian continuous-shrinkage PRS method wrapped by OTTERS to train eQTL weights;
    needs LD reference panels (e.g., 1000G EUR) and validation BIM prefix
  repo: https://github.com/getian107/PRScs
- name: FUSION
  role: Trains eQTL weights from individual-level reference genotypes and expression
    for the matched head-to-head benchmark (GTEx V8 whole blood, n=574)
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-023-36862-w
    title: 'OTTERS: a powerful TWAS framework leveraging summary-level reference data'
  source_content_hash: sha256:55479f97e207ddd397188b37735e138580bf5b2670080633dbd2bae6ce7a6d4d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/daiqile96/OTTERS
  tools:
  - FUSION
  - OTTERS
  - PRS-CS
  - SDPR
  - lassosum
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_3053
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 9
---

# grex-imputation-weight-training

## Summary

Trains GReX/eQTL imputation weights (TWAS Stage I) from reference data, either summary-level eQTL statistics via OTTERS' adaptation of four PRS methods (P+T, lassosum, SDPR, PRS-CS), or individual-level genotypes and expression via FUSION, producing per-gene weight files for downstream TWAS testing.

## When to use

Use when the goal is TWAS Stage I: estimating per-gene eQTL weights from a reference panel, especially when only summary-level eQTL data (e.g., eQTLGen, GTEx V8) are available instead of individual-level genotype/expression data.

## When NOT to use

- The analysis needs trans-eQTL effects or cross-population transferability to non-European reference data — OTTERS currently considers only cis-eQTL effects and available summary references are mainly European
- The downstream result must report the direction of the gene-phenotype association from the omnibus test — OTTERS cannot provide this; users must refer to per-method TWAS Z-scores
- Reference eQTL summary statistics for the target tissue are unavailable and no individual-level genotype/expression data exist as an alternative reference

## Inputs

- eQTL summary statistics (CHROM, POS, A1, A2, Zscore, TargetID, N), sorted by chromosome then ascending SNP position
- Gene annotation file (CHROM, GeneStart, GeneEnd, TargetID)
- LD reference panel in PLINK 1.9 genotype format
- Individual-level reference genotypes and expression (for FUSION-weight training, e.g., GTEx V8 whole blood, n=574)
- GWAS summary statistics target cohort metadata (e.g., UKBB cardiovascular GWAS, for BIM prefix in PRS-CS)

## Outputs

- Per-gene eQTL weight files for each training method (P0.001/P0.05 P+T weights, lassosum, SDPR, PRScs)
- Trained weights ready for TWAS Stage II gene-based association testing against GWAS summary statistics

## How to apply

Prepare OTTERS inputs: an annotation file (CHROM, GeneStart, GeneEnd, TargetID), eQTL summary statistics sorted by chromosome then ascending SNP position (CHROM, POS, A1, A2, Zscore, TargetID, N), and an LD reference panel of genotype data. Run STAGE I via OTTERS training.py with --models=PT,lassosum,SDPR,PRScs and an LD-clumping threshold of --r2=0.99; OTTERS integrates TABIX and PLINK 1.9 to extract per-gene data, then wraps the PRS-CS software, lassosum R package, and SDPR software to train each method's weights, enabling parallel computation across genes. For lassosum, select LD blocks with --lassosum_ld_blocks (EUR.hg38 by default, per Berisa & Pickrell 2015). When individual-level reference data exist (e.g., the same GTEx V8 whole blood samples, n=574), train FUSION weights on the same samples for a matched head-to-head comparison. Budget ~20 min per gene for all five training models, which becomes computationally challenging for ~20K genes across multiple tissues.

## Related tools

- **OTTERS** (Integrated TWAS Stage I framework that runs LD-clumping, formats inputs, and trains eQTL weights with all four PRS methods in parallel) — https://github.com/daiqile96/OTTERS
- **lassosum** (Frequentist LASSO regression-based PRS method wrapped by OTTERS to estimate eQTL weights, with pseudo-validation) — https://github.com/tshmak/lassosum
- **SDPR** (Summary-statistics Dirichlet Process Regression method wrapped by OTTERS to train eQTL weights) — https://github.com/eldronzhou/SDPR
- **PRS-CS** (Bayesian continuous-shrinkage PRS method wrapped by OTTERS to train eQTL weights; needs LD reference panels (e.g., 1000G EUR) and validation BIM prefix) — https://github.com/getian107/PRScs
- **FUSION** (Trains eQTL weights from individual-level reference genotypes and expression for the matched head-to-head benchmark (GTEx V8 whole blood, n=574))

## Examples

```
python training.py --models=PT,lassosum,SDPR,PRScs --r2=0.99
```

## Evaluation signals

- Head-to-head TWAS against the same UKBB cardiovascular GWAS reproduces Supplementary Fig. S11: OTTERS finds 34 independently significant genes, FUSION 21, with 14 shared
- eQTL summary statistics are correctly sorted by chromosome then ascending SNP position before training; GWAS summary statistics need no sorting
- STAGE I is run with all four models (--models=PT,lassosum,SDPR,PRScs) and LD-clumping threshold --r2=0.99
- Runtime stays near the reported benchmark of ~20 min per gene for all five training models
- Downstream STAGE II p-values from P0.001, P0.05, lassosum, SDPR and PRScs are combined into ACAT-O omnibus TWAS p values

## Limitations

- Training GReX imputation models by all individual methods costs ~20 min per gene for all five models, which may be computationally challenging for ~20K genes and multiple tissues
- Available eQTL summary statistics are mainly from individuals of European descent, limiting cross-population transferability of TWAS results
- OTTERS considers only cis-eQTL effects and cannot provide the direction of identified gene-phenotype associations (only per-method TWAS Z-scores)

## Evidence

- [abstract] Stage I training rationale: "adapts multiple polygenic risk score (PRS) methods to estimate eQTL weights from summary-level eQTL reference data"
- [methods] STAGE I command definition: "# STAGE I
# train eQTL weights using P+T, lassosum, SDPR and PRS-CS."
- [discussion] Computational cost: "training GReX imputation models by all individual methods on average cost ~20 min for all five training models per gene"
- [readme] OTTERS tool integration: "integrate [PRS-CS software](https://github.com/getian107/PRScs), [lassosum R package](https://github.com/tshmak/lassosum), and [SDPR software](https://github.com/eldronzhou/SDPR) to train eQTL weights"
- [readme] lassosum LD block selection: "Purpose: Select LD blocks for lassosum analysis"
