---
name: ld-clumping-pvalue-thresholding
description: Use when eQTL summary statistics (e.g., eQTLGen, n=31,684) must be converted
  into per-gene expression imputation weights via P+T with LD clumping, and valid-model
  counts (test R2 > 0.01) compared across PRS-based trainers.
when_to_use_negative:
- Only individual-level reference genotype/expression data are available and small
  — FUSION is the appropriate trainer (OTTERS targets summary-level reference data).
- Non-European or cross-population eQTL data are needed — available eQTL summary statistics
  are mainly from individuals of European descent, limiting transferability.
- Direction of the gene-phenotype association is required from training alone — OTTERS
  cannot provide direction; only the sign of the per-method TWAS Z-score is available.
edam_operation: http://edamontology.org/operation_0282
edam_topics:
- http://edamontology.org/topic_0622
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_0203
tools:
- name: OTTERS
  role: Integrated pipeline that performs LD-clumping, prepares P+T/lassosum/SDPR/PRS-CS
    inputs per gene, and runs TWAS Stage I weight training via training.py per chromosome
  repo: https://github.com/daiqile96/OTTERS
- name: PLINK
  role: LD clumping and per-gene variant extraction from the binary PLINK GTEx V6
    LD reference panel
- name: TABIX
  role: Extracts per-target-gene regions from sorted, indexed eQTL summary statistics
    (extract_vcf_region_with_tabix)
  repo: http://www.htslib.org/doc/tabix.html
- name: lassosum
  role: Frequentist LASSO weight training via R with fdrtool pseudo-validation, using
    --lassosum_ld_blocks=EUR.hg38
  repo: https://github.com/tshmak/lassosum
- name: SDPR
  role: Summary-statistics Dirichlet Process Regression weight training; requires
    bundled MKL/GSL dynamic libraries preserved
  repo: https://github.com/eldronzhou/SDPR
- name: PRS-CS
  role: Bayesian continuous-shrinkage weight training invoked as a component of OTTERS
    Stage I; produced the most valid models (10,337)
  repo: https://github.com/getian107/PRScs
- name: FUSION
  role: Individual-level baseline trainer on GTEx V6 (n=338) used for comparison against
    the summary-level methods
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1038/s41467-023-36862-w
    title: 'OTTERS: a powerful TWAS framework leveraging summary-level reference data'
  source_content_hash: sha256:e6c647ee606a6906c8ff6cb51a655245d60aa3be50d0f109f6e8b8c2f147a631
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
  - TABIX
  - fdrtool
  - lassosum
  edam_operation: http://edamontology.org/operation_0282
  edam_topics:
  - http://edamontology.org/topic_0622
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 14
---

# ld-clumping-pvalue-thresholding

## Summary

Trains GReX imputation models (TWAS Stage I eQTL weights) from summary-level eQTL statistics using P+T with LD clumping, as implemented in OTTERS, and benchmarks the resulting weights against lassosum, SDPR, PRS-CS, and FUSION on held-out GTEx V8 whole blood samples.

## When to use

Use when eQTL summary statistics (e.g., eQTLGen, n=31,684) must be converted into per-gene expression imputation weights via P+T with LD clumping, and valid-model counts (test R2 > 0.01) compared across PRS-based trainers.

## When NOT to use

- Only individual-level reference genotype/expression data are available and small — FUSION is the appropriate trainer (OTTERS targets summary-level reference data).
- Non-European or cross-population eQTL data are needed — available eQTL summary statistics are mainly from individuals of European descent, limiting transferability.
- Direction of the gene-phenotype association is required from training alone — OTTERS cannot provide direction; only the sign of the per-method TWAS Z-score is available.

## Inputs

- eQTL summary statistics sorted by chromosome and position with columns CHROM, POS, A1, A2, Zscore, TargetID, N (eQTLGen, n=31,684 whole blood)
- Gene annotation file with columns CHROM, GeneStart, GeneEnd, TargetID
- LD reference panel in binary PLINK format (GTEx V6)
- LD block definition file for lassosum (EUR.hg38 from Berisa & Pickrell 2015)
- Held-out individual-level GTEx V8 whole-blood genotype and expression data (n=315) for evaluation

## Outputs

- Per-gene eQTL weight files (GReX imputation models) trained by P+T (P0.001 and P0.05), lassosum, SDPR, and PRS-CS
- Per-gene test R2 values from prediction in 315 held-out GTEx V8 samples
- CSV summary table of per-method valid-model counts (R2 > 0.01) and median test R2 (e.g., PRS-CS: 10,337 valid genes, median R2 0.0517; FUSION: 4,704)

## How to apply

