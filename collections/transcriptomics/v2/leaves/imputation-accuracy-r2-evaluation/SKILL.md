---
name: imputation-accuracy-r2-evaluation
description: Use when comparing TWAS weight-training methods (summary-level PRS methods
  vs FUSION) by imputation accuracy, and a held-out individual-level genotype+expression
  reference (e.g., GTEx V8 whole blood, n=315) is available to compute per-gene test
  R2.
when_to_use_negative:
- No held-out individual-level genotype and expression data exist for the target tissue
  — per-gene test R2 cannot be computed and only out-of-sample TWAS associations (Stage
  II) are possible
- The goal is the Stage II gene-phenotype association test itself, not the benchmarking
  of weight-training methods; use the omnibus ACAT-O TWAS workflow instead
- Working in non-European populations — eQTLGen-derived weights and their R2 benchmarks
  reflect individuals of European descent and may not transfer
tools:
- name: OTTERS
  role: Integrated pipeline that runs all four summary-level training methods per
    gene and enables the parallel training whose outputs are evaluated here
  repo: https://github.com/daiqile96/OTTERS
- name: PRS-CS
  role: Best-performing trainer under this R2 evaluation (10,337 valid models of 16,699);
    invoked via OTTERS for eQTL weight training
  repo: https://github.com/getian107/PRScs
- name: lassosum
  role: Trainer whose weights are evaluated; uses fdrtool pseudo-validation and Berisa
    & Pickrell LD blocks (--lassosum_ld_blocks)
  repo: https://github.com/tshmak/lassosum
- name: SDPR
  role: Summary-statistics DPR trainer whose weights are evaluated under the same
    R2 criterion
  repo: https://github.com/eldronzhou/SDPR
- name: FUSION
  role: Individual-level reference baseline (trained on GTEx V6, n=338) benchmarked
    against summary-level methods; obtained only 4,704 valid models
- name: PLINK
  role: Used within OTTERS (call_PLINK_extract) to extract per-gene genotypes for
    training and for predicting expression in the validation panel
  repo: https://www.cog-genomics.org/plink
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-36862-w
    title: 'OTTERS: a powerful TWAS framework leveraging summary-level reference data'
  source_content_hash: sha256:ec7016328e6123c7c1f9ca4517e6d8bf16364b9eaa52a12db2978ab0d64928c9
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
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9444444444444444
n_claims: 18
---

# imputation-accuracy-r2-evaluation

## Summary

Benchmark protocol for TWAS Stage I GReX imputation models: trained eQTL weights (from P+T, lassosum, SDPR, PRS-CS on eQTLGen summary data, or FUSION on individual-level GTEx V6) are evaluated by predicting expression in 315 held-out GTEx V8 whole-blood samples and computing per-gene test R2, counting 'valid' models at R2 > 0.01.

## When to use

Use when comparing TWAS weight-training methods (summary-level PRS methods vs FUSION) by imputation accuracy, and a held-out individual-level genotype+expression reference (e.g., GTEx V8 whole blood, n=315) is available to compute per-gene test R2.

## When NOT to use

- No held-out individual-level genotype and expression data exist for the target tissue — per-gene test R2 cannot be computed and only out-of-sample TWAS associations (Stage II) are possible
- The goal is the Stage II gene-phenotype association test itself, not the benchmarking of weight-training methods; use the omnibus ACAT-O TWAS workflow instead
- Working in non-European populations — eQTLGen-derived weights and their R2 benchmarks reflect individuals of European descent and may not transfer

## Inputs

- Trained eQTL weights per method (P+T with --r2=0.99 clumping, lassosum with fdrtool pseudo-validation, SDPR, PRS-CS) trained on eQTLGen whole-blood summary statistics (n=31,684)
- FUSION eQTL weights trained on individual-level GTEx V6 genotype and expression for the same gene set
- GTEx V8 whole-blood individual-level genotypes and normalized expression for 315 held-out samples (validation panel)
- Gene annotation file with CHROM, GeneStart, GeneEnd, TargetID
- Per-gene OTTERS training.py outputs from chromosomes 1-22 (plus X, Y in recent versions)

## Outputs

