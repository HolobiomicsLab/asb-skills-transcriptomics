---
name: gwas-summary-statistics-harmonization
description: Use when only summary-level eQTL statistics (e.g., eQTLGen), GWAS summary
  statistics (e.g., UK Biobank), and a PLINK LD reference are available and TWAS weights
  must be trained without individual-level reference expression/genotype data.
when_to_use_negative:
- Individual-level reference genotype and expression data are available — the article
  positions OTTERS against individual-level tools like PrediXcan/FUSION/TIGAR, which
  may be preferable (e.g., the smaller GTEx V6 whole-blood reference, n=338)
- Trans-eQTL effects must be modeled — OTTERS 'only considers cis-eQTL effects'
- Non-European ancestry summary statistics are the sole input — available eQTL summary
  data are mainly of European descent, limiting transferability
edam_topics:
- http://edamontology.org/topic_3423
tools:
- name: OTTERS
  role: Integrated harness that performs LD-clumping, prepares LD reference and eQTL
    summary statistics in the required formats for all four PRS methods, and runs
    TWAS Stages I and II
  repo: https://github.com/daiqile96/OTTERS
- name: PLINK 1.9
  role: Extracts LD reference genotypes for each target gene region (call_PLINK_extract)
    and performs LD clumping
  repo: https://www.cog-genomics.org/plink
- name: TABIX
  role: Extracts per-gene VCF regions from indexed summary/genotype files (extract_vcf_region_with_tabix)
  repo: http://www.htslib.org/doc/tabix.html
- name: lassosum
  role: Frequentist LASSO PRS method trained on the harmonized eQTL summary statistics;
    its pseudo-validation and LD block boundaries (EUR.hg38 default) are invoked during
    preparation
  repo: https://github.com/tshmak/lassosum
- name: PRS-CS
  role: Bayesian continuous-shrinkage PRS method trained on the harmonized eQTL summary
    statistics and LD reference panel
  repo: https://github.com/getian107/PRScs
- name: SDPR
  role: Summary-statistics-based Dirichlet Process Regression PRS method trained on
    the harmonized eQTL summary statistics
  repo: https://github.com/eldronzhou/SDPR
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-36862-w
    title: 'OTTERS: a powerful TWAS framework leveraging summary-level reference data'
  source_content_hash: sha256:f8df55d31111d8c87197bf9ef362bc28babc2e2ca31a304bf5eb56f93c5308be
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/daiqile96/OTTERS
  tools:
  - OTTERS
  - PLINK 1.9
  - PRS-CS
  - SDPR
  - TABIX
  - lassosum
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3423
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# gwas-summary-statistics-harmonization

## Summary

Prepare summary-level eQTL/GWAS statistics, an LD reference panel in PLINK binary format, and a gene annotation file into the input formats required by OTTERS' four PRS-based eQTL weight trainers (P+T, lassosum, SDPR, PRS-CS), so TWAS Stage I and II can run on summary data alone.

## When to use

Use when only summary-level eQTL statistics (e.g., eQTLGen), GWAS summary statistics (e.g., UK Biobank), and a PLINK LD reference are available and TWAS weights must be trained without individual-level reference expression/genotype data.

## When NOT to use

- Individual-level reference genotype and expression data are available — the article positions OTTERS against individual-level tools like PrediXcan/FUSION/TIGAR, which may be preferable (e.g., the smaller GTEx V6 whole-blood reference, n=338)
- Trans-eQTL effects must be modeled — OTTERS 'only considers cis-eQTL effects'
- Non-European ancestry summary statistics are the sole input — available eQTL summary data are mainly of European descent, limiting transferability

## Inputs

- Gene annotation file (columns: CHROM, GeneStart, GeneEnd, TargetID)
- eQTL reference summary statistics (e.g., eQTLGen), sorted by chromosome then ascending SNP position
- GWAS summary statistics (e.g., UK Biobank cardiovascular disease, n=459,324)
- LD reference panel genotypes in PLINK binary format (.bed/.bim/.fam)
- LD block boundaries for lassosum (Berisa & Pickrell EUR/ASN/AFR, hg19 or hg38)

## Outputs

- Per-gene LD reference and eQTL summary statistic subsets formatted for P+T, lassosum, SDPR, and PRS-CS
- Trained eQTL weight sets (P0.001, P0.05, lassosum, SDPR, PRScs) ready for TWAS Stage II gene-based tests

## How to apply

