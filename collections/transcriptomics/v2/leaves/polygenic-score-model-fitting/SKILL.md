---
name: polygenic-score-model-fitting
description: Use when TWAS Stage I eQTL weights must be trained from summary-level
  eQTL summary statistics (with CHROM, POS, A1, A2, Zscore, TargetID, N columns) because
  individual-level genotype-expression reference data are unavailable or too small
  (e.g., GTEx V6 n=338 vs eQTLGen n=31,684).
when_to_use_negative:
- The goal involves trans-eQTL effects or cross-population TWAS — OTTERS currently
  considers only cis-eQTL effects and available summary-level eQTL references are
  mainly European, limiting transferability
- Individual-level reference genotype-expression data of adequate size already exist
  for the tissue, making direct individual-level training (e.g., FUSION-style) applicable
  instead
- No ancestry-matched LD reference panel is available for the eQTL summary statistics
  (PRS-CS requires the reference panel to match the GWAS/eQTL sample ancestry, not
  the target sample)
edam_topics:
- http://edamontology.org/topic_3053
- http://edamontology.org/topic_0203
tools:
- name: OTTERS
  role: Integrated wrapper that performs LD-clumping, prepares inputs in the required
    formats, and runs all four PRS methods to train eQTL weights per chromosome (training.py)
  repo: https://github.com/daiqile96/OTTERS
- name: lassosum
  role: Frequentist LASSO regression on summary statistics with LD accounted for via
    the reference panel; invoked through R with fdrtool pseudo-validation
  repo: https://github.com/tshmak/lassosum
- name: SDPR
  role: Summary-statistics Dirichlet Process Regression (Bayesian nonparametric) used
    to train eQTL weights; precompiled binary requires preserved gsl/libgsl.so and
    MKL/libmkl_rt.so dynamic libraries
  repo: https://github.com/eldronzhou/SDPR
- name: PRS-CS
  role: Bayesian continuous-shrinkage prior MCMC method used to train eQTL weights
    from the eQTL summary statistics and an external EUR LD panel
  repo: https://github.com/getian107/PRScs
- name: PLINK
  role: PLINK 1.9 (with TABIX) integrated by OTTERS to extract input data per target
    gene and handle binary PLINK LD reference panels
  repo: https://www.cog-genomics.org/plink
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-36862-w
    title: 'OTTERS: a powerful TWAS framework leveraging summary-level reference data'
  source_content_hash: sha256:5e691766d8a8d17ee90f20fd65cdaaecc74169089204af1f2038fcbfd4f0d38d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/daiqile96/OTTERS
  tools:
  - OTTERS
  - PLINK
  - PRS-CS
  - SDPR
  - fdrtool
  - lassosum
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3053
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# polygenic-score-model-fitting

## Summary

Train GReX/TWAS Stage I eQTL imputation weights from summary-level eQTL statistics (e.g., eQTLGen whole blood, n=31,684) by repurposing polygenic risk score methods — P+T with LD clumping, lassosum, SDPR, and PRS-CS — inside the OTTERS framework, avoiding dependence on small individual-level expression references.

## When to use

Use when TWAS Stage I eQTL weights must be trained from summary-level eQTL summary statistics (with CHROM, POS, A1, A2, Zscore, TargetID, N columns) because individual-level genotype-expression reference data are unavailable or too small (e.g., GTEx V6 n=338 vs eQTLGen n=31,684).

## When NOT to use

- The goal involves trans-eQTL effects or cross-population TWAS — OTTERS currently considers only cis-eQTL effects and available summary-level eQTL references are mainly European, limiting transferability
- Individual-level reference genotype-expression data of adequate size already exist for the tissue, making direct individual-level training (e.g., FUSION-style) applicable instead
- No ancestry-matched LD reference panel is available for the eQTL summary statistics (PRS-CS requires the reference panel to match the GWAS/eQTL sample ancestry, not the target sample)

## Inputs

- eQTL summary statistics sorted by chromosome and position with columns CHROM, POS, A1, A2, Zscore, TargetID, N (e.g., eQTLGen whole blood, n=31,684)
- Gene annotation file with columns CHROM, GeneStart, GeneEnd, TargetID
- LD reference panel in binary PLINK format (e.g., GTEx V6)
- PRS-CS external LD reference panel (hdf5 ldblk files, e.g., ldblk_1kg_eur) and validation .bim prefix
- lassosum LD block definition file (e.g., EUR.hg38 from Berisa & Pickrell)
- Held-out individual-level genotype and expression data (e.g., 315 GTEx V8 whole blood samples) for test R2 evaluation

## Outputs

- Per-gene eQTL weights (GReX imputation models) trained by each of P+T, lassosum, SDPR, and PRS-CS
- Per-gene test R2 from predicting expression in held-out samples
- CSV summary table of valid-model counts (test R2 > 0.01) and median test R2 per method

## How to apply

