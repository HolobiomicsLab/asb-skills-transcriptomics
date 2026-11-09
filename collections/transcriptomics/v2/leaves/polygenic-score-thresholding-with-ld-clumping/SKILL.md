---
name: polygenic-score-thresholding-with-ld-clumping
description: Use when only summary-level eQTL reference data (e.g., eQTLGen-style
  p-values + beta coefficients per gene) are available and individual-level genotype/expression
  cannot be used to fit GReX imputation weights.
when_to_use_negative:
- Gene expression genetic architecture is dense — the paper reports P+T performs better
  for sparse architecture, whereas Bayesian methods (PRS-CS, SDPR) perform better
  for denser architecture
- Only individual-level reference genotype/expression data are available and no summary-level
  eQTL statistics exist to clump
- Reference and GWAS populations are not of European descent — available eQTL summary
  statistics and the shipped LD blocks (EUR.hg19/hg38) are mainly European, limiting
  cross-population transferability
edam_operation: http://edamontology.org/operation_3731
edam_topics:
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_3308
tools:
- name: OTTERS
  role: integrated framework that clumps LD, formats P+T inputs, and runs TWAS Stages
    I and II for the P+T-trained eQTL weights
  repo: https://github.com/daiqile96/OTTERS
- name: PLINK
  role: performs LD clumping and cis-region SNP extraction for each target gene
- name: lassosum
  role: sibling summary-data method integrated alongside P+T in OTTERS; shares the
    LD block preparation (--lassosum_ld_blocks)
  repo: https://github.com/tshmak/lassosum
- name: PRS-CS
  role: Bayesian alternative to P+T in the omnibus ACAT-O combination
  repo: https://github.com/getian107/PRScs
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-023-36862-w
    title: 'OTTERS: a powerful TWAS framework leveraging summary-level reference data'
  source_content_hash: sha256:2d2583f5a6007f16095163408dd01e032984f26d084f37c91fa63ca881cab8fb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/daiqile96/OTTERS
  tools:
  - OTTERS
  - PLINK
  - PRS-CS
  - SDPR
  - lassosum
  - numpy
  - pandas
  - scipy
  edam_operation: http://edamontology.org/operation_3731
  edam_topics:
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7857142857142857
n_claims: 14
---

# polygenic-score-thresholding-with-ld-clumping

## Summary

Adapts P+T (p value thresholding with LD clumping), a classic polygenic risk score method, to train cis-eQTL weights from summary-level eQTL statistics for TWAS Stage I, treating eQTL summary data as a reference 'GWAS' of gene expression.

## When to use

Use when only summary-level eQTL reference data (e.g., eQTLGen-style p-values + beta coefficients per gene) are available and individual-level genotype/expression cannot be used to fit GReX imputation weights.

## When NOT to use

- Gene expression genetic architecture is dense — the paper reports P+T performs better for sparse architecture, whereas Bayesian methods (PRS-CS, SDPR) perform better for denser architecture
- Only individual-level reference genotype/expression data are available and no summary-level eQTL statistics exist to clump
- Reference and GWAS populations are not of European descent — available eQTL summary statistics and the shipped LD blocks (EUR.hg19/hg38) are mainly European, limiting cross-population transferability

## Inputs

- summary-level eQTL statistics per gene (two-sided p-values, beta coefficients, SNP positions) converted to Z-scores
- LD reference panel / LD blocks (e.g., Berisa & Pickrell 2015 EUR.hg38 blocks) in PLINK 1.9-compatible format
- cis-eQTL region extraction inputs (VCF/genotype panel sliced with TABIX/PLINK per target gene)

## Outputs

- trained P+T eQTL weights per gene (TWAS Stage I weight file)
- gene-based TWAS p-values from Stage II association testing with P+T weights (feeds the ACAT-O omnibus)

## How to apply

Convert two-sided eQTL p-values and beta coefficients to Z-scores as Z = sign(beta) * abs(norm.ppf(p/2.0)) with scipy.stats.norm, then LD-clump the per-gene summary statistics with PLINK 1.9 against an LD reference (in OTTERS the simulation protocol used a loose clumping threshold of --r2=0.99). Construct P+T eQTL weights from the clumped, significance-thresholded SNPs; OTTERS integrates TABIX and PLINK 1.9 to extract the cis region per target gene and formats the LD reference and eQTL summary statistics as required for P+T before running Stage I via training.py. Run TWAS Stage II (testing.py) with the P+T weights (model key P0.001/P0.05 style thresholds) and aggregate the resulting per-method TWAS p-values via ACAT-O in the omnibus OTTERS test. Rationale: P+T performs better under sparse genetic architecture, so it complements Bayesian methods (SDPR, PRS-CS) that favor denser architecture, and the omnibus combination is near-optimal across scenarios.

## Related tools

- **OTTERS** (integrated framework that clumps LD, formats P+T inputs, and runs TWAS Stages I and II for the P+T-trained eQTL weights) — https://github.com/daiqile96/OTTERS
- **PLINK** (performs LD clumping and cis-region SNP extraction for each target gene)
- **lassosum** (sibling summary-data method integrated alongside P+T in OTTERS; shares the LD block preparation (--lassosum_ld_blocks)) — https://github.com/tshmak/lassosum
- **PRS-CS** (Bayesian alternative to P+T in the omnibus ACAT-O combination) — https://github.com/getian107/PRScs

## Examples

```
python training.py --models=PT --r2=0.99 --lassosum_ld_blocks=EUR.hg38
```

## Evaluation signals

- After clumping at --r2=0.99, no retained SNP pair within the gene's cis region exceeds the LD r2 threshold
- P+T weights are produced for the same genes as the lassosum/SDPR/PRScs weights and can be supplied to testing.py as the PT/P0.001 model key
- Out-of-sample test R2 for P+T weights is highest in simulated sparse-architecture scenarios (small p_causal) relative to Bayesian methods
- Omnibus OTTERS (ACAT-O over P+T, lassosum, SDPR, PRS-CS) achieves at least near-optimal power versus each individual method's TWAS p-value

## Limitations

- P+T relies on a chosen significance threshold and LD r2 cutoff; performance is architecture-dependent (better sparse, worse dense)
- OTTERS training all five models costs ~20 min per gene, which is computationally challenging for ~20K genes across multiple tissues
- OTTERS considers only cis-eQTL effects and cannot give the direction of identified gene-phenotype associations; check per-method TWAS Z-score signs instead

## Evidence

- [intro] P+T summary-based method: "we adapt four representative summary-data-based PRS methods—p value thresholding with linkage disequilibrium (LD) clumping (P+T)26, frequentist LASSO32 regression-based method lassosum27"
- [discussion] architecture-dependent performance of P+T: "P+T methods generally performing better for sparse architecture, whereas the Bayesian methods performing better for denser architecture"
- [readme] OTTERS prepares P+T inputs including LD reference and eQTL summary statistics: "prepare the required inputs (LD reference and eQTL summary statistics) in the required formats for P+T, lassosum, SDPR, and PRS-CS"
- [readme] LD blocks selection options: "Purpose: Select LD blocks for lassosum analysis"