Sort the eQTLGen summary statistics by chromosome and position with columns CHROM, POS, A1, A2, Zscore, TargetID, N, and pair them with a gene annotation file (CHROM, GeneStart, GeneEnd, TargetID) and a binary PLINK LD reference panel (GTEx V6). Run OTTERS training.py per chromosome with --models=PT,lassosum,SDPR,PRScs, using a permissive LD-clumping threshold --r2=0.99 and P+T p-value cutoffs of P0.001 and P0.05 so clumped SNPs are retained for weight estimation. OTTERS integrates TABIX and PLINK 1.9 to extract per-gene genotype regions and prepares inputs for each PRS method, invoking lassosum through R with fdrtool pseudo-validation (LD blocks selected via --lassosum_ld_blocks=EUR.hg38) and running SDPR with its bundled MKL/GSL dynamic libraries intact. Evaluate each method's trained weights by predicting expression in 315 held-out GTEx V8 whole-blood samples, compute per-gene test R2, and count valid models as genes with R2 > 0.01. The expected outcome, grounded in this article's Table 1, is that PRS-CS yields the most valid models (10,337 of 16,699 genes; median R2 0.0517), P+T at P0.001 yields 9,816, and FUSION trained on the much smaller GTEx V6 individual-level reference (n=338) yields only 4,704 — demonstrating that the larger summary-level reference compensates for P+T's simplicity.

## Related tools

- **OTTERS** (Integrated pipeline that performs LD-clumping, prepares P+T/lassosum/SDPR/PRS-CS inputs per gene, and runs TWAS Stage I weight training via training.py per chromosome) — https://github.com/daiqile96/OTTERS
- **PLINK** (LD clumping and per-gene variant extraction from the binary PLINK GTEx V6 LD reference panel)
- **TABIX** (Extracts per-target-gene regions from sorted, indexed eQTL summary statistics (extract_vcf_region_with_tabix)) — http://www.htslib.org/doc/tabix.html
- **lassosum** (Frequentist LASSO weight training via R with fdrtool pseudo-validation, using --lassosum_ld_blocks=EUR.hg38) — https://github.com/tshmak/lassosum
- **SDPR** (Summary-statistics Dirichlet Process Regression weight training; requires bundled MKL/GSL dynamic libraries preserved) — https://github.com/eldronzhou/SDPR
- **PRS-CS** (Bayesian continuous-shrinkage weight training invoked as a component of OTTERS Stage I; produced the most valid models (10,337)) — https://github.com/getian107/PRScs
- **FUSION** (Individual-level baseline trainer on GTEx V6 (n=338) used for comparison against the summary-level methods)

## Examples

```
python training.py --chrom=1 --models=PT,lassosum,SDPR,PRScs --r2=0.99 --lassosum_ld_blocks=EUR.hg38
```

## Evaluation signals

- Valid-model count check: P+T at P0.001 should yield 9,816 genes with test R2 > 0.01 out of 16,699 tested; PRS-CS should exceed all methods with 10,337.
- FUSION trained on GTEx V6 individual-level data should yield only 4,704 valid models — far fewer than summary-level methods, reflecting the small reference (n=338 vs n=31,684).
- Median test R2 per method should match Table 1 (PRS-CS median R2 0.0517) when evaluated on the same 315 held-out GTEx V8 samples.
- Input schema check: eQTL summary file must be sorted by chromosome and position and contain CHROM, POS, A1, A2, Zscore, TargetID, N columns before training.
- SDPR run must retain its dynamic libraries (gsl/lib/libgsl.so, MKL/lib/libmkl_rt.so); a load error indicates an environment misconfiguration, not a training failure.

## Limitations

- Training GReX imputation models by all individual methods costs ~20 min per gene, which is computationally challenging for multiple tissues and ~20K genome-wide genes.
- Available eQTL summary statistics are mainly from individuals of European descent, limiting cross-population transferability of trained weights.
- OTTERS currently considers only cis-eQTL effects; trans-eQTL effects are not incorporated.

## Evidence

- [methods] P+T clumping parameters: [evidence span withheld — anchor status: not_found]
- [intro] eQTLGen vs GTEx V6 sample sizes: "the sample size of the summary-based eQTLGen reference sample is 31,684 for blood, whereas the sample size of the individual-level GTEx V6 reference is only 338 for the same tissue"
- [abstract] PRS methods adapted for weight training: "adapts multiple polygenic risk score (PRS) methods to estimate eQTL weights from summary-level eQTL reference data"
- [readme] OTTERS tool integrations: "integrate [PRS-CS software](https://github.com/getian107/PRScs), [lassosum R package](https://github.com/tshmak/lassosum), and [SDPR software](https://github.com/eldronzhou/SDPR) to train eQTL weights"
- [readme] PRS-CS invocation: [evidence span withheld — anchor status: external_doc]
- [readme] lassosum LD blocks: "### **January 14, 2025: Added `--lassosum_ld_blocks` argument**"
