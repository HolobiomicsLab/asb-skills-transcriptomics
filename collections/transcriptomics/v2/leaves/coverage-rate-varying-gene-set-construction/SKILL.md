---
name: coverage-rate-varying-gene-set-construction
description: Use when evaluating gene-set enrichment method calibration or power as
  a function of gene set size, by building synthetic gene sets at fixed coverage rates
  (e.g., 1%, 2%, 5%, 10%) over a fixed gene universe of null summary statistics.
when_to_use_negative:
- The goal is enrichment analysis on real, biologically fixed pathways (e.g., MSigDB
  KEGG/GO sets) rather than deliberately size-controlled synthetic sets for calibration
  testing
- Summary statistics contain embedded DE or enrichment signal — this skill targets
  the null scenario (tau0=-2, tau1=0) where no true gene-set enrichment exists
- The input is individual-level scRNA-seq data without gene-level beta/variance summaries,
  since CreateiDEAObject and the competitive GSE tools operate on summary statistics
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: iDEA
  role: Provides the CreateiDEAObject annotation framework whose min_percent_annot
    and max_var_beta filters define and prune the coverage-rate-varying gene sets;
    downstream iDEA.fit/iDEA.louis consumes them for calibrated p-values
  repo: https://github.com/xzhoulab/iDEA
- name: CAMERA
  role: Competitive GSE method run on the same CR-stratified null gene sets to expose
    its increasing over-conservatism as coverage rate rises
- name: fGSEA
  role: Competitive GSE baseline run on the CR-stratified null gene sets; expected
    to remain well-calibrated across coverage rates
- name: zingeRsimulationFunctions (zinbwaveZinger)
  role: Source of the simulation framework used to estimate parameters from real scRNA-seq
    data and generate the zero-inflated null counts underlying the summary statistics
  repo: https://github.com/statOmics/zinbwaveZinger
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-020-15298-6
    title: Integrative differential expression and gene set enrichment analysis using
      summary statistics for scRNA-seq studies
  source_content_hash: sha256:58c02ab5e71be92aa7dd2b2b869019ed7c6c4c819e46102e601f255d3407fd1a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xzhoulab/iDEA
  tools:
  - CAMERA
  - GSEA
  - PAGE
  - fGSEA
  - iDEA
  - zingeRsimulationFunctions (zinbwaveZinger)
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9285714285714286
n_claims: 14
---

# coverage-rate-varying-gene-set-construction

## Summary

Constructs synthetic gene sets of controlled size, expressed as coverage rate (CR = gene set size / number of tested genes), within the iDEA CreateiDEAObject annotation framework. This enables null-calibration evaluation of GSE methods (iDEA, fGSEA, PAGE, GSEA, CAMERA) as a function of gene set size on zero-inflated scRNA-seq summary statistics.

## When to use

Use when evaluating gene-set enrichment method calibration or power as a function of gene set size, by building synthetic gene sets at fixed coverage rates (e.g., 1%, 2%, 5%, 10%) over a fixed gene universe of null summary statistics.

## When NOT to use

- The goal is enrichment analysis on real, biologically fixed pathways (e.g., MSigDB KEGG/GO sets) rather than deliberately size-controlled synthetic sets for calibration testing
- Summary statistics contain embedded DE or enrichment signal — this skill targets the null scenario (tau0=-2, tau1=0) where no true gene-set enrichment exists
- The input is individual-level scRNA-seq data without gene-level beta/variance summaries, since CreateiDEAObject and the competitive GSE tools operate on summary statistics

## Inputs

- Null gene-level summary statistics (effect size beta and variance of beta) derived from simulated zero-inflated NB scRNA-seq counts (10,000 genes × 174 cells, ZINB parameters inferred from reference data)
- Gene annotation database (e.g., annotation_data.RData or humanGeneSets.RData from the xzhoulab/iDEA repository)
- Target coverage-rate grid (CR = 1%, 2%, 5%, 10% of the tested gene universe)

## Outputs

- Synthetic gene sets with controlled coverage rate, one collection per CR level over the same gene universe
- Filtered iDEA object from CreateiDEAObject (min_percent_annot=0.0025, max_var_beta=100) containing the annotated gene sets
- Per-method gene-set p-values at each CR, aggregated into lambda_gc inflation factors and quantile-quantile plots (calibration curves vs. the diagonal)

## How to apply

