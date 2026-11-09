---
name: snp-gene-pair-interaction-testing
description: Use when you have RNA-seq expression counts, SNP genotype data, a SNP-to-gene
  map, and sample metadata (including patientID) and need to test whether genotype
  effects on gene expression differ by infection condition.
when_to_use_negative:
- You only need standard differential expression without SNP–gene interaction testing
  — plain kmFit (kimma_lm, kimma_lme, kimma_lmerel) is the appropriate entry point.
- No genotype data or SNP-to-gene map is available, so no eQTL model can be specified.
- Samples are unrelated and no kinship matrix exists; the kinship-covariance mixed
  model rationale does not apply (a simpler lm-style eQTL approach may suffice).
edam_operation: http://edamontology.org/operation_3226
edam_topics:
- http://edamontology.org/topic_3517
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0625
tools:
- name: kimma
  role: Provides kmFit_eQTL for fitting SNP–gene pair linear mixed models with genotype-by-condition
    interaction and kinship covariance, and summarise_kmFit for FDR-based significance
    calls
  repo: https://github.com/BIGslu/kimma
- name: R
  role: Runtime environment in which the kimma package is installed and executed
- name: devtools
  role: Used to install kimma from GitHub via install_github("BIGslu/kimma")
- name: lme4qtl
  role: Mixed-model machinery underlying kimma's kinship-covariance eQTL fits
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1093/bioinformatics/btad279
    title: 'Kimma: flexible linear mixed effects modeling with kinship covariance
      for RNA-seq data'
  source_content_hash: sha256:4b04cc0976666f97ffab73992111aedef2a6bdd7d9fef9cd954da155b28919fa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BIGslu/kimma
  tools:
  - R
  - devtools
  - kimma
  - lme4qtl
  edam_operation: http://edamontology.org/operation_3226
  edam_topics:
  - http://edamontology.org/topic_3517
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0625
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# snp-gene-pair-interaction-testing

## Summary

Test expression quantitative trait loci (eQTL) by fitting each SNP–gene pair with kimma's kmFit_eQTL as a linear mixed model containing a genotype-by-condition interaction term and a kinship covariance random effect, then calling significance at FDR < 0.05 via summarise_kmFit.

## When to use

Use when you have RNA-seq expression counts, SNP genotype data, a SNP-to-gene map, and sample metadata (including patientID) and need to test whether genotype effects on gene expression differ by infection condition.

## When NOT to use

- You only need standard differential expression without SNP–gene interaction testing — plain kmFit (kimma_lm, kimma_lme, kimma_lmerel) is the appropriate entry point.
- No genotype data or SNP-to-gene map is available, so no eQTL model can be specified.
- Samples are unrelated and no kinship matrix exists; the kinship-covariance mixed model rationale does not apply (a simpler lm-style eQTL approach may suffice).

## Inputs

- RNA-seq expression counts (e.g. example.count)
- Sample metadata (e.g. example.dat) including patientID and condition/infection variables
- Kinship covariance matrix (e.g. example.kin)
- SNP genotype data (dat_snp) keyed by genotypeID
- SNP-to-gene map (dat_map) linking genotypeID to geneID

## Outputs

- Per SNP–gene pair interaction effect sizes, P-values, and FDR values (e.g. compiled into eqtl_results.csv)
- FDR-based significance calls per SNP–gene pair at the chosen fdr_cutoff (FDR < 0.05)

## How to apply

Install kimma from GitHub with devtools (devtools::install_github("BIGslu/kimma")) and load the packaged example datasets example.dat, example.count, and example.kin. Prepare SNP genotypes as dat_snp plus a SNP-to-gene map dat_map linking each genotypeID to its geneID, both required by kmFit_eQTL. Call kmFit_eQTL with the expression counts, metadata, kinship matrix (kin), patientID, libraryID, dat_snp, and dat_map so each SNP–gene pair is fit with a linear mixed model including a genotype-by-condition interaction and a kinship covariance random effect (implemented via lme4qtl). Pass the fit through summarise_kmFit with an fdr_cutoff to derive per-pair significance calls, then compile interaction effect sizes, P-values, and FDR values into a single table such as eqtl_results.csv.

## Related tools

- **kimma** (Provides kmFit_eQTL for fitting SNP–gene pair linear mixed models with genotype-by-condition interaction and kinship covariance, and summarise_kmFit for FDR-based significance calls) — https://github.com/BIGslu/kimma
- **R** (Runtime environment in which the kimma package is installed and executed)
- **devtools** (Used to install kimma from GitHub via install_github("BIGslu/kimma"))
- **lme4qtl** (Mixed-model machinery underlying kimma's kinship-covariance eQTL fits)

## Examples

```
devtools::install_github("BIGslu/kimma"); fit <- kmFit_eQTL(dat_snp = dat_snp, dat_map = dat_map, dat = example.dat, counts = example.count, kin = example.kin, patientID = "patientID"); eqtl <- summarise_kmFit(fit, fdr_cutoff = 0.05)
```

## Evaluation signals

- eqtl_results.csv exists and contains effect-size, P-value, and FDR columns for every tested SNP–gene pair
- All FDR values fall within [0, 1]
- Significant calls are consistent with the applied fdr_cutoff (FDR < 0.05) in summarise_kmFit
- Every row maps one genotypeID to its geneID as defined in dat_map
- Fitted models include both the genotype-by-condition interaction term and the kinship random effect

## Limitations

- Significance depends on the chosen fdr_cutoff; the paper's convention is FDR < 0.05, and calls must match the cutoff actually applied
- kmFit_eQTL requires a dat_map linking every genotypeID to a geneID — pairs absent from the map cannot be tested
- kimma is designed for RNA-seq linear mixed modeling; performance claims are benchmarked against limma unpaired and dream paired models, not eQTL-specific tools

## Evidence

- [other] kmFit_eQTL models each SNP–gene pair as an expression quantitative trait locus using an interaction term between infection condition and genotype, with significance defined at FDR < 0.05.: [evidence span withheld — anchor status: not_found]
- [methods] kmFit_eQTL signature with SNP and gene inputs.: "kmFit_eQTL(dat_snp, dat_map, geneID, genotypeID, dat, kin, patientID, libraryID, counts"
- [methods] summarise_kmFit applies FDR-based significance cutoffs.: "summarise_kmFit(fdr, fdr_cutoff, p_cutoff, FCgroup, intercept, fdr.cutoff, p.cutoff)"
- [readme] Installation command from README.: "install.packages("devtools")
devtools::install_github("BIGslu/kimma")"
- [abstract] kimma supports covariance matrices as well as fit metrics like AIC.: "Unlike other software, kimma supports covariance matrices as well as fit metrics like Akaike information criterion (AIC)"