Assemble four OTTERS inputs: a gene annotation file with CHROM, GeneStart, GeneEnd, TargetID columns; eQTL summary statistics (e.g., eQTLGen, n=31,684 for blood) sorted by chromosome then ascending SNP position; GWAS summary statistics (e.g., UKBB cardiovascular disease, n=459,324); and LD reference genotypes in PLINK binary format. Use OTTERS' built-in LD-clumping (threshold --r2=0.99) and its integration of TABIX and PLINK 1.9 (via extract_vcf_region_with_tabix and call_PLINK_extract in ottersutils.py) to extract each target gene's region from the VCF/genotype data efficiently. Select lassosum LD blocks with --lassosum_ld_blocks (options EUR.hg19/hg38, ASN, AFR; default EUR.hg38) and run Stage I with training.py --models=PT,lassosum,SDPR,PRScs so the same harmonized inputs feed all four PRS methods. The rationale is that eQTL effect sizes are essentially SNP effect sizes from a reference 'GWAS' of gene expression, so each PRS method can be repurposed to estimate eQTL weights from harmonized summary statistics.

## Related tools

- **OTTERS** (Integrated harness that performs LD-clumping, prepares LD reference and eQTL summary statistics in the required formats for all four PRS methods, and runs TWAS Stages I and II) — https://github.com/daiqile96/OTTERS
- **PLINK 1.9** (Extracts LD reference genotypes for each target gene region (call_PLINK_extract) and performs LD clumping) — https://www.cog-genomics.org/plink
- **TABIX** (Extracts per-gene VCF regions from indexed summary/genotype files (extract_vcf_region_with_tabix)) — http://www.htslib.org/doc/tabix.html
- **lassosum** (Frequentist LASSO PRS method trained on the harmonized eQTL summary statistics; its pseudo-validation and LD block boundaries (EUR.hg38 default) are invoked during preparation) — https://github.com/tshmak/lassosum
- **PRS-CS** (Bayesian continuous-shrinkage PRS method trained on the harmonized eQTL summary statistics and LD reference panel) — https://github.com/getian107/PRScs
- **SDPR** (Summary-statistics-based Dirichlet Process Regression PRS method trained on the harmonized eQTL summary statistics) — https://github.com/eldronzhou/SDPR

## Examples

```
python training.py --models=PT,lassosum,SDPR,PRScs --r2=0.99 --lassosum_ld_blocks=EUR.hg38
```

## Evaluation signals

- eQTL summary statistics are sorted by chromosome then ascending SNP position before Stage I
- LD reference panel ancestry/format matches the GWAS and each PRS method runs without error (lassosum expects PLINK 1 bed format; PRS-CS expects snpinfo + hdf5 LD blocks)
- Per-gene extraction via tabix/PLINK returns the gene's annotated CHROM/GeneStart/GeneEnd interval for every TargetID
- All five weight sets (P0.001, P0.05, lassosum, SDPR, PRScs) are produced per gene; missing SDPR/PRScs outputs indicate format mismatch
- Applied to eQTLGen + UKBB cardiovascular disease, the downstream TWAS should recover 40 significant and 38 independently significant genes with top gene RP11-378A13.1 at p=9.78e-09

## Limitations

- Training GReX imputation models by all methods costs ~20 min per gene across the five models, which is challenging for multiple tissues and ~20K genome-wide genes
- Available eQTL summary statistics are mainly from individuals of European descent, limiting cross-population transferability
- LD reference panels must match the GWAS sample's ancestry (e.g., PRS-CS EUR/UKBB panels), and lassosum LD blocks must be selected for the correct genome build (hg19 vs hg38)

## Evidence

- [readme] OTTERS integrates TABIX/PLINK per-gene extraction for input preparation: "integrate [TABIX](http://www.htslib.org/doc/tabix.html) and [PLINK 1.9](https://www.cog-genomics.org/plink) tools to extract input data per target gene more efficiently"
- [readme] OTTERS prepares LD reference and eQTL summary statistics in required PRS-method formats: "prepare the required inputs (LD reference and eQTL summary statistics) in the required formats for P+T, lassosum, SDPR, and PRS-CS"
- [intro] Rationale for adapting PRS methods to eQTL summary statistics: [evidence span withheld — anchor status: not_found]
- [methods] lassosum pseudo-validation is part of the preparation/training pipeline: "to perform pseudo-validation implemented in lassosum."
- [discussion] Ancestry limitation of available eQTL summary statistics: "the currently available eQTL summary statistics are mainly derived from individuals of European descent"