First generate or obtain gene-level summary statistics (effect size beta and its variance) on a fixed universe — in the reference simulation this is a zero-inflated negative-binomial count matrix of 10,000 genes × 174 cells with no embedded signal (null tau0=-2, tau1=0). Then sweep target coverage rates CR = gene set size / tested genes across 1%, 2%, 5%, and 10%, sampling genes into synthetic sets at each level. Build the annotated object with CreateiDEAObject, applying its filters: min_percent_annot (default 0.0025, the CR threshold below which sets are dropped) and max_var_beta (default 100, retaining only genes whose coefficient variance is below this cutoff). Feed the resulting CR-stratified gene sets plus summary statistics into iDEA.fit (em_iter=15, mcmc_iter=1000, fit.tol=1e-5, init_tau=c(-2,0.5)) and the competing competitive GSE tools (fGSEA, PAGE, GSEA, CAMERA). The rationale is that coverage rate modulates gene-gene correlation effects: CAMERA p-values become increasingly over-conservative as CR rises, whereas methods ignoring gene-gene correlation (iDEA, fGSEA, PAGE, GSEA) stay calibrated — quantified via lambda_gc and QQ plots per method per CR.

## Related tools

- **iDEA** (Provides the CreateiDEAObject annotation framework whose min_percent_annot and max_var_beta filters define and prune the coverage-rate-varying gene sets; downstream iDEA.fit/iDEA.louis consumes them for calibrated p-values) — https://github.com/xzhoulab/iDEA
- **CAMERA** (Competitive GSE method run on the same CR-stratified null gene sets to expose its increasing over-conservatism as coverage rate rises)
- **fGSEA** (Competitive GSE baseline run on the CR-stratified null gene sets; expected to remain well-calibrated across coverage rates)
- **zingeRsimulationFunctions (zinbwaveZinger)** (Source of the simulation framework used to estimate parameters from real scRNA-seq data and generate the zero-inflated null counts underlying the summary statistics) — https://github.com/statOmics/zinbwaveZinger

## Examples

```
library(iDEA); idea_obj <- CreateiDEAObject(beta, var_beta, gene_id, gene_sets, min_percent_annot=0.0025, max_var_beta=100)
```

## Evaluation signals

- For each CR level, gene set size divided by the number of tested genes (10,000 in the reference simulation) matches the target rate of 1%, 2%, 5%, or 10%
- All retained genes/sets pass the CreateiDEAObject filters: min_percent_annot=0.0025 (CR cutoff) and max_var_beta=100 (variance-of-coefficient cutoff)
- Under the null (tau0=-2, tau1=0), gene-set p-values for iDEA, fGSEA, PAGE, and GSEA yield lambda_gc near 1 with QQ points on the diagonal at every CR
- CAMERA p-values become increasingly deflated/conservative as CR increases, reproducing the Fig. 2 calibration pattern
- The tested gene universe is held constant across all CR levels so only gene set size (coverage rate) varies

## Limitations

- Coverage-rate effects conflate gene set size with gene-gene correlation structure; conclusions are specific to the simulated zero-inflated scRNA-seq setting and do not automatically transfer to bulk RNA-seq
- Very small gene sets are silently dropped by the min_percent_annot=0.0025 CR threshold, so extremely low-coverage constructions may yield no testable sets
- iDEA itself does not explicitly account for gene set overlap that may cause non-independence among gene sets, so per-set p-values across overlapping synthetic sets are not independent

## Evidence

- [methods] min_percent_annot: the threshold of coverage rate (CR), i.e., the number of annotated genes (gene set size) divided by the  number of tested genes. Default value is 0.0025.: "min_percent_annot: the threshold of coverage rate (CR), i.e., the number of annotated genes (gene set size) divided by the  number of tested genes. Default value is 0.0025."
- [methods] max_var_beta: the cutoff of the variance of the coefficient of genes. Genes with variance smaller than 'max_var_beta' are maintained. Default is 100: "max_var_beta: the cutoff of the variance of the coefficient of genes. Genes with variance smaller than 'max_var_beta' are maintained. Default is 100"
- [methods] The iDEA object is created by the function CreateiDEAObject.: "The iDEA object is created by the function CreateiDEAObject."
- [discussion] iDEA does not explicitly account for gene set overlap that may cause non-independence among gene sets: "iDEA does not explicitly account for gene set overlap that may cause non-independence among gene sets"
- [discussion] GSE methods that do not explicitly account for gene-gene correlation (e.g. iDEA, PAGE, fGSEA and GSEA) appear to provide more calibrated p-values: "GSE methods that do not explicitly account for gene-gene correlation (e.g. iDEA, PAGE, fGSEA and GSEA) appear to provide more calibrated p-values"
- [readme] devtools::install_github('xzhoulab/iDEA'): "devtools::install_github('xzhoulab/iDEA')"
