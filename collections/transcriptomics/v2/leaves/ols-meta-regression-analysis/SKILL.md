---
name: ols-meta-regression-analysis
description: Use when per-gene model measures (e.g., CTMM cell type-specificity, mean-difference)
  must be related to gene-level annotations such as EDS, enhancer count, LOEUF, or
  pLI via decile-binned regression.
when_to_use_negative:
- When the question requires gene-level (single-gene) association with annotations
  rather than decile-aggregated trends — decile binning discards within-bin variation.
- When the input measures are not per-gene (e.g., cell-level or variant-level), since
  genes are ranked and binned into deciles across the whole gene set.
- When annotations are not gene-mapped (e.g., cell type proportions or individual-level
  covariates), which cannot be deciled per gene.
tools:
- name: Python
  role: Implements decile binning, per-decile aggregation, and the OLS regressions;
    used to write gene_decile_regression_stats.tsv
- name: ctmm
  role: Provides the per-gene cell type-specificity and mean-difference measures that
    are aggregated by annotation decile
  repo: https://github.com/Minhui-Chen/ctmm_python
- name: rpy2
  role: Bridge through which ctmm calls R routines during model fitting that precedes
    the decile regression
  repo: https://rpy2.github.io/
- name: R
  role: Required runtime for ctmm's routines (with abind, mvtnorm, numDeriv, softImpute,
    tensor)
- name: Snakemake
  role: Orchestrates the simulation and analysis pipeline steps containing this regression
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-49242-9
    title: A robust model for cell type-specific interindividual variation in single-cell
      RNA sequencing data
  source_content_hash: sha256:3f0c9dbf1dd824bf1f45ffe4a4bb42ad8411488d7fca6d95beaf34bd81c36217
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Minhui-Chen/ctmm_python
  tools:
  - Python
  - R
  - Snakemake
  - ctmm
  - rpy2
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# ols-meta-regression-analysis

## Summary

Gene-decile OLS regression analysis that relates CTMM's cell type-specific interindividual variance measure (and mean-difference specificity) to genome-structure and conservation annotations (EDS, enhancer count, LOEUF, pLI) by aggregating per-gene measures into 10 equal-size annotation deciles and regressing decile means on decile ranks.

## When to use

Use when per-gene model measures (e.g., CTMM cell type-specificity, mean-difference) must be related to gene-level annotations such as EDS, enhancer count, LOEUF, or pLI via decile-binned regression.

## When NOT to use

- When the question requires gene-level (single-gene) association with annotations rather than decile-aggregated trends — decile binning discards within-bin variation.
- When the input measures are not per-gene (e.g., cell-level or variant-level), since genes are ranked and binned into deciles across the whole gene set.
- When annotations are not gene-mapped (e.g., cell type proportions or individual-level covariates), which cannot be deciled per gene.

## Inputs

- per-gene CTMM cell type-specificity measures (from ctp Free-model REML fits)
- per-gene cell type mean-difference measures (ct_beta differentiation between cell types)
- gene-level annotation table with EDS (enhancer domain score), number of enhancers, LOEUF, and pLI scores

## Outputs

- gene_decile_regression_stats.tsv: regression coefficients, standard errors, and p-values for each annotation-measure pair
- per-decile mean values of the CTMM and mean-difference measures for each of the four annotations

## How to apply

Load per-gene CTMM results (cell type-specificity measures from the Free-model fit, e.g., ctp.free_REML, and cell type mean-difference measures such as ct_beta differentiation) together with a gene-level annotation table containing EDS, number of enhancers, LOEUF, and pLI. Rank all genes and assign each to one of 10 equal-size decile bins independently for each of the four annotations. Aggregate the CTMM and mean-difference measures by taking their means within each decile, then fit ordinary least-squares (OLS) regressions of each per-decile measure on the annotation decile ranks — one regression per annotation-measure pair. Collect regression coefficients, standard errors, and p-values into a summary table written as gene_decile_regression_stats.tsv.

## Related tools

- **Python** (Implements decile binning, per-decile aggregation, and the OLS regressions; used to write gene_decile_regression_stats.tsv)
- **ctmm** (Provides the per-gene cell type-specificity and mean-difference measures that are aggregated by annotation decile) — https://github.com/Minhui-Chen/ctmm_python
- **rpy2** (Bridge through which ctmm calls R routines during model fitting that precedes the decile regression) — https://rpy2.github.io/
- **R** (Required runtime for ctmm's routines (with abind, mvtnorm, numDeriv, softImpute, tensor))
- **Snakemake** (Orchestrates the simulation and analysis pipeline steps containing this regression)

## Examples

```
import statsmodels.api as sm; res = sm.OLS(ctmm_decile_means, sm.OLS? annotation_decile_ranks, ...)
```

## Evaluation signals

- The OLS regression of the mean-difference measure on LOEUF decile ranks should yield p ~ 1.0e-3, matching the reported meta-regression result.
- The OLS regression of the CTMM cell type-specificity measure on LOEUF decile ranks should yield p ~ 1.2e-8 and slope in the direction opposite to the mean-difference result.
- Each annotation's decile assignment must yield 10 equal-size bins containing all ranked genes.
- gene_decile_regression_stats.tsv must contain, per annotation-measure pair, a regression coefficient, standard error, and p-value.
- Both CTMM and mean-difference signals should show enrichment in genes with larger enhancer domains (larger EDS deciles).

## Limitations

- CTMM results underlying the measures require biological validation and replication due to test inflation noted in the paper.
- CTMM is not reliable for sample sizes below ~50 individuals, and the Free test is slightly inflated, which propagates into the aggregated decile measures.
- Decile aggregation sacrifices within-bin resolution; conclusions are about binned trends, not individual gene associations.
- The measures derive from pseudobulk data, which discards cell-level information and limits CTMM to common cell types (>10 cells per individual-cell type pair).

## Evidence

- [other] Decile binning and regression workflow: [evidence span withheld — anchor status: not_found]
- [other] OLS specification: [evidence span withheld — anchor status: not_found]
- [other] Validated p-value expectations: [evidence span withheld — anchor status: not_found]
- [readme] Upstream CTMM measure source: "free_jk, p_jk = ctp.free_REML(y_f=CTP_f, P_f=P_f, ctnu_f=ctnu_f, method='BFGS', optim_by_R=True, jack_knife=True)"
- [readme] Pipeline context: "* [Snakefile](Snakefile) contains steps for perform simulations and iPSCs analyses"