Format the eQTL summary statistics sorted by chromosome and position (columns CHROM, POS, A1, A2, Zscore, TargetID, N) and supply a gene annotation file (CHROM, GeneStart, GeneEnd, TargetID) plus an LD reference panel in binary PLINK format (e.g., GTEx V6). Run OTTERS training.py per chromosome with --models=PT,lassosum,SDPR,PRScs, using LD-clumping threshold --r2=0.99 for P+T (at P0.001 and P0.05), --lassosum_ld_blocks=EUR.hg38, invoking lassosum through R with fdrtool pseudo-validation, SDPR with its MKL/GSL dynamic libraries preserved, and PRS-CS with a matching ancestry (EUR) LD panel and --n_gwas=31684. The rationale is that eQTL effect sizes are SNP effect sizes from a reference 'GWAS' of gene expression, so PRS fitting directly yields eQTL weights. Validate the trained weights by predicting expression in held-out individual-level samples (e.g., 315 GTEx V8 whole blood samples), computing per-gene test R2, and counting valid models as genes with R2 > 0.01. Expect P+T/Bayesian behavior to differ by architecture — P+T performs better for sparse architecture, Bayesian methods (PRS-CS, SDPR) for denser architecture — and consider running all four so an omnibus test can cover unknown architectures.

## Related tools

- **OTTERS** (Integrated wrapper that performs LD-clumping, prepares inputs in the required formats, and runs all four PRS methods to train eQTL weights per chromosome (training.py)) — https://github.com/daiqile96/OTTERS
- **lassosum** (Frequentist LASSO regression on summary statistics with LD accounted for via the reference panel; invoked through R with fdrtool pseudo-validation) — https://github.com/tshmak/lassosum
- **SDPR** (Summary-statistics Dirichlet Process Regression (Bayesian nonparametric) used to train eQTL weights; precompiled binary requires preserved gsl/libgsl.so and MKL/libmkl_rt.so dynamic libraries) — https://github.com/eldronzhou/SDPR
- **PRS-CS** (Bayesian continuous-shrinkage prior MCMC method used to train eQTL weights from the eQTL summary statistics and an external EUR LD panel) — https://github.com/getian107/PRScs
- **PLINK** (PLINK 1.9 (with TABIX) integrated by OTTERS to extract input data per target gene and handle binary PLINK LD reference panels) — https://www.cog-genomics.org/plink

## Examples

```
python training.py --chrom=1 --models=PT,lassosum,SDPR,PRScs --r2=0.99 --lassosum_ld_blocks=EUR.hg38
```

## Evaluation signals

- Count genes with test R2 > 0.01 per method on held-out samples: replication target is PRS-CS 10,337 valid models of 16,699 tested genes (median R2 0.0517), P+T (P0.001) 9,816, FUSION on GTEx V6 only 4,704 (Table 1 reproduction)
- Schema check: eQTL input must contain CHROM, POS, A1, A2, Zscore, TargetID, N and be sorted by chromosome and position before training
- Verify lassosum runs with fdrtool pseudo-validation and --lassosum_ld_blocks=EUR.hg38, and P+T clumping used --r2=0.99 with P0.001 and P0.05 variant sets
- Sanity check runtime: training GReX imputation models by all individual methods costs on average ~20 min per gene for all five training models
- Confirm PRS-CS was run with the ancestry-matched reference panel (EUR) and the eQTL reference sample size (--n_gwas=31684)

## Limitations

- Training GReX imputation models by all individual methods costs on average ~20 min per gene for all five training models, which is computationally challenging for ~20K genome-wide genes and multiple tissues
- Available eQTL summary statistics are mainly from individuals of European descent, limiting cross-population transferability; cross-population TWAS is out of scope (ongoing research)
- OTTERS only considers cis-eQTL effects; trans-eQTL effects are not incorporated due to computation time and lack of summary-level trans-eQTL reference data
- PRS-CS truncates p-values smaller than 1e-323 when BETA/OR + P is used as input, which may reduce prediction accuracy for traits with highly significant loci (BETA/OR + SE input is now recommended)

## Evidence

- [abstract] adapts multiple polygenic risk score (PRS) methods to estimate eQTL weights from summary-level eQTL reference data: "adapts multiple polygenic risk score (PRS) methods to estimate eQTL weights from summary-level eQTL reference data"
- [intro] We can adapt these PRS methods for TWAS since eQTL effect sizes are essentially SNP effect sizes resulting from a reference "GWAS" of gene expression: [evidence span withheld — anchor status: not_found]
- [methods] train eQTL weights using P+T, lassosum, SDPR and PRS-CS.: "train eQTL weights using P+T, lassosum, SDPR and PRS-CS."
- [discussion] P+T methods generally performing better for sparse architecture, whereas the Bayesian methods performing better for denser architecture: "P+T methods generally performing better for sparse architecture, whereas the Bayesian methods performing better for denser architecture"
- [readme] integrate [PRS-CS software](https://github.com/getian107/PRScs), [lassosum R package](https://github.com/tshmak/lassosum), and [SDPR software](https://github.com/eldronzhou/SDPR) to train eQTL weights: "integrate [PRS-CS software](https://github.com/getian107/PRScs), [lassosum R package](https://github.com/tshmak/lassosum), and [SDPR software](https://github.com/eldronzhou/SDPR) to train eQTL weights"
