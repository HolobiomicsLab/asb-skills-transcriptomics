---
name: prs-method-weight-estimation
description: Use when only summary-level eQTL reference statistics (e.g., eQTLGen,
  n=31,684) are available and per-gene eQTL weights must be estimated for TWAS Stage
  I using P+T, lassosum, SDPR, and PRS-CS.
when_to_use_negative:
- Individual-level reference expression and genotype data are available and a tool
  like FUSION or PrediXcan is preferred — although OTTERS/PRS-based training outperformed
  FUSION across simulations, the skill is designed for summary-level reference data
- The target tissue has no summary-level eQTL reference statistics (e.g., no eQTLGen/
  CommonMind equivalent)
- Trans-eQTL effects must be modeled — OTTERS only considers cis-eQTL effects
edam_operation: http://edamontology.org/operation_2424
edam_topics:
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0622
tools:
- name: OTTERS
  role: Integrated framework orchestrating input prep, per-gene extraction, and all
    four PRS-based weight training runs (STAGE I, training.py)
  repo: github.com/daiqile96/OTTERS
- name: lassosum
  role: Frequentist LASSO regression on summary statistics with LD reference and pseudo-validation,
    called as R package by OTTERS
  repo: github.com/tshmak/lassosum
- name: SDPR
  role: Summary-statistics-based Dirichlet Process Regression Bayesian weight training
  repo: github.com/eldronzhou/SDPR
- name: PRS-CS
  role: Bayesian continuous-shrinkage MCMC weight training integrated by OTTERS
  repo: github.com/getian107/PRScs
- name: PLINK 1.9
  role: Per-gene genotype extraction from the LD reference panel (call_PLINK_extract)
- name: TABIX
  role: Efficient per-gene region extraction from tabixed eQTL summary files (extract_vcf_region_with_tabix)
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-023-36862-w
    title: 'OTTERS: a powerful TWAS framework leveraging summary-level reference data'
  source_content_hash: sha256:0c84d9a2a5c0d0e20e2fac3f18abfb8d9a3908355dcbedad4cad26b9605d7865
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - OTTERS
  - PLINK 1.9
  - PRS-CS
  - SDPR
  - TABIX
  - lassosum
  edam_operation: http://edamontology.org/operation_2424
  edam_topics:
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 10
---

# prs-method-weight-estimation

## Summary

Train GReX/eQTL imputation weights (TWAS Stage I) from summary-level reference eQTL data by adapting four PRS methods — P+T with LD clumping, lassosum, SDPR, and PRS-CS — inside OTTERS, instead of individual-level reference panels such as GTEx.

## When to use

Use when only summary-level eQTL reference statistics (e.g., eQTLGen, n=31,684) are available and per-gene eQTL weights must be estimated for TWAS Stage I using P+T, lassosum, SDPR, and PRS-CS.

## When NOT to use

- Individual-level reference expression and genotype data are available and a tool like FUSION or PrediXcan is preferred — although OTTERS/PRS-based training outperformed FUSION across simulations, the skill is designed for summary-level reference data
- The target tissue has no summary-level eQTL reference statistics (e.g., no eQTLGen/ CommonMind equivalent)
- Trans-eQTL effects must be modeled — OTTERS only considers cis-eQTL effects

## Inputs

- Gene annotation file with columns CHROM, GeneStart, GeneEnd, TargetID
- Summary-level eQTL reference statistics (e.g., eQTLGen whole blood, n=31,684), sorted by chromosome then ascending SNP position
- LD reference panel genotypes in PLINK binary (.bed/.bim/.fam) format
- LD block definitions for lassosum (Berisa & Pickrell EUR.hg38 etc.)
- GWAS summary statistics (for downstream Stage II testing)

## Outputs

- Trained eQTL weights per gene per method (P0.001, P0.05, lassosum, SDPR, PRScs)
- Per-method weight files consumable by OTTERS STAGE II testing.py for gene-based TWAS
- Lassosum pseudo-validation selection results per gene

## How to apply

