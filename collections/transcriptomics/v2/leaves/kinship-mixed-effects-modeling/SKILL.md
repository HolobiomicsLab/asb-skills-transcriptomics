---
name: kinship-mixed-effects-modeling
description: Use when modeling RNA-seq counts from paired-sample designs of genetically
  related individuals and you need to incorporate pairwise kinship covariance, assess
  its impact on model fit (delta AIC), or on DEG detection at FDR < 0.05.
when_to_use_negative:
- Samples are unrelated or no pairwise kinship matrix is available — use kimma_lm/kimma_lme
  fits without run_lmerel
- The goal is SNP–gene eQTL interaction testing — that is handled by kmFit_eQTL with
  dat_snp and dat_map rather than the kinship random-effect workflow
- Unpaired single-time-point designs where a simple limma-style lm fit suffices and
  kinship covariance adds no modeling information
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: kimma
  role: R package providing kmFit, kimma_cleaning, kimma_lmerel, and summarise_kmFit
    to fit kinship-covariance linear mixed models on RNA-seq
  repo: https://github.com/BIGslu/kimma
- name: lme4qtl
  role: underlying engine whose relmatLmer function executes the kinship-capable mixed
    model fits
- name: R
  role: execution environment for the kimma package and downstream AIC/FDR computation
- name: devtools
  role: used to install kimma from GitHub (devtools::install_github)
provenance:
  source_task_ids:
  - task_003
  - task_005
  - task_004
  source_papers:
  - doi: 10.1093/bioinformatics/btad279
    title: 'Kimma: flexible linear mixed effects modeling with kinship covariance
      for RNA-seq data'
  source_content_hash: sha256:75a1341c3c1e47efa3964881420d8d03e95fd57995b21074cbbea8e852b409c5
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BIGslu/kimma
  tools:
  - R
  - devtools
  - dplyr
  - kimma
  - lme4qtl
  - tibble
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# kinship-mixed-effects-modeling

## Summary

Fit linear mixed effects models for RNA-seq data that include a pairwise genetic kinship covariance matrix as a random effect, using the kimma R package (which delegates kinship-capable fits to lme4qtl::relmatLmer), and compare kinship vs. non-kinship model fit per gene via AIC.

## When to use

Use when modeling RNA-seq counts from paired-sample designs of genetically related individuals and you need to incorporate pairwise kinship covariance, assess its impact on model fit (delta AIC), or on DEG detection at FDR < 0.05.

## When NOT to use

- Samples are unrelated or no pairwise kinship matrix is available — use kimma_lm/kimma_lme fits without run_lmerel
- The goal is SNP–gene eQTL interaction testing — that is handled by kmFit_eQTL with dat_snp and dat_map rather than the kinship random-effect workflow
- Unpaired single-time-point designs where a simple limma-style lm fit suffices and kinship covariance adds no modeling information

## Inputs

- gene-by-sample count matrix (e.g. kimma's example.count)
- sample metadata table with patientID and libraryID (e.g. example.dat)
- pairwise genetic kinship matrix (e.g. example.kin)
- optional voom weights (example.voom / example.seasnake outputs)
- model formula string such as 'lme' with patientID random effect

## Outputs

- per-gene AIC table with AIC_lm/lme, AIC_lmerel, and delta AIC (non-kinship minus kinship)
- per-gene results table with estimates, P-values, and FDR columns for kinship and non-kinship fits
- set-comparison counts of DEGs newly detected with kinship, only without kinship, and percentage agreement of calls

## How to apply

Load kimma's packaged example data (example.count gene counts, example.dat sample metadata, example.kin pairwise kinship matrix) and run kimma_cleaning to map patientID, libraryID, counts, meta, genes, and kin into the expected structures. Call kmFit with model='lme', use_weights=TRUE, metrics=TRUE, and both run_lm=TRUE and run_lmerel=TRUE, passing kin=example.kin so each gene is fit without (lm/lme) and with (lmerel, a kinship covariance random effect executed via lme4qtl::relmatLmer) relatedness. Extract per-gene AIC from both fits and compute delta AIC (non-kinship minus kinship); delta AIC > 2 indicates better fit with kinship. Pass contrast results through summarise_kmFit with fdr.cutoff (e.g. FDR < 0.05) and p.cutoff to derive per-gene significance calls, then compare DEG sets between models to quantify newly detected DEGs and percentage agreement.

## Related tools

- **kimma** (R package providing kmFit, kimma_cleaning, kimma_lmerel, and summarise_kmFit to fit kinship-covariance linear mixed models on RNA-seq) — https://github.com/BIGslu/kimma
- **lme4qtl** (underlying engine whose relmatLmer function executes the kinship-capable mixed model fits)
- **R** (execution environment for the kimma package and downstream AIC/FDR computation)
- **devtools** (used to install kimma from GitHub (devtools::install_github))

## Examples

```
kmFit(dat=example.dat, counts=example.count, kin=example.kin, patientID='patientID', libraryID='libraryID', model='lme', run_lm=TRUE, run_lmerel=TRUE, use_weights=TRUE, metrics=TRUE)
```

## Evaluation signals

- Both output tables contain exactly one row per modeled gene
- Every gene has AIC values for both model forms (lm/lme and lmerel), enabling a per-gene delta AIC computation
- FDR columns are populated and all FDR values fall in [0,1], consistent with the applied fdr.cutoff (e.g. 0.05)
- Genes with delta AIC > 2 identify better fit with kinship, mirroring the paper's model-fit comparison
- DEG-set comparison quantifies newly detected DEGs in the kinship model at FDR < 0.05 (the paper reports 25 new DEGs in a related cohort) and a percentage agreement of calls between models

## Limitations

- The kinship random effect requires a pairwise kinship matrix and identifiable patientID groups; it only impacts fit and DEG detection when individuals are genetically related
- Unlike other software kimma supports covariance matrices plus fit metrics like AIC — this combination is what enables the per-gene delta AIC comparison, so other pipelines cannot reproduce it directly
- kimma_lmerel accepts a kin_subset argument, so only the subjects present in the subset kinship matrix can be modeled with kinship; mismatched sample-to-kinship mapping will fail or bias fits

## Evidence

- [abstract] Unlike other software, kimma supports covariance matrices as well as fit metrics like Akaike information criterion (AIC): "Unlike other software, kimma supports covariance matrices as well as fit metrics like Akaike information criterion (AIC)"
- [abstract] Utilizing genetic kinship covariance, kimma revealed that kinship impacts model fit and DEG detection in a related cohort: "Utilizing genetic kinship covariance, kimma revealed that kinship impacts model fit and DEG detection in a related cohort"
- [methods] kimma_lmerel executes kinship-capable fits: "kimma_lmerel(model_lme, to_model_gene, gene, kin_subset, use_weights, patientID, metrics)"
- [methods] summarise_kmFit applies FDR and p cutoffs: "summarise_kmFit(fdr, fdr_cutoff, p_cutoff, FCgroup, intercept, fdr.cutoff, p.cutoff)"
- [readme] README installation and scope statement: "an open-source R package for flexible linear mixed effects modeling of RNA-seq including covariates, weights, random effects, covariance matrices, and fit metrics"
