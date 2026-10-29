---
name: genetic-constraint-score-interpretation
description: Use when per-gene statistics such as CTMM cell type-specificity or mean-difference
  measures must be related to gene-level annotations like LOEUF, pLI, enhancer domain
  score (EDS), or enhancer count.
when_to_use_negative:
- Input is raw scRNA-seq counts rather than per-gene CTMM model outputs — this skill
  operates on fitted cell type-specificity and mean-difference measures, not single-cell
  data.
- The goal is assessing calibration or power of CTMM tests (e.g., via simulations)
  — decile regression against LOEUF/pLI/EDS tests annotation association, not statistical
  calibration.
- The gene set lacks constraint/enhancer annotations (LOEUF, pLI, EDS, enhancer count)
  for all genes, since decile binning and rank regression cannot be computed.
edam_operation: http://edamontology.org/operation_2238
edam_topics:
- http://edamontology.org/topic_2269
- http://edamontology.org/topic_0203
tools:
- name: Python
  role: Implements gene ranking into 10 equal-size decile bins per annotation, per-decile
    mean aggregation, OLS regressions on decile ranks, and writes gene_decile_regression_stats.tsv
  repo: https://github.com/Minhui-Chen/CTMM
- name: ctmm
  role: Fits CTMM models (op/ctp modules, REML/HE with jackknife) to produce the per-gene
    cell type-specificity and mean-difference measures used as dependent variables
  repo: https://github.com/Minhui-Chen/ctmm_python
- name: Snakemake
  role: Orchestrates the Snakefile pipeline steps that generate the CTMM outputs feeding
    the decile regressions
  repo: https://github.com/Minhui-Chen/CTMM
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-49242-9
    title: A robust model for cell type-specific interindividual variation in single-cell
      RNA sequencing data
  source_content_hash: sha256:10c4e302523fcc4bfc53b5554efac13547b36af1a0e78035e49f561a377c903c
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Minhui-Chen/CTMM
  tools:
  - Python
  - R
  - Snakemake
  - ctmm
  edam_operation: http://edamontology.org/operation_2238
  edam_topics:
  - http://edamontology.org/topic_2269
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 5
---

# genetic-constraint-score-interpretation

## Summary

Relates gene-level genetic constraint and enhancer annotations (LOEUF, pLI, enhancer domain score, enhancer count) to CTMM cell type-specific interindividual variance measures via 10 equal-size decile binning and OLS regression on decile ranks, revealing that LOEUF constraint associates in opposite directions with CTMM specificity versus mean-difference cell type-specificity.

## When to use

Use when per-gene statistics such as CTMM cell type-specificity or mean-difference measures must be related to gene-level annotations like LOEUF, pLI, enhancer domain score (EDS), or enhancer count.

## When NOT to use

- Input is raw scRNA-seq counts rather than per-gene CTMM model outputs — this skill operates on fitted cell type-specificity and mean-difference measures, not single-cell data.
- The goal is assessing calibration or power of CTMM tests (e.g., via simulations) — decile regression against LOEUF/pLI/EDS tests annotation association, not statistical calibration.
- The gene set lacks constraint/enhancer annotations (LOEUF, pLI, EDS, enhancer count) for all genes, since decile binning and rank regression cannot be computed.

## Inputs

- Per-gene CTMM results table: cell type-specificity measures and cell type mean-difference measures
- Gene-level annotation table: EDS (enhancer domain score), number of enhancers, LOEUF, and pLI scores per gene

## Outputs

- gene_decile_regression_stats.tsv: OLS regression coefficients, standard errors, and p-values for each annotation-measure pair
- Per-decile mean values of the CTMM cell type-specificity and mean-difference measures for each annotation decile

## How to apply