- Per-gene test R2 for each training method evaluated on the 315 held-out GTEx V8 whole-blood samples
- Per-method count of valid imputation models (genes with R2 > 0.01), e.g., PRS-CS 10,337; P+T (P0.001) 9,816; FUSION 4,704 of 16,699 tested genes
- Per-method median test R2 (e.g., PRS-CS 0.0517)
- CSV summary table of valid-model counts and median R2 per method (reproducing Table 1)

## How to apply

For each trained weight set (P+T at P0.001/P0.05, lassosum, SDPR, PRS-CS from eQTLGen n=31,684, and FUSION weights trained on GTEx V6 n=338), predict GReX in the 315 held-out GTEx V8 whole-blood samples and compute per-gene test R2. Define a model as 'valid' when test R2 > 0.01, then count valid models per method and compute the median test R2 per method. Summarize counts and medians in a CSV mirroring Table 1 (PRS-CS: 10,337 valid genes, median R2 0.0517; FUSION: 4,704 valid genes). The rationale is that summary-level references far exceed individual-level reference sample size (eQTLGen 31,684 vs GTEx V6 338 for blood), so the R2-based valid-model count directly quantifies whether the larger reference translates into better GReX imputation.

## Related tools

- **OTTERS** (Integrated pipeline that runs all four summary-level training methods per gene and enables the parallel training whose outputs are evaluated here) — https://github.com/daiqile96/OTTERS
- **PRS-CS** (Best-performing trainer under this R2 evaluation (10,337 valid models of 16,699); invoked via OTTERS for eQTL weight training) — https://github.com/getian107/PRScs
- **lassosum** (Trainer whose weights are evaluated; uses fdrtool pseudo-validation and Berisa & Pickrell LD blocks (--lassosum_ld_blocks)) — https://github.com/tshmak/lassosum
- **SDPR** (Summary-statistics DPR trainer whose weights are evaluated under the same R2 criterion) — https://github.com/eldronzhou/SDPR
- **FUSION** (Individual-level reference baseline (trained on GTEx V6, n=338) benchmarked against summary-level methods; obtained only 4,704 valid models)
- **PLINK** (Used within OTTERS (call_PLINK_extract) to extract per-gene genotypes for training and for predicting expression in the validation panel) — https://www.cog-genomics.org/plink

## Examples

```
python training.py --models=PT,lassosum,SDPR,PRScs --r2=0.99 --lassosum_ld_blocks=EUR.hg38  # then evaluate per-gene test R2 in 315 GTEx V8 held-out samples and count R2>0.01
```

## Evaluation signals

- Per-method valid-model counts approach Table 1: PRS-CS 10,337, P+T (P0.001) 9,816, FUSION 4,704 valid genes out of 16,699 tested
- Median test R2 for PRS-CS reproduces the published 0.0517
- Intra-run invariant: every gene's test R2 lies in a plausible range (R2 > 0 for valid models; valid threshold strictly R2 > 0.01)
- FUSION weights trained on the smaller GTEx V6 reference (n=338) should yield substantially fewer valid models than the summary-level eQTLGen-trained methods (n=31,684)
- CSV summary schema contains one row per method with columns for valid-model count and median test R2

## Limitations

- Training GReX imputation models by all five methods costs ~20 min per gene on average, making genome-wide (~20K genes) and multi-tissue evaluations computationally heavy
- eQTLGen-derived weights are mainly from individuals of European descent, limiting transferability of the R2 benchmark to other populations
- OTTERS only considers cis-eQTL effects and cannot provide the direction of gene-phenotype associations (use the sign of the TWAS Z-score per training method)

## Evidence

- [intro] eQTLGen vs GTEx V6 sample size contrast: "the sample size of the summary-based eQTLGen reference sample is 31,684 for blood, whereas the sample size of the individual-level GTEx V6 reference is only 338 for the same tissue"
- [discussion] FUSION baseline on smaller reference: "FUSION TWAS results using a substantially smaller individual-level reference dataset from GTEx V6"
- [other] Valid-model counting rule: [evidence span withheld — anchor status: not_found]
- [discussion] Per-gene training cost: "training GReX imputation models by all individual methods on average cost ~20 min for all five training models per gene"
- [readme] OTTERS parallel training of imputation models: "enable parallel computation to train GReX imputation models / perform gene-based association test simultaneously for multiple genes."
