---
name: rna-seq-differential-expression-analysis
description: Use when modeling RNA-seq gene counts from paired/related sample designs
  where a genetic kinship matrix is available, and you need per-gene DEG calls plus
  AIC-based comparison of models with and without the kinship covariance random effect.
when_to_use_negative:
- Samples are unrelated — without a meaningful kinship matrix, run only the lm/lme
  forms; the paper's kinship comparison requires a related cohort where kinship impacts
  model fit
- You need a model form not supported by kimma (it targets covariates, weights, random
  effects, covariance matrices, and fit metrics for RNA-seq, not arbitrary non-linear
  or single-cell modeling)
- Input is raw sequencing data (FASTQ) — kimma starts from a gene count matrix, not
  from alignment or quantification
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: kimma
  role: 'Primary package: kmFit fits lm, lme, and lmerel (kinship via lme4qtl::relmatLmer)
    models per gene; kimma_cleaning formats data; kmFit_contrast and summarise_kmFit
    produce estimates and FDR calls'
  repo: https://github.com/BIGslu/kimma
- name: R
  role: Runtime environment for the open-source package implementing linear mixed
    effects modeling of RNA-seq with covariates, weights, random effects, covariance
    matrices, and fit metrics
- name: devtools
  role: Installs kimma from GitHub via devtools::install_github("BIGslu/kimma")
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad279
    title: 'Kimma: flexible linear mixed effects modeling with kinship covariance
      for RNA-seq data'
  source_content_hash: sha256:768761dba51df13f458923edc9d78dbfe1920c988152f4c55a1c9c3054e4dddf
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BIGslu/kimma
  tools:
  - R
  - devtools
  - kimma
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 6
---

# rna-seq-differential-expression-analysis

## Summary

Fit per-gene linear models and linear mixed effects models to RNA-seq gene counts using the kimma R package, including a kinship-covariance model variant executed through lme4qtl::relmatLmer, then compare model fit (delta AIC) and differential expression calls (FDR) between kinship and non-kinship forms.

## When to use

Use when modeling RNA-seq gene counts from paired/related sample designs where a genetic kinship matrix is available, and you need per-gene DEG calls plus AIC-based comparison of models with and without the kinship covariance random effect.

## When NOT to use

- Samples are unrelated — without a meaningful kinship matrix, run only the lm/lme forms; the paper's kinship comparison requires a related cohort where kinship impacts model fit
- You need a model form not supported by kimma (it targets covariates, weights, random effects, covariance matrices, and fit metrics for RNA-seq, not arbitrary non-linear or single-cell modeling)
- Input is raw sequencing data (FASTQ) — kimma starts from a gene count matrix, not from alignment or quantification

## Inputs

- Gene count matrix (e.g., kimma's example.count)
- Sample metadata table (e.g., example.dat) with patientID and libraryID columns
- Pairwise genetic kinship matrix (e.g., example.kin)
- Optional voom precision weights (e.g., example.voom) for use_weights=TRUE
- Model formula string (e.g., model='lme') and contrast definitions

## Outputs

- delta_aic_per_gene.csv with columns gene, AIC_lm/lme, AIC_lmerel, delta AIC (one row per modeled gene)
- kmfit_gene_results.csv with columns gene, estimate, FDR, group (one row per gene contrast)
- Per-gene fit metrics (AIC) for both lm/lme and lmerel (kinship) model forms

## How to apply

Load kimma's packaged example data (example.count, example.dat, example.kin) and pass patientID, libraryID, counts, meta, genes, and kin through kimma_cleaning to build the expected structures. Run kmFit with model='lme', use_weights=TRUE, metrics=TRUE, run_lm=TRUE, and run_lmerel=TRUE, supplying kin=example.kin so each gene is fit both without relatedness (lm/lme) and with the kinship covariance via lme4qtl::relmatLmer. Extract per-gene AIC from both fits and compute delta AIC as non-kinship minus kinship; delta AIC > 2 flags genes better explained by kinship. Apply kmFit_contrast to the fit and summarise_kmFit with fdr.cutoff and p.cutoff to produce per-gene FDR calls for each group contrast. Rationale: kimma is uniquely positioned among DEG tools because, unlike other software, it supports covariance matrices alongside fit metrics like AIC, letting kinship's impact on model fit and DEG detection be quantified directly.

## Related tools

- **kimma** (Primary package: kmFit fits lm, lme, and lmerel (kinship via lme4qtl::relmatLmer) models per gene; kimma_cleaning formats data; kmFit_contrast and summarise_kmFit produce estimates and FDR calls) — https://github.com/BIGslu/kimma
- **R** (Runtime environment for the open-source package implementing linear mixed effects modeling of RNA-seq with covariates, weights, random effects, covariance matrices, and fit metrics)
- **devtools** (Installs kimma from GitHub via devtools::install_github("BIGslu/kimma"))

## Examples

```
install.packages("devtools"); devtools::install_github("BIGslu/kimma"); kmFit(dat=example.count, kin=example.kin, patientID=patientID, libraryID=libraryID, model="lme", run_lm=TRUE, run_lmerel=TRUE, metrics=TRUE, use_weights=TRUE)
```

## Evaluation signals

- Both output tables contain exactly one row per modeled gene, with AIC values present for both the non-kinship (lm/lme) and kinship (lmerel) fits
- Delta AIC computed consistently as non-kinship minus kinship; genes with delta AIC > 2 indicate better fit with kinship, mirroring the paper's model-fit comparison
- FDR and p-value columns are populated after summarise_kmFit with the chosen fdr.cutoff and p.cutoff, and estimates carry the expected group contrast sign
- The lmerel run only succeeds when a kin matrix was supplied; absence of lmerel results indicates kin was not passed to kmFit
- DEG calls should be broadly consistent in specificity/sensitivity with limma unpaired and dream paired baselines, per the paper's simulation benchmarks

## Limitations

- In simulated datasets kimma performs with similar specificity, sensitivity, and computational time to limma unpaired and dream paired models, so gains are in model flexibility (covariance matrices + AIC), not raw detection power
- Kinship's effect on model fit and DEG detection was demonstrated in a related cohort; unrelated designs do not benefit from the lmerel/relmatLmer path
- Fit metrics like AIC require metrics=TRUE; omitting weights (use_weights=FALSE) changes the comparison relative to the voom-weighted workflow

## Evidence

- [abstract] Unlike other software, kimma supports covariance matrices as well as fit metrics like Akaike information criterion (AIC): "Unlike other software, kimma supports covariance matrices as well as fit metrics like Akaike information criterion (AIC)"
- [intro] Utilizing genetic kinship covariance, kimma revealed that kinship impacts model fit and DEG detection in a related cohort: "Utilizing genetic kinship covariance, kimma revealed that kinship impacts model fit and DEG detection in a related cohort"
- [methods] kmFit(dat, kin, patientID, libraryID, counts, meta, genes, weights, subset_var, subset_lvl: "kmFit(dat, kin, patientID, libraryID, counts, meta, genes, weights, subset_var, subset_lvl"
- [full_text] Kimma fits mixed effects models for paired sample designs with kinship by passing the covariance matrix to lme4qtl's relmatLmer function, which underlies the kinship-capable model type.: [evidence span withheld — anchor status: not_found]
- [readme] devtools::install_github("BIGslu/kimma"): "devtools::install_github("BIGslu/kimma")"
