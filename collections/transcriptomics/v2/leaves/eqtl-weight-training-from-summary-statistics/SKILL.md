---
name: eqtl-weight-training-from-summary-statistics
description: Use when eQTL summary statistics (CHROM, POS, A1, A2, Zscore, TargetID,
  N) plus an LD reference panel are available but individual-level genotype/expression
  data are not, and gene-level eQTL weights are needed for TWAS.
when_to_use_negative:
- Individual-level reference genotype and expression data are available and desired
  — the article benchmarks FUSION trained on individual-level data as the alternative
  baseline
- Trans-eQTL effects are required — OTTERS currently considers only cis-eQTL effects
- Reference data are from a non-European population — available eQTL summary statistics
  are mainly European, and cross-population TWAS is listed as ongoing work
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_0203
tools:
- name: OTTERS
  role: 'Integrated Stage I driver: LD-clumping, input preparation, and orchestration
    of P+T, lassosum, SDPR, and PRS-CS to train eQTL weights'
  repo: https://github.com/daiqile96/OTTERS.git
- name: lassosum
  role: Frequentist LASSO regression on summary statistics with LD from a reference
    panel, run via R with fdrtool pseudo-validation
  repo: https://github.com/tshmak/lassosum
- name: SDPR
  role: Summary-statistics-based Dirichlet Process Regression (Bayesian nonparametric)
    PRS method for eQTL weight estimation
  repo: https://github.com/eldronzhou/SDPR.git
- name: PRS-CS
  role: Bayesian continuous shrinkage prior MCMC method inferring posterior SNP effect
    sizes for eQTL weights from summary statistics and an external LD panel
  repo: https://github.com/getian107/PRScs
- name: PLINK
  role: LD-clumping (r2=0.99) and per-gene genotype extraction from the binary LD
    reference panel
- name: fdrtool
  role: Pseudo-validation implemented within lassosum for selecting the regularization
    parameter
provenance:
  source_task_ids:
  - task_001
  - task_003
  - task_005
  - task_006
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-36862-w
    title: 'OTTERS: a powerful TWAS framework leveraging summary-level reference data'
  source_content_hash: sha256:8bdbd8c33aa05e8c3daf5b2dd62fbcc4cfc411e2cdd0db6604e0800a80702b85
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/daiqile96/OTTERS.git
  tools:
  - OTTERS
  - PLINK
  - PRS-CS
  - SDPR
  - fdrtool
  - lassosum
  - numpy
  - pandas
  - scipy
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# eqtl-weight-training-from-summary-statistics

## Summary

TWAS Stage I skill for training GReX imputation models (eQTL weights) from summary-level eQTL reference data (e.g. eQTLGen, n=31,684) using four summary-data-based PRS methods — P+T with LD clumping, lassosum, SDPR, and PRS-CS — instead of individual-level reference expression data.

## When to use

Use when eQTL summary statistics (CHROM, POS, A1, A2, Zscore, TargetID, N) plus an LD reference panel are available but individual-level genotype/expression data are not, and gene-level eQTL weights are needed for TWAS.

## When NOT to use

- Individual-level reference genotype and expression data are available and desired — the article benchmarks FUSION trained on individual-level data as the alternative baseline
- Trans-eQTL effects are required — OTTERS currently considers only cis-eQTL effects
- Reference data are from a non-European population — available eQTL summary statistics are mainly European, and cross-population TWAS is listed as ongoing work

## Inputs

- eQTL summary statistics file with columns CHROM, POS, A1, A2, Zscore, TargetID, N (sorted by chromosome then ascending SNP position), e.g. eQTLGen (n=31,684)
- Gene annotation file (CHROM, GeneStart, GeneEnd, TargetID)
- LD reference panel in PLINK binary (bed/bim/fam) format, e.g. GTEx V6 panel
- lassosum LD block definitions (EUR.hg38 default from Berisa & Pickrell 2015)
- PRScs LD reference panels (e.g. ldblk_1kg_eur hdf5 files)

## Outputs

- Trained eQTL weight files per gene for all five model variants (P0.001, P0.05, lassosum, SDPR, PRScs) written to the OTTERS Results directory
- Per-gene weight tables consumable by OTTERS Stage II (testing.py) for FUSION/S-PrediXcan-style TWAS

## How to apply

