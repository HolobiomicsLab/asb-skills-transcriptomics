---
name: gtex-expression-reference-modeling
description: Use when GReX imputation weights must be trained from whole-blood eQTL
  reference data (eQTLGen summary statistics or GTEx individual-level genotypes/expression)
  and compared by number of valid models (test R2 > 0.01).
when_to_use_negative:
- The available eQTL summary reference is not from individuals of European descent
  — the article notes available summary-level references are mainly European, limiting
  cross-population transferability
- Trans-eQTL effects are needed — OTTERS only considers cis-eQTL effects
- Only a small individual-level reference is available and summary-level data exist
  for the tissue — the larger summary-level eQTLGen reference (n=31,684 vs GTEx V6
  n=338) is preferred for power
edam_operation: http://edamontology.org/operation_3800
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_0203
tools:
- name: OTTERS
  role: Integrated framework that prepares inputs, performs LD clumping, and runs
    P+T, lassosum, SDPR, and PRS-CS to train eQTL weights per chromosome via training.py
  repo: https://github.com/daiqile96/OTTERS
- name: lassosum
  role: Frequentist LASSO regression-based PRS method invoked through R with fdrtool
    pseudo-validation and EUR.hg38 LD blocks
  repo: https://github.com/tshmak/lassosum
- name: SDPR
  role: Summary-statistics Dirichlet Process Regression method for training eQTL weights;
    requires its MKL/GSL dynamic libraries preserved
  repo: https://github.com/eldronzhou/SDPR
- name: PRS-CS
  role: Bayesian continuous-shrinkage PRS method that produced the most valid models
    (10,337 of 16,699 genes with test R2 > 0.01)
  repo: https://github.com/getian107/PRScs
- name: FUSION
  role: Baseline individual-level weight trainer applied to the GTEx V6 genotype/expression
    reference for comparison
- name: PLINK
  role: Binary PLINK format for the GTEx V6 LD reference panel; PLINK 1.9 integrated
    in OTTERS to extract input data per target gene
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-36862-w
    title: 'OTTERS: a powerful TWAS framework leveraging summary-level reference data'
  source_content_hash: sha256:1591b97aad2df6464370d7b4ef6d091aef8d4d8706252f7e527c6d5abb678796
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/daiqile96/OTTERS
  tools:
  - FUSION
  - OTTERS
  - PLINK
  - PRS-CS
  - SDPR
  - fdrtool
  - lassosum
  edam_operation: http://edamontology.org/operation_3800
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9375
n_claims: 16
---

# gtex-expression-reference-modeling

## Summary

Train GReX imputation (eQTL weight) models for TWAS Stage I from GTEx/eQTLGen reference data — using summary-level eQTLGen data (n=31,684) via P+T, lassosum, SDPR, and PRS-CS, and individual-level GTEx V6 data via FUSION — then benchmark model validity on 315 held-out GTEx V8 whole-blood samples.

## When to use

Use when GReX imputation weights must be trained from whole-blood eQTL reference data (eQTLGen summary statistics or GTEx individual-level genotypes/expression) and compared by number of valid models (test R2 > 0.01).

## When NOT to use

- The available eQTL summary reference is not from individuals of European descent — the article notes available summary-level references are mainly European, limiting cross-population transferability
- Trans-eQTL effects are needed — OTTERS only considers cis-eQTL effects
- Only a small individual-level reference is available and summary-level data exist for the tissue — the larger summary-level eQTLGen reference (n=31,684 vs GTEx V6 n=338) is preferred for power

## Inputs

- eQTLGen eQTL summary statistics (n=31,684 whole blood) with columns CHROM, POS, A1, A2, Zscore, TargetID, N
- Gene annotation file with columns CHROM, GeneStart, GeneEnd, TargetID
- GTEx V6 LD reference panel in binary PLINK format
- GTEx V6 individual-level genotype and expression reference (for FUSION)
- 315 held-out GTEx V8 whole-blood samples for validation

## Outputs

- Trained eQTL weights per gene for P+T, lassosum, SDPR, and PRS-CS (OTTERS Stage I)
- FUSION eQTL weights trained on GTEx V6
- Per-gene test R2 values in the 315 GTEx V8 whole-blood samples
- CSV summary table of per-method valid-model counts (R2 > 0.01) and median test R2

## How to apply

