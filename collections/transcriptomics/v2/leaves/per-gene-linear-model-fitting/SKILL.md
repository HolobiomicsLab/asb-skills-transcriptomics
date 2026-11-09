---
name: per-gene-linear-model-fitting
description: Use when you have a gene-by-sample count matrix plus sample metadata
  and need unpaired per-gene differential expression testing without random effects,
  kinship, or weights.
when_to_use_negative:
- When samples are paired or repeated (patientID/libraryID structure) — use kimma's
  kimma_lme or kimma_lmerel modes instead of run_lm
- When modeling genetic relatedness is important — kinship covariance (example.kin)
  requires the lmerel mode, not the unpaired lm mode
- When you need voom precision weights — leave use_weights unset only for unweighted
  lm; weighted fitting is a different kimma option
edam_operation: http://edamontology.org/operation_0314
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: kimma
  role: R package whose kmFit(run_lm=TRUE) and kimma_lm machinery fit one linear model
    per gene on TMM-normalized log2 CPM
  repo: https://github.com/BIGslu/kimma
- name: stats
  role: Base R linear modeling engine (stats::lm) invoked per gene by kimma_lm
- name: devtools
  role: Used to install kimma from GitHub (devtools::install_github('BIGslu/kimma'))
  repo: https://github.com/BIGslu/kimma
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad279
    title: 'Kimma: flexible linear mixed effects modeling with kinship covariance
      for RNA-seq data'
  source_content_hash: sha256:772c949b4470e42e393002d85636478a6dd50222e6d177abd1e74a80a427f20c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BIGslu/kimma
  tools:
  - R
  - devtools
  - kimma
  - stats
  edam_operation: http://edamontology.org/operation_0314
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# per-gene-linear-model-fitting

## Summary

Fits an ordinary least-squares linear model per gene to RNA-seq expression using kimma's kmFit in unpaired lm mode, with expression input as TMM-normalized log2 CPM and p-values estimated via restricted maximum likelihood.

## When to use

Use when you have a gene-by-sample count matrix plus sample metadata and need unpaired per-gene differential expression testing without random effects, kinship, or weights.

## When NOT to use

- When samples are paired or repeated (patientID/libraryID structure) — use kimma's kimma_lme or kimma_lmerel modes instead of run_lm
- When modeling genetic relatedness is important — kinship covariance (example.kin) requires the lmerel mode, not the unpaired lm mode
- When you need voom precision weights — leave use_weights unset only for unweighted lm; weighted fitting is a different kimma option

## Inputs

- Gene count matrix (e.g., kimma's example.count) to be expressed as TMM-normalized log2 CPM
- Sample metadata table (e.g., kimma's example.dat) with covariates and grouping variables
- Gene list/identifier argument supplied to kmFit (genes)

## Outputs

- Per-gene linear model results table (one record per modeled gene) with coefficient estimates, fit metrics (e.g., AIC), and p-values/FDR, exportable as kimma_lm_results.csv

## How to apply

Load the kimma R package (installed via devtools::install_github('BIGslu/kimma')) and attach its bundled example.count (gene counts) and example.dat (sample metadata) datasets as in the instructional vignette. Call kmFit with run_lm=TRUE, supplying counts, meta, and genes arguments so expression is handled as TMM-normalized log2 CPM; this routes each gene through the kimma_lm machinery, which fits one stats::lm model per gene and estimates P-values using restricted maximum likelihood. Enable fit metrics (e.g., AIC) via the metrics argument and leave use_weights unset to fit unweighted models, since weights apply to paired/lme modes. Collect the returned per-gene results table of coefficient estimates, fit metrics, and p-values/FDR, and write it out (e.g., kimma_lm_results.csv). The article motivates this mode by showing kimma detects DEGs with specificity, sensitivity, and computational time similar to limma unpaired models.

## Related tools

- **kimma** (R package whose kmFit(run_lm=TRUE) and kimma_lm machinery fit one linear model per gene on TMM-normalized log2 CPM) — https://github.com/BIGslu/kimma
- **stats** (Base R linear modeling engine (stats::lm) invoked per gene by kimma_lm)
- **devtools** (Used to install kimma from GitHub (devtools::install_github('BIGslu/kimma'))) — https://github.com/BIGslu/kimma

## Examples

```
kmFit(dat = example.count, meta = example.dat, genes = gene_list, run_lm = TRUE, metrics = TRUE)
```

## Evaluation signals

- kimma_lm_results.csv exists and contains one record per modeled gene from the example data
- Each record includes estimate, fit-metric, and p-value/FDR fields as returned by kmFit lm mode
- Results are consistent with the lm-mode run shown in the kimma vignette (https://bigslu.github.io/kimma_vignette/kimma_vignette.html)
- Expression values entered the fit as TMM-normalized log2 CPM, and use_weights was unset (unweighted fitting)
- DEG detection should show specificity/sensitivity comparable to limma unpaired models per the paper's simulations

## Limitations

- The unpaired lm mode does not handle random effects, kinship covariance matrices, or paired designs; those require kimma_lme/lmerel
- Fit metrics like AIC are only reported when the metrics argument is enabled
- Performance equivalence to limma/dream was demonstrated on simulated datasets, not universal guarantees

## Evidence

- [abstract] kimma lm mode fits models per gene: "we introduce kimma (Kinship In Mixed Model Analysis), an open-source R package for flexible linear mixed effects modeling"
- [abstract] kimma comparable to limma unpaired: "kimma detects DEGs with similar specificity, sensitivity, and computational time as limma unpaired and dream paired models"
- [methods] kimma_lm internal function signature: "kimma_lm(model_lm, to_model_gene, gene, use_weights, metrics)"
- [methods] kmFit call signature: "kmFit(dat, kin, patientID, libraryID, counts, meta, genes, weights, subset_var, subset_lvl"
- [abstract] fit metrics support: "Unlike other software, kimma supports covariance matrices as well as fit metrics like Akaike information criterion (AIC)"
- [readme] installation from GitHub: "devtools::install_github("BIGslu/kimma")"
