---
name: differential-expression-modeling
description: Use when you need per-gene DEG testing on RNA-seq count data with an
  unpaired design (no within-subject pairing, no kinship/random effects), using kimma's
  kmFit run_lm=TRUE on TMM-normalized log2 CPM.
when_to_use_negative:
- The design is paired (within-subject repeated measures) — use kimma_lme / kmFit
  lme mode instead
- Samples are genetically related and you need kinship covariance — use kimma_lmerel
  with a kinship matrix (e.g., example.kin) instead
- Expression input is raw counts or not TMM-normalized log2 CPM — normalize first
  (e.g., via the example.seasnake/voom-style preprocessing shown in the vignette)
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: kimma
  role: Provides kmFit and the kimma_lm machinery for per-gene unpaired linear model
    DEG fitting
  repo: https://github.com/BIGslu/kimma
- name: R
  role: Runtime environment for the kimma package and stats::lm fitting
  repo: https://github.com/BIGslu/kimma
- name: stats
  role: Base R stats::lm performs the ordinary least-squares fit per gene
- name: devtools
  role: Used to install kimma from GitHub
  repo: https://github.com/BIGslu/kimma
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1093/bioinformatics/btad279
    title: 'Kimma: flexible linear mixed effects modeling with kinship covariance
      for RNA-seq data'
  source_content_hash: sha256:d3e27e0dcfa82c46ef65c802b0b0745f60675ddb2d3cf2c62336471ecb4e5383
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/BIGslu/kimma
  tools:
  - R
  - devtools
  - kimma
  - stats
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.4
n_claims: 5
---

# differential-expression-modeling

## Summary

Fit per-gene unpaired linear models for RNA-seq differential expression using kimma's kmFit in lm mode, where gene expression is TMM-normalized log2 CPM and P-values are estimated via restricted maximum likelihood through base R stats::lm (kimma_lm machinery).

## When to use

Use when you need per-gene DEG testing on RNA-seq count data with an unpaired design (no within-subject pairing, no kinship/random effects), using kimma's kmFit run_lm=TRUE on TMM-normalized log2 CPM.

## When NOT to use

- The design is paired (within-subject repeated measures) — use kimma_lme / kmFit lme mode instead
- Samples are genetically related and you need kinship covariance — use kimma_lmerel with a kinship matrix (e.g., example.kin) instead
- Expression input is raw counts or not TMM-normalized log2 CPM — normalize first (e.g., via the example.seasnake/voom-style preprocessing shown in the vignette)

## Inputs

- Gene count matrix (e.g., kimma's example.count)
- Sample metadata table (e.g., kimma's example.dat)
- Gene list/vector (genes argument) naming the genes to model
- Model formula specification for the unpaired linear model (model_lm)

## Outputs

- Per-gene linear model results table with coefficient estimates, fit metrics, and p-values/FDR (e.g., kimma_lm_results.csv)
- Summarised lm fit results from summarise_lmFit (FDR/p cutoffs, fold change group, intercept)

## How to apply

Attach kimma's bundled example data (example.count gene counts, example.dat sample metadata) as in the instructional vignette, then call kmFit with run_lm=TRUE, supplying counts, meta, and genes so expression is handled as TMM-normalized log2 CPM. Under the hood, kmFit dispatches to kimma_lm (model_lm, to_model_gene, gene, use_weights, metrics), which fits one ordinary least-squares model per gene with stats::lm; leave use_weights unset for unweighted fitting and enable fit metrics (e.g., AIC) via the metrics argument. Collect the returned per-gene results table of coefficient estimates, fit metrics, and p-values, then apply FDR summarisation via summarise_lmFit (fdr_cutoff, p_cutoff, FCgroup, intercept) before export. Validate that the output contains one record per modeled gene with estimate, fit-metric, and p-value fields, consistent with the lm-mode run in the kimma vignette.

## Related tools

- **kimma** (Provides kmFit and the kimma_lm machinery for per-gene unpaired linear model DEG fitting) — https://github.com/BIGslu/kimma
- **R** (Runtime environment for the kimma package and stats::lm fitting) — https://github.com/BIGslu/kimma
- **stats** (Base R stats::lm performs the ordinary least-squares fit per gene)
- **devtools** (Used to install kimma from GitHub) — https://github.com/BIGslu/kimma

## Examples

```
library(kimma); res <- kmFit(dat = example.voom, counts = example.count, meta = example.dat, genes = gene, run_lm = TRUE, metrics = TRUE); write.csv(res$lm, "kimma_lm_results.csv", row.names = FALSE)
```

## Evaluation signals

- kimma_lm_results.csv exists and contains one record per modeled gene from the example data
- Each record carries estimate, fit-metric, and p-value fields
- Results are consistent with the lm-mode run shown in the kimma vignette (https://bigslu.github.io/kimma_vignette/kimma_vignette.html)
- FDR summarisation via summarise_lmFit applies the intended fdr_cutoff/p_cutoff and matches reported FCgroup/intercept values
- Expression values fed to the model are TMM-normalized log2 CPM, not raw counts

## Limitations

- Unpaired lm mode does not use observation weights (use_weights unset) and ignores random effects and kinship covariance
- P-value estimation relies on restricted maximum likelihood behavior of the fitted linear models; performance is benchmarked against limma unpaired and dream paired models, so complex designs may require kimma_lme/lmerel
- Results depend on kimma's internal normalization to TMM-normalized log2 CPM; supplying differently normalized data changes inference

## Evidence

- [other] kimma's unpaired linear modeling fits linear models per gene and estimates P-values using restricted maximum likelihood, with gene expression modeled as TMM normalized log2 CPM.: [evidence span withheld — anchor status: not_found]
- [methods] kmFit dispatch signature: "kmFit(dat, kin, patientID, libraryID, counts, meta, genes, weights, subset_var, subset_lvl"
- [methods] kimma_lm internal function: "kimma_lm(model_lm, to_model_gene, gene, use_weights, metrics)"
- [methods] summarise_lmFit signature: "summarise_lmFit(fdr, fdr_cutoff, p_cutoff, FCgroup, intercept, fdr.cutoff, p.cutoff)"
- [readme] installation and vignette from README: "Kimma is freely available on GitHub https://github.com/BIGslu/kimma with an instructional vignette at https://bigslu.github.io/kimma_vignette/kimma_vignette.html"
- [abstract] kimma benchmark context: "kimma detects DEGs with similar specificity, sensitivity, and computational time as limma unpaired and dream paired models"
