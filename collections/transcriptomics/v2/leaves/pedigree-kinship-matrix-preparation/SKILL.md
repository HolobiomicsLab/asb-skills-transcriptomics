---
name: pedigree-kinship-matrix-preparation
description: Use when RNA-seq samples come from related individuals (family/pedigree
  cohort) and you want to model pairwise genetic relatedness as a covariance random
  effect, or to compare kinship vs non-kinship model fit via per-gene AIC.
when_to_use_negative:
- Samples are unrelated or no genetic relatedness/pedigree data exist — the lme random
  effect without kin, or an unpaired lm, is the appropriate model
- Input is raw sequence data or an unnormalized count matrix lacking the per-sample
  library structure (patientID/libraryID) that kmFit's paired design requires
- You only need standard unpaired DEG detection — limma unpaired or kimma's lm model
  suffices and matches kimma's performance for that design
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0203
tools:
- name: kimma
  role: R package whose kimma_cleaning and kmFit functions format the kinship matrix
    and fit the kinship-capable lmerel model (lme4qtl::relmatLmer underlies it)
  repo: https://github.com/BIGslu/kimma
- name: R
  role: runtime environment for installing kimma via devtools and running the kmFit
    pipeline
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad279
    title: 'Kimma: flexible linear mixed effects modeling with kinship covariance
      for RNA-seq data'
  source_content_hash: sha256:325d032fa34f083ac233f7906ebe840ecb4cb1d53ea200978e989406e9c29a5e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BIGslu/kimma
  tools:
  - R
  - kimma
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 12
---

# pedigree-kinship-matrix-preparation

## Summary

Prepare a pairwise genetic kinship matrix (e.g., kimma's example.kin) and format it so it can be passed as a covariance random effect to a linear mixed effects RNA-seq model via kimma's kmFit, which routes kinship to lme4qtl::relmatLmer. This enables testing whether relatedness among samples impacts model fit and differential expression detection.

## When to use

Use when RNA-seq samples come from related individuals (family/pedigree cohort) and you want to model pairwise genetic relatedness as a covariance random effect, or to compare kinship vs non-kinship model fit via per-gene AIC.

## When NOT to use

- Samples are unrelated or no genetic relatedness/pedigree data exist — the lme random effect without kin, or an unpaired lm, is the appropriate model
- Input is raw sequence data or an unnormalized count matrix lacking the per-sample library structure (patientID/libraryID) that kmFit's paired design requires
- You only need standard unpaired DEG detection — limma unpaired or kimma's lm model suffices and matches kimma's performance for that design

## Inputs

- Pairwise genetic kinship matrix over sample/patient IDs (e.g., example.kin packaged with kimma)
- Gene count matrix per sample/library (example.count)
- Sample metadata with patientID and libraryID columns (example.dat)
- Optional voom-derived observation weights (example.voom) when use_weights=TRUE

## Outputs

- Cleaned, mapped data structures (counts, meta, genes, kin) ready for kmFit
- Per-gene kinship (lmerel) model fits with AIC fit metrics
- delta_aic_per_gene.csv (gene, AIC_lm/lme, AIC_lmerel, delta AIC)
- kmfit_gene_results.csv (gene, estimate, FDR, group) from summarise_kmFit

## How to apply

Obtain a pairwise kinship matrix over the same sample IDs as your metadata (kimma ships example.kin for this). Load kimma's packaged example data (example.count gene counts, example.dat sample metadata, example.kin kinship matrix) and run kimma_cleaning with patientID, libraryID, counts, meta, genes, and kin arguments to map each object into the structures kmFit expects; this ensures row/column identity between kin and the patientID/libraryID design. Then call kmFit with model='lme', kin=example.kin, and run_lmerel=TRUE so each gene is fit both without relatedness (lm/lme) and with kinship covariance via lme4qtl::relmatLmer. Extract per-gene AIC from both fits and compute delta AIC (non-kinship minus kinship); delta AIC > 2 indicates better fit with kinship. Apply summarise_kmFit with chosen fdr.cutoff and p.cutoff to derive per-gene FDR calls on the contrasts. The rationale is that kimma uniquely supports covariance matrices together with fit metrics like AIC, so kinship's effect on model fit and DEG detection can be quantified within one pipeline.

## Related tools

- **kimma** (R package whose kimma_cleaning and kmFit functions format the kinship matrix and fit the kinship-capable lmerel model (lme4qtl::relmatLmer underlies it)) — https://github.com/BIGslu/kimma
- **R** (runtime environment for installing kimma via devtools and running the kmFit pipeline)

## Examples

```
devtools::install_github("BIGslu/kimma"); kmFit(dat=example.dat, kin=example.kin, model="lme", run_lm=TRUE, run_lmerel=TRUE, use_weights=TRUE, metrics=TRUE)
```

## Evaluation signals

- Output tables contain exactly one row per modeled gene
- Both AIC_lm/lme and AIC_lmerel columns are populated for every gene so delta AIC can be computed per gene
- Kinship matrix IDs align with patientID/libraryID in the metadata after kimma_cleaning
- Genes with delta AIC > 2 indicate better fit with kinship, mirroring the paper's kinship model-fit comparison
- FDR columns in kmfit_gene_results.csv are populated after summarise_kmFit with the chosen fdr.cutoff and p.cutoff

## Limitations

- Kinship modeling requires the paired sample design (patientID/libraryID); kimma's kinship model is specifically the lmerel type and does not apply to unpaired lm
- The kinship matrix must correspond to the cohort being modeled — the paper's demonstration used a related cohort, so unrelated cohorts gain nothing from covariance
- Interpretation of delta AIC > 2 is a per-gene fit comparison, not itself a differential expression call; FDR thresholds from summarise_kmFit remain necessary

## Evidence

- [methods] Kimma fits mixed effects models for paired sample designs with kinship by passing the covariance matrix to lme4qtl's relmatLmer function: [evidence span withheld — anchor status: not_found]
- [abstract] Unlike other software, kimma supports covariance matrices as well as fit metrics like Akaike information criterion (AIC): "Unlike other software, kimma supports covariance matrices as well as fit metrics like Akaike information criterion (AIC)"
- [abstract] Utilizing genetic kinship covariance, kimma revealed that kinship impacts model fit and DEG detection in a related cohort: "Utilizing genetic kinship covariance, kimma revealed that kinship impacts model fit and DEG detection in a related cohort"
- [methods] kimma_cleaning(dat, kin, patientID, libraryID, counts, meta, genes, weights, subset_var, subset_lvl: "kimma_cleaning(dat, kin, patientID, libraryID, counts, meta, genes, weights, subset_var, subset_lvl"
- [readme] Install kimma from GitHub via devtools: devtools::install_github("BIGslu/kimma"): "devtools::install_github("BIGslu/kimma")"