First sort eQTL summary statistics by chromosome then ascending SNP position and prepare a PLINK 1.9 binary LD reference panel plus a gene annotation file (CHROM, GeneStart, GeneEnd, TargetID). Run OTTERS Stage I (training.py) with --models=PT,lassosum,SDPR,PRScs and an LD-clumping threshold of --r2=0.99 for P+T; OTTERS integrates TABIX and PLINK to extract per-gene regions, invokes lassosum through R (with fdrtool pseudo-validation, using --lassosum_ld_blocks=EUR.hg38 LD blocks), runs SDPR (Linux x86, requiring MKL/GSL dynamic libraries on LD_LIBRARY_PATH), and runs PRS-CS with its external LD reference. The rationale is that eQTL effect sizes are SNP effect sizes from a reference 'GWAS' of gene expression, so PRS methods adapt directly, and the much larger summary-level reference (eQTLGen n=31,684 vs GTEx V6 n=338/338-scale) yields more valid models. Output is a set of trained eQTL weight files per method (P0.001, P0.05, lassosum, SDPR, PRScs) written to the Results directory per chromosome.

## Related tools

- **OTTERS** (Integrated Stage I driver: LD-clumping, input preparation, and orchestration of P+T, lassosum, SDPR, and PRS-CS to train eQTL weights) — https://github.com/daiqile96/OTTERS.git
- **lassosum** (Frequentist LASSO regression on summary statistics with LD from a reference panel, run via R with fdrtool pseudo-validation) — https://github.com/tshmak/lassosum
- **SDPR** (Summary-statistics-based Dirichlet Process Regression (Bayesian nonparametric) PRS method for eQTL weight estimation) — https://github.com/eldronzhou/SDPR.git
- **PRS-CS** (Bayesian continuous shrinkage prior MCMC method inferring posterior SNP effect sizes for eQTL weights from summary statistics and an external LD panel) — https://github.com/getian107/PRScs
- **PLINK** (LD-clumping (r2=0.99) and per-gene genotype extraction from the binary LD reference panel)
- **fdrtool** (Pseudo-validation implemented within lassosum for selecting the regularization parameter)

## Examples

```
python training.py --chr=4 --models=PT,lassosum,SDPR,PRScs --r2=0.99 --lassosum_ld_blocks=EUR.hg38
```

## Evaluation signals

- Results directory must contain trained weights for all five model variants (P0.001, P0.05, lassosum, SDPR, PRScs) for every annotated gene on the chromosome
- Evaluate trained weights on held-out individual-level samples by computing per-gene out-of-sample test R2; valid models defined as R2 > 0.01
- Reproduce Table 1 benchmarks on 315 GTEx V8 whole-blood samples: PRS-CS 10,337 valid genes (median R2 0.0517), P+T(0.001) 9,816, FUSION on GTEx V6 only 4,704
- Summary statistics must be sorted by chromosome then ascending SNP position and eQTLGen input must carry the CHROM, POS, A1, A2, Zscore, TargetID, N schema
- SDPR runs require MKL/GSL libraries on LD_LIBRARY_PATH and pinned MKL_NUM_THREADS/NUMEXPR_NUM_THREADS/OMP_NUM_THREADS; missing these causes library-load errors

## Limitations

- Training GReX imputation models by all five methods costs ~20 min per gene, making whole-genome (~20K genes) or multi-tissue training computationally challenging
- Currently available eQTL summary statistics are mainly from individuals of European descent, limiting cross-population transferability
- Only cis-eQTL effects are considered; trans-eQTL integration is blocked by computation time and data availability
- SDPR explicitly uses SSE instructions and has only been tested on Linux x86 with specific MKL/GSL library versions

## Evidence

- [abstract] adapts multiple polygenic risk score (PRS) methods to estimate eQTL weights from summary-level eQTL reference data: "adapts multiple polygenic risk score (PRS) methods to estimate eQTL weights from summary-level eQTL reference data"
- [intro] eQTL effect sizes are essentially SNP effect sizes: [evidence span withheld — anchor status: not_found]
- [intro] summary-based eQTLGen reference sample is 31,684: "the sample size of the summary-based eQTLGen reference sample is 31,684 for blood, whereas the sample size of the individual-level GTEx V6 reference is only 338"
- [methods] STAGE I command with r2=0.99 clumping: "# STAGE I
  # train eQTL weights using P+T, lassosum, SDPR and PRS-CS."
- [readme] OTTERS README integrated tool description: "perform TWAS Stage I: run P+T, lassosum, SDPR, and PRS-CS to train eQTL weights."
- [readme] lassosum LD blocks argument: "Added `--lassosum_ld_blocks` argument
  * Purpose: Select LD blocks for lassosum analysis"
- [readme] SDPR library requirements: [evidence span withheld — anchor status: external_doc]