Sort the eQTLGen eQTL summary statistics (columns CHROM, POS, A1, A2, Zscore, TargetID, N; n=31,684) by chromosome and position, pair them with a gene annotation file (CHROM, GeneStart, GeneEnd, TargetID) and the GTEx V6 LD reference panel in binary PLINK format. Run OTTERS training.py per chromosome with --models=PT,lassosum,SDPR,PRScs, using LD clumping at --r2=0.99 for P+T (P0.001 and P0.05 thresholds), --lassosum_ld_blocks=EUR.hg38 for lassosum (with fdrtool pseudo-validation), and preserving SDPR's MKL/GSL dynamic libraries. In parallel, train FUSION eQTL weights on the GTEx V6 individual-level genotype and expression reference for the same gene set, reflecting the much smaller individual-level reference sample (n=338 for whole blood in GTEx V6). Evaluate each method's weights by predicting expression in the 315 held-out GTEx V8 whole-blood samples, computing per-gene test R2, and count valid models as genes with R2 > 0.01; write per-method valid-model counts and median test R2 to a CSV summary table.

## Related tools

- **OTTERS** (Integrated framework that prepares inputs, performs LD clumping, and runs P+T, lassosum, SDPR, and PRS-CS to train eQTL weights per chromosome via training.py) — https://github.com/daiqile96/OTTERS
- **lassosum** (Frequentist LASSO regression-based PRS method invoked through R with fdrtool pseudo-validation and EUR.hg38 LD blocks) — https://github.com/tshmak/lassosum
- **SDPR** (Summary-statistics Dirichlet Process Regression method for training eQTL weights; requires its MKL/GSL dynamic libraries preserved) — https://github.com/eldronzhou/SDPR
- **PRS-CS** (Bayesian continuous-shrinkage PRS method that produced the most valid models (10,337 of 16,699 genes with test R2 > 0.01)) — https://github.com/getian107/PRScs
- **FUSION** (Baseline individual-level weight trainer applied to the GTEx V6 genotype/expression reference for comparison)
- **PLINK** (Binary PLINK format for the GTEx V6 LD reference panel; PLINK 1.9 integrated in OTTERS to extract input data per target gene)

## Examples

```
python OTTERS/training.py --models=PT,lassosum,SDPR,PRScs --r2=0.99 --lassosum_ld_blocks=EUR.hg38 --chrom=$CHR
```

## Evaluation signals

- PRS-CS should yield ~10,337 valid models (test R2 > 0.01) among 16,699 tested genes, exceeding P+T (P0.001) with 9,816
- FUSION trained on GTEx V6 should yield ~4,704 valid models, roughly half of PRS-CS
- Median test R2 for PRS-CS should be approximately 0.0517, matching Table 1
- Per-gene predictions must successfully match expression in the 315 held-out GTEx V8 whole-blood samples with per-gene test R2 computable
- Summary CSV must contain per-method valid-model counts and median test R2 for all five training approaches

## Limitations

- Training GReX imputation models by all individual methods costs ~20 min per gene, which is computationally challenging for multiple tissues and ~20K genome-wide genes
- Available eQTL summary statistics are mainly from individuals of European descent, limiting cross-population transferability
- Training on the smaller individual-level GTEx V6 reference (n=338) yields far fewer valid models than summary-level training on eQTLGen (n=31,684)

## Evidence

- [results] PRCS yields most valid models: [evidence span withheld — anchor status: not_found]
- [intro] reference sample size comparison: "the sample size of the summary-based eQTLGen reference sample is 31,684 for blood, whereas the sample size of the individual-level GTEx V6 reference is only 338 for the same tissue"
- [methods] OTTERS training stage: "# STAGE I
# train eQTL weights using P+T, lassosum, SDPR and PRS-CS."
- [readme] OTTERS readme capabilities: "integrate [TABIX](http://www.htslib.org/doc/tabix.html) and [PLINK 1.9](https://www.cog-genomics.org/plink) tools to extract input data per target gene more efficiently"
- [discussion] FUSION on GTEx V6: "FUSION TWAS results using a substantially smaller individual-level reference dataset from GTEx V6"
- [readme] lassosum ld blocks option: "### **January 14, 2025: Added `--lassosum_ld_blocks` argument**
  * Purpose: Select LD blocks for lassosum analysis"