Load the per-gene CTMM results — cell type-specificity measures and cell type mean-difference measures — together with the gene-level annotation table containing EDS (enhancer domain score), enhancer count, LOEUF, and pLI. Independently rank all genes and assign each to one of 10 equal-size decile bins for each of the four annotations, then aggregate both CTMM measures by their mean within each annotation decile. Fit ordinary least-squares (OLS) regressions of the per-decile mean-difference measure and the per-decile CTMM cell type-specificity measure on the annotation decile ranks, one regression per annotation-measure pair (4 annotations × 2 measures = 8 pairs). Collect the regression coefficients, standard errors, and p-values into a summary table written as gene_decile_regression_stats.tsv. The decile-mean regression stabilizes noisy per-gene estimates so the trend tests whether constraint or enhancer annotations explain cell type-specific variance at the meta-analysis level; the reference outcomes are mean-difference vs LOEUF p ≈ 1.0e-3 and CTMM specificity vs LOEUF p ≈ 1.2e-8, with the two measures showing opposite slopes.

## Related tools

- **Python** (Implements gene ranking into 10 equal-size decile bins per annotation, per-decile mean aggregation, OLS regressions on decile ranks, and writes gene_decile_regression_stats.tsv) — https://github.com/Minhui-Chen/CTMM
- **ctmm** (Fits CTMM models (op/ctp modules, REML/HE with jackknife) to produce the per-gene cell type-specificity and mean-difference measures used as dependent variables) — https://github.com/Minhui-Chen/ctmm_python
- **Snakemake** (Orchestrates the Snakefile pipeline steps that generate the CTMM outputs feeding the decile regressions) — https://github.com/Minhui-Chen/CTMM

## Examples

```
from scipy.stats import linregress; slope, intercept, r, p, se = linregress(loeuf_decile_rank, per_decile_mean_measure)
```

## Evaluation signals

- OLS regression of per-decile mean-difference measure on LOEUF decile rank yields p ≈ 1.0e-3 (more-constrained genes have lower mean differences across cell types).
- OLS regression of per-decile CTMM cell type-specificity measure on LOEUF decile rank yields p ≈ 1.2e-8 with slope opposite in sign to the mean-difference regression.
- gene_decile_regression_stats.tsv contains coefficient, standard error, and p-value entries for all 8 annotation-measure pairs (EDS, enhancer count, LOEUF, pLI × CTMM specificity, mean-difference).
- Each of the 10 deciles contains an equal number of genes for every annotation (equal-size bins, independently ranked per annotation).
- Both measures trend upward across EDS deciles, consistent with enrichment of CTMM and mean-difference signals in genes with larger enhancer domains.

## Limitations

- CTMM is not reliable for sample sizes below ~50 individuals (the Free test is slightly inflated), so upstream per-gene measures require adequate cohort size.
- CTMM requires >10 cells per individual-cell type pair, limiting the common cell types whose measures enter the regression.
- Lowly-expressed genes can be severely non-Gaussian, increasing the number of cells needed for the Gaussian approximation and potentially biasing variance estimates.
- Decile binning discards within-bin gene-level variation; associations are tested on decile means (meta-regression), not on individual genes.

## Evidence

- [results] more-constrained genes have lower mean differences across cell types (p = 1.0e-3), while CTMM's cell type-specificity measure correlates with LOEUF in the opposite direction (p = 1.2e-8).: [evidence span withheld — anchor status: not_found]
- [methods] Rank all genes and assign each to one of 10 equal-size decile bins independently for each of the four annotations (EDS, enhancer count, LOEUF, pLI).: [evidence span withheld — anchor status: not_found]
- [other] Fit ordinary least-squares (OLS) regressions of the per-decile mean-difference measure and the per-decile CTMM measure on the annotation decile ranks, one regression per annotation-measure pair.: [evidence span withheld — anchor status: not_found]
- [methods] Collect regression coefficients, standard errors, and p-values into a summary table and write it as gene_decile_regression_stats.tsv.: [evidence span withheld — anchor status: not_found]
- [readme] CTMM calls R routines through [rpy2](https://rpy2.github.io/), so a working R installation is required.: "CTMM calls R routines through [rpy2](https://rpy2.github.io/), so a working R installation is required."
- [discussion] In practice, we require >10 cells per individual-cell type pair, which limits CTMM to common cell types.: "In practice, we require >10 cells per individual-cell type pair, which limits CTMM to common cell types."
