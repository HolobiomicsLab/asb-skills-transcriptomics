---
name: eqtl-mapping-genotype-expression
description: Use when you have SNP genotype data plus RNA-seq counts from (possibly
  related) individuals across conditions (e.g. infection), and need per-SNP–gene eQTL
  effect sizes, P-values, and FDR calls rather than plain DEG calls.
when_to_use_negative:
- Your goal is differential expression between conditions without genotype testing
  — kimma's kmFit, or limma/dream comparisons, cover that instead
- SNPs cannot be mapped to genes (no dat_map linking genotypeID to geneID), so per-pair
  eQTL models cannot be constructed
- Samples are unrelated and no kinship matrix is available, so the kinship-covariance
  random effect central to this model is not estimable
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0625
- http://edamontology.org/topic_3308
tools:
- name: kimma
  role: Provides kmFit_eQTL and summarise_kmFit, which fit each SNP–gene pair with
    a genotype-by-condition interaction LMM and apply the FDR cutoff
  repo: https://github.com/BIGslu/kimma
- name: devtools
  role: Installs the kimma package from GitHub before running the eQTL workflow
- name: lme4qtl
  role: Underlying linear mixed model machinery for kinship-covariance random effects
    used by kimma
- name: R
  role: Runtime environment for kimma, the example datasets, and assembling eqtl_results.csv
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad279
    title: 'Kimma: flexible linear mixed effects modeling with kinship covariance
      for RNA-seq data'
  source_content_hash: sha256:3cb5d4fd696cf845d89147c6c821c71447c73f86b9676237d0e6717bc645d6ee
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BIGslu/kimma
  tools:
  - R
  - devtools
  - kimma
  - lme4qtl
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0625
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# eqtl-mapping-genotype-expression

## Summary

Maps expression quantitative trait loci by fitting each SNP–gene pair with kimma's kmFit_eQTL linear mixed model that includes a genotype-by-infection-condition interaction term and a kinship covariance random effect, then calling pairs significant at FDR < 0.05.

## When to use

Use when you have SNP genotype data plus RNA-seq counts from (possibly related) individuals across conditions (e.g. infection), and need per-SNP–gene eQTL effect sizes, P-values, and FDR calls rather than plain DEG calls.

## When NOT to use

- Your goal is differential expression between conditions without genotype testing — kimma's kmFit, or limma/dream comparisons, cover that instead
- SNPs cannot be mapped to genes (no dat_map linking genotypeID to geneID), so per-pair eQTL models cannot be constructed
- Samples are unrelated and no kinship matrix is available, so the kinship-covariance random effect central to this model is not estimable

## Inputs

- Gene expression count matrix (e.g. example.count from the kimma package)
- Sample metadata with patientID and condition variables (e.g. example.dat)
- Kinship covariance matrix for related individuals (e.g. example.kin)
- SNP genotype data structured as dat_snp with genotypeID identifiers
- SNP-to-gene map dat_map linking each genotypeID to its geneID

## Outputs

- eqtl_results.csv containing per SNP–gene pair effect sizes, P-values, and FDR values
- Significant eQTL calls defined at FDR < 0.05 via the summarise_kmFit fdr_cutoff

## How to apply

Install kimma from GitHub with devtools (devtools::install_github("BIGslu/kimma")) and load the package example datasets example.dat, example.count, and example.kin. Prepare SNP genotype data as dat_snp together with a SNP-to-gene map dat_map that links each genotypeID to its geneID, which restricts testing to mapped SNP–gene pairs. Call kmFit_eQTL with the expression counts, metadata (dat), kinship matrix (kin), patientID, libraryID, dat_snp, and dat_map so each pair is fit with a linear mixed model containing a genotype-by-condition interaction term and the kinship covariance random effect. Pass the fit through summarise_kmFit with an fdr_cutoff to derive FDR-based significance calls per SNP–gene pair, then compile effect sizes, P-values, and FDR values into eqtl_results.csv.

## Related tools

- **kimma** (Provides kmFit_eQTL and summarise_kmFit, which fit each SNP–gene pair with a genotype-by-condition interaction LMM and apply the FDR cutoff) — https://github.com/BIGslu/kimma
- **devtools** (Installs the kimma package from GitHub before running the eQTL workflow)
- **lme4qtl** (Underlying linear mixed model machinery for kinship-covariance random effects used by kimma)
- **R** (Runtime environment for kimma, the example datasets, and assembling eqtl_results.csv)

## Examples

```
fit <- kmFit_eQTL(dat = example.dat, counts = example.count, kin = example.kin, dat_snp = dat_snp, dat_map = dat_map, patientID = "patientID")
```

## Evaluation signals

- eqtl_results.csv exists and contains effect-size, P-value, and FDR columns for every tested SNP–gene pair
- All FDR values fall within [0,1]
- Significant calls are consistent with the applied fdr_cutoff (FDR < 0.05 by default)
- Every genotypeID in dat_snp maps to a valid geneID in dat_map, and no pair is fit without the kinship random effect
- Fit results include the genotype-by-condition interaction term alongside the kinship covariance, consistent with kmFit_eQTL's model

## Limitations

- Testing is restricted to SNP–gene pairs enumerated in dat_map; pairs absent from the map are never modeled
- The kinship covariance random effect presumes related individuals; with unrelated samples this design term is inappropriate
- The repository README is minimal and points to the vignette (https://bigslu.github.io/kimma_vignette/kimma_vignette.html) for full usage details

## Evidence

- [methods] kmFit_eQTL function signature with dat_snp and dat_map inputs: "kmFit_eQTL(dat_snp, dat_map, geneID, genotypeID, dat, kin, patientID, libraryID, counts"
- [methods] summarise_kmFit applies FDR and P cutoffs: "summarise_kmFit(fdr, fdr_cutoff, p_cutoff, FCgroup, intercept, fdr.cutoff, p.cutoff)"
- [readme] kimma models RNA-seq with covariance matrices and fit metrics: "an open-source R package for flexible linear mixed effects modeling of RNA-seq including covariates, weights, random effects, covariance matrices, and fit metrics"
- [abstract] kimma supports covariance matrices unlike other software: "Unlike other software, kimma supports covariance matrices as well as fit metrics like Akaike information criterion (AIC)"
- [intro] Installation of kimma from GitHub via devtools: "install.packages("devtools")
devtools::install_github("BIGslu/kimma")"