Prepare a gene annotation file (CHROM, GeneStart, GeneEnd, TargetID), sort the eQTL summary statistics by chromosome then ascending SNP position, and provide an LD reference panel in PLINK binary format. Run OTTERS STAGE I via training.py with --models=PT,lassosum,SDPR,PRScs, LD-clumping threshold --r2=0.99, and --lassosum_ld_blocks=EUR.hg38 (options EUR.hg19/ASN.hg19/AFR.hg19 and hg38 equivalents; default EUR.hg38). OTTERS integrates TABIX and PLINK 1.9 to extract per-gene input regions (via extract_vcf_region_with_tabix and call_PLINK_extract), runs lassosum with pseudo-validation, and supports parallel computation across genes. The rationale is that eQTL effect sizes are SNP effect sizes from a reference 'GWAS' of gene expression, so PRS methods can be adapted directly; P+T suits sparse genetic architecture while lassosum/SDPR/PRS-CS cover denser architectures, and the resulting per-method weights feed downstream TWAS tests aggregated by ACAT-O.

## Related tools

- **OTTERS** (Integrated framework orchestrating input prep, per-gene extraction, and all four PRS-based weight training runs (STAGE I, training.py)) — github.com/daiqile96/OTTERS
- **lassosum** (Frequentist LASSO regression on summary statistics with LD reference and pseudo-validation, called as R package by OTTERS) — github.com/tshmak/lassosum
- **SDPR** (Summary-statistics-based Dirichlet Process Regression Bayesian weight training) — github.com/eldronzhou/SDPR
- **PRS-CS** (Bayesian continuous-shrinkage MCMC weight training integrated by OTTERS) — github.com/getian107/PRScs
- **PLINK 1.9** (Per-gene genotype extraction from the LD reference panel (call_PLINK_extract))
- **TABIX** (Efficient per-gene region extraction from tabixed eQTL summary files (extract_vcf_region_with_tabix))

## Examples

```
git clone https://github.com/daiqile96/OTTERS.git; python training.py --gene_anno=genes.txt --eqtl=eqtlgen_sorted.txt --ldref=plink_ref --models=PT,lassosum,SDPR,PRScs --r2=0.99 --lassosum_ld_blocks=EUR.hg38
```

## Evaluation signals

- All input eQTL summary statistics are correctly sorted by chromosome then ascending SNP position before Stage I
- Weights are produced for all five variants (P0.001, P0.05, lassosum, SDPR, PRScs) for each annotated gene
- Reproducing the eQTLGen + UKBB cardiovascular disease application yields 40 significant and 38 independently significant TWAS genes (Table 2), with RP11-378A13.1 at p=9.78e-09
- Lassosum pseudo-validation reports a chosen parameter per gene rather than defaulting
- Training completes in ~20 min per gene for all five models, so large deviations indicate a configuration problem

## Limitations

- Training GReX imputation models by all individual methods costs ~20 min per gene, which may be computationally challenging for multiple tissues and ~20K genome-wide genes
- Currently available summary-level eQTL references are mainly from individuals of European descent, limiting cross-population transferability
- Only cis-eQTL effects are considered; trans-eQTL incorporation is blocked by computation time and lack of summary-level trans-eQTL reference data

## Evidence

- [intro] Adapt PRS methods for Stage I weight estimation: [evidence span withheld — anchor status: not_found]
- [methods] STAGE I command: "# STAGE I
  # train eQTL weights using P+T, lassosum, SDPR and PRS-CS."
- [intro] Sample size rationale: "the sample size of the summary-based eQTLGen reference sample is 31,684 for blood, whereas the sample size of the individual-level GTEx V6 reference is only 338 for the same tissue"
- [methods] Lassosum pseudo-validation: "to perform pseudo-validation implemented in lassosum."
- [readme] OTTERS integrations: "integrate [TABIX](http://www.htslib.org/doc/tabix.html) and [PLINK 1.9](https://www.cog-genomics.org/plink) tools to extract input data per target gene more efficiently"
- [discussion] Architecture-dependent performance: "P+T methods generally performing better for sparse architecture, whereas the Bayesian methods performing better for denser architecture"
