---
name: delta-aic-model-comparison
description: Use when RNA-seq data come from related/paired samples and you need to
  quantify, per gene, whether adding a kinship covariance random effect improves mixed
  model fit via AIC.
when_to_use_negative:
- Samples are unrelated/unpaired and no kinship matrix exists — the lm/lme-only fits
  provide no delta AIC contrast to interpret.
- The goal is differential expression calling alone; kimma matches limma unpaired
  and dream paired specificity/sensitivity, so those pipelines suffice without AIC
  model comparison.
- Input is already per-gene model output/AIC values — skip refitting and compute delta
  AIC directly.
edam_operation: http://edamontology.org/operation_3891
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0203
tools:
- name: kimma
  role: R package whose kmFit, kimma_cleaning, kimma_lm, kimma_lme, kimma_lmerel and
    summarise_kmFit functions run both model forms and expose per-gene AIC metrics
  repo: https://github.com/BIGslu/kimma
- name: R
  role: execution environment for kimma and for computing per-gene delta AIC between
    fit forms
- name: lme4qtl
  role: relmatLmer underlies the kinship-capable (lmerel) model fit whose AIC is compared
    against the non-kinship fit
- name: devtools
  role: installs kimma from GitHub via devtools::install_github('BIGslu/kimma')
  repo: https://github.com/BIGslu/kimma
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1093/bioinformatics/btad279
    title: 'Kimma: flexible linear mixed effects modeling with kinship covariance
      for RNA-seq data'
  source_content_hash: sha256:54b8d4bb3de8a787025451db3e25d07127b187ebca24a3bbb167bdf1e8d5e5d8
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BIGslu/kimma
  tools:
  - R
  - devtools
  - kimma
  - lme4qtl
  edam_operation: http://edamontology.org/operation_3891
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# delta-aic-model-comparison

## Summary

Per-gene model-fit comparison using Akaike information criterion (AIC) from paired linear mixed effects fits, computing delta AIC (non-kinship model minus kinship model) to quantify whether genetic kinship covariance improves RNA-seq model fit for each gene.

## When to use

Use when RNA-seq data come from related/paired samples and you need to quantify, per gene, whether adding a kinship covariance random effect improves mixed model fit via AIC.

## When NOT to use

- Samples are unrelated/unpaired and no kinship matrix exists — the lm/lme-only fits provide no delta AIC contrast to interpret.
- The goal is differential expression calling alone; kimma matches limma unpaired and dream paired specificity/sensitivity, so those pipelines suffice without AIC model comparison.
- Input is already per-gene model output/AIC values — skip refitting and compute delta AIC directly.

## Inputs

- gene count matrix (e.g. kimma example.count)
- sample metadata (e.g. kimma example.dat) with patientID and libraryID columns
- pairwise genetic kinship matrix (e.g. kimma example.kin)
- optionally voom-derived observation weights (example.voom)

## Outputs

- delta_aic_per_gene.csv with columns gene, AIC_lm/lme, AIC_lmerel, delta AIC (one row per modeled gene)
- kmfit_gene_results.csv with columns gene, estimate, FDR, group from summarise_kmFit

## How to apply

Load kimma's packaged example data (example.count, example.dat, example.kin) and format them with kimma_cleaning, mapping patientID, libraryID, counts, meta, genes, and kin into expected structures. Run kmFit with model='lme', use_weights=TRUE, metrics=TRUE, run_lm=TRUE and run_lmerel=TRUE, passing kin=example.kin so each gene is fit both without relatedness (lm/lme) and with kinship covariance handled by lme4qtl::relmatLmer. Extract per-gene AIC from both fit forms, compute delta AIC as the non-kinship AIC minus the kinship-model AIC for every gene. Run summarise_kmFit with fdr.cutoff and p.cutoff on contrast results to populate FDR calls alongside the fit comparison; genes with delta AIC > 2 indicate better fit with kinship.

## Related tools

- **kimma** (R package whose kmFit, kimma_cleaning, kimma_lm, kimma_lme, kimma_lmerel and summarise_kmFit functions run both model forms and expose per-gene AIC metrics) — https://github.com/BIGslu/kimma
- **R** (execution environment for kimma and for computing per-gene delta AIC between fit forms)
- **lme4qtl** (relmatLmer underlies the kinship-capable (lmerel) model fit whose AIC is compared against the non-kinship fit)
- **devtools** (installs kimma from GitHub via devtools::install_github('BIGslu/kimma')) — https://github.com/BIGslu/kimma

## Examples

```
kmFit(dat, kin=example.kin, patientID, libraryID, counts, meta, genes, model='lme', use_weights=TRUE, metrics=TRUE, run_lm=TRUE, run_lmerel=TRUE)
```

## Evaluation signals

- Both output tables contain exactly one row per modeled gene.
- Each row carries AIC values for both model forms (lm/lme and lmerel) so delta AIC is fully populated.
- FDR columns in kmfit_gene_results.csv are populated after summarise_kmFit with fdr.cutoff and p.cutoff.
- Genes with delta AIC > 2 are interpreted as better fit with kinship, mirroring the paper's kinship model-fit comparison.
- kmFit was invoked with metrics=TRUE so fit metrics (AIC) are actually returned.

## Limitations

- Kinship comparison requires a pairwise kinship matrix for the related cohort; without it the lmerel fit cannot be run.
- kimma supports covariance matrices plus fit metrics like AIC, unlike other software, so equivalent delta AIC comparisons may not be reproducible in limma/dream pipelines.
- Kinship impacts both model fit and DEG detection, so delta AIC fit improvements may coincide with changed DEG calls rather than only improved fits.

## Evidence

- [abstract] Unlike other software, kimma supports covariance matrices as well as fit metrics like Akaike information criterion (AIC): "Unlike other software, kimma supports covariance matrices as well as fit metrics like Akaike information criterion (AIC)"
- [methods] kimma_lmerel function: "kimma_lmerel(model_lme, to_model_gene, gene, kin_subset, use_weights, patientID, metrics)"
- [methods] summarise_kmFit function: "summarise_kmFit(fdr, fdr_cutoff, p_cutoff, FCgroup, intercept, fdr.cutoff, p.cutoff)"
- [intro] kinship impacts model fit and DEG detection: "Utilizing genetic kinship covariance, kimma revealed that kinship impacts model fit and DEG detection in a related cohort"
- [readme] vignette link in README: "Kimma is freely available on GitHub https://github.com/BIGslu/kimma with an instructional vignette at https://bigslu.github.io/kimma_vignette/kimma_vignette.html"
