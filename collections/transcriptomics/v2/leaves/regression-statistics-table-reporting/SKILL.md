---
name: regression-statistics-table-reporting
description: Use when per-gene CTMM results (cell type-specificity and mean-difference
  measures) must be compared against gene-level annotations such as enhancer domain
  score, enhancer count, LOEUF, or pLI to ask whether interindividual variance tracks
  genome features differently than mean-based cell.
when_to_use_negative:
- The goal is to fit the CTMM mixed model itself or generate pseudobulk (OP/CTP) inputs
  — this skill only consumes per-gene CTMM output; use the ctmm package (op/ctp modules)
  instead
- Gene-level annotations such as LOEUF or EDS are unavailable, since all four annotations
  must be decile-binned for the regression design
- You need per-gene resolution rather than decile-level summaries — the 10-bin aggregation
  discards within-decile variation
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: Python
  role: Implements decile binning, per-decile aggregation, OLS regression fitting,
    and writes the gene_decile_regression_stats.tsv summary table
- name: ctmm
  role: Produces the per-gene CTMM cell type-specificity measures that serve as one
    of the two response measures in the decile regressions
  repo: https://github.com/Minhui-Chen/ctmm_python
- name: Snakemake
  role: Orchestrates the analysis pipeline steps that run the regression workflow
  repo: https://github.com/Minhui-Chen/CTMM
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-024-49242-9
    title: A robust model for cell type-specific interindividual variation in single-cell
      RNA sequencing data
  source_content_hash: sha256:6641cc70df063e7608c654501120b0e04bd4c2df1fd0e550833e0e97e3b3e70b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Minhui-Chen/ctmm_python
  tools:
  - Python
  - R
  - Snakemake
  - ctmm
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# regression-statistics-table-reporting

## Summary

Method for relating per-gene CTMM cell type-specificity and cell type mean-difference measures to genome-structure/conservation annotations (EDS, enhancer count, LOEUF, pLI) by decile binning genes and fitting per-decile OLS regressions, then reporting coefficients, standard errors, and p-values as a TSV summary table.

## When to use

Use when per-gene CTMM results (cell type-specificity and mean-difference measures) must be compared against gene-level annotations such as enhancer domain score, enhancer count, LOEUF, or pLI to ask whether interindividual variance tracks genome features differently than mean-based cell type-specificity.

## When NOT to use

- The goal is to fit the CTMM mixed model itself or generate pseudobulk (OP/CTP) inputs — this skill only consumes per-gene CTMM output; use the ctmm package (op/ctp modules) instead
- Gene-level annotations such as LOEUF or EDS are unavailable, since all four annotations must be decile-binned for the regression design
- You need per-gene resolution rather than decile-level summaries — the 10-bin aggregation discards within-decile variation

## Inputs

- Per-gene CTMM results table (cell type-specificity measure and cell type mean-difference measure per gene)
- Gene-level annotation table with EDS (enhancer domain score), number of enhancers, LOEUF, and pLI scores

## Outputs

- gene_decile_regression_stats.tsv — per-decile mean measures plus OLS regression coefficients, standard errors, and p-values for each annotation-measure pair
- Per-annotation decile assignments of all genes (10 equal-size bins per annotation)

## How to apply

Load the per-gene CTMM results (cell type-specificity measures and cell type mean-difference measures) together with the gene-level annotation table containing EDS, number of enhancers, LOEUF, and pLI. Independently rank all genes and assign each to one of 10 equal-size decile bins per annotation (EDS, enhancer count, LOEUF, pLI), then average the CTMM cell type-specificity measure and the mean-difference measure within each decile. Fit ordinary least-squares regressions of each per-decile measure on the annotation decile ranks — one regression per annotation-measure pair — collecting coefficients, standard errors, and p-values. Write the 8 regression results to gene_decile_regression_stats.tsv. Binning into deciles before regression smooths gene-level noise and lets the two variance notions (CTMM vs mean difference) be compared on the same ordinal annotation scale.

## Related tools

- **Python** (Implements decile binning, per-decile aggregation, OLS regression fitting, and writes the gene_decile_regression_stats.tsv summary table)
- **ctmm** (Produces the per-gene CTMM cell type-specificity measures that serve as one of the two response measures in the decile regressions) — https://github.com/Minhui-Chen/ctmm_python
- **Snakemake** (Orchestrates the analysis pipeline steps that run the regression workflow) — https://github.com/Minhui-Chen/CTMM

## Evaluation signals

- OLS of the mean-difference measure vs LOEUF deciles should yield p ~ 1.0e-3, matching the reported result that more-constrained genes have lower mean differences
- OLS of the CTMM cell type-specificity measure vs LOEUF deciles should yield p ~ 1.2e-8, with the opposite direction to the mean-difference association
- Each annotation (EDS, enhancer count, LOEUF, pLI) should contain 10 equal-size decile bins covering all genes
- gene_decile_regression_stats.tsv must contain one row per annotation-measure pair (8 pairs) with coefficient, standard error, and p-value columns
- Both CTMM and mean-difference measures should show enrichment in genes with larger enhancer domains when plotted/regressed against EDS deciles

## Limitations

- Regression on 10 decile means reduces effective sample size to the number of bins, discarding within-decile variation
- CTMM's cell type-specificity measure and mean-difference measures relate to LOEUF in opposite directions (p = 1.2e-8 vs p = 1.0e-3), so the choice of variance measure changes the biological interpretation of constraint associations
- CTMM-derived measures inherit upstream caveats — CTMM is not reliable for sample sizes below ~50 individuals and requires >10 cells per individual-cell type pair, limiting it to common cell types

## Evidence

- [methods] Fit ordinary least-squares (OLS) regressions of the per-decile mean-difference measure and the per-decile CTMM measure on the annotation decile ranks, one regression per annotation-measure pair.: [evidence span withheld — anchor status: not_found]
- [methods] Rank all genes and assign each to one of 10 equal-size decile bins independently for each of the four annotations (EDS, enhancer count, LOEUF, pLI).: [evidence span withheld — anchor status: not_found]
- [results] Both CTMM and mean-difference signals are enriched in genes with larger enhancer domains; more-constrained genes have lower mean differences across cell types (p = 1.0e-3), while CTMM's cell type-specificity measure correlates with LOEUF in the opposite direction (p = 1.2e-8).: [evidence span withheld — anchor status: not_found]
