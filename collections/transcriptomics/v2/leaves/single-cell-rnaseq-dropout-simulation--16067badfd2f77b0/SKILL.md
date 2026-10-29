---
name: single-cell-rnaseq-dropout-simulation
description: 'Use when assessing robustness of bulk deconvolution (MuSiC) to zero-inflation
  in the single-cell reference: apply a dropout grid to a scRNA-seq ExpressionSet,
  re-estimate proportions with music_prop, and score against real proportions.'
when_to_use_negative:
- Bulk and single-cell data are matched count data with no reference sparsity concern
  — dropout simulation adds artificial bias rather than information
- Expression values are TPM/RPKM rather than counts — cell type proportions cannot
  be estimated with TPM as the input, and dropout models calibrated on counts would
  not apply
- The goal is absolute cell type proportions rather than robustness assessment — under
  the model assumptions only trends across subjects are recoverable
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: MuSiC
  role: Provides music_prop for weighted deconvolution at each dropout level, bulk_construct
    and relative.ab for the artificial bulk benchmark, and Eval_multi for RMSD/mAD/R
    scoring
  repo: https://github.com/xuranw/MuSiC
- name: R
  role: Implementation environment; MuSiC is an R/Bioconductor-style package installed
    from GitHub via devtools
  repo: https://github.com/xuranw/MuSiC
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1038/s41467-018-08023-x
    title: Bulk tissue cell type deconvolution with multi-subject single-cell expression
      reference
  source_content_hash: sha256:403e32683de626705b49a38b37c4a3483408ba4f448a80637e739f3a9f00cc62
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/xuranw/MuSiC
  tools:
  - MuSiC
  - R
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 14
---

# single-cell-rnaseq-dropout-simulation

## Summary

Benchmarking protocol that artificially zero-inflates a single-cell RNA-seq reference (via the Jia et al. dropout scheme) and tests whether MuSiC's cell type deconvolution stays accurate as dropout increases, reproducing the paper's Supplementary Note 5 result that MuSiC remains accurate at ~30% dropout.

## When to use

Use when assessing robustness of bulk deconvolution (MuSiC) to zero-inflation in the single-cell reference: apply a dropout grid to a scRNA-seq ExpressionSet, re-estimate proportions with music_prop, and score against real proportions.

## When NOT to use

- Bulk and single-cell data are matched count data with no reference sparsity concern — dropout simulation adds artificial bias rather than information
- Expression values are TPM/RPKM rather than counts — cell type proportions cannot be estimated with TPM as the input, and dropout models calibrated on counts would not apply
- The goal is absolute cell type proportions rather than robustness assessment — under the model assumptions only trends across subjects are recoverable

## Inputs

- Single-cell reference count matrix as ExpressionSet (EMTABesethealthy.rds, Segerstolpe E-MTAB-5060) with 'cellType' and 'sampleID' pheno columns
- Single-cell dataset for artificial bulk construction (XinT2Deset.rds, GSE81608)
- Grid of dropout rates spanning low to high values, including ~0.3

## Outputs

- Zero-inflated (dropout-modified) EMTAB reference ExpressionSet per dropout level
- MuSiC estimated cell type proportions per dropout level (Est.prop.weighted)
- Metrics table dropout_benchmark_results.csv with columns dropout_rate, RMSD, mAD, R

## How to apply

Load the Segerstolpe healthy-donor reference (EMTABesethealthy.rds) and the Xin et al. dataset (XinT2Deset.rds) as ExpressionSet objects, then build an artificial bulk benchmark with bulk_construct over the 4 endocrine cell types (alpha, beta, delta, gamma), computing real proportions via relative.ab(num.real, by.col=FALSE). For each dropout rate in a grid spanning low to high values (including the ~0.3 rate), apply the Jia et al. dropout scheme to the EMTAB reference count matrix to produce a zero-inflated reference. Run music_prop(bulk.eset=Bulk.counts, sc.eset=dropout-modified EMTAB.eset, clusters='cellType', samples='sampleID', select.ct=c('alpha','beta','delta','gamma')) at each level. Score against real proportions with Eval_multi recording RMSD, mAD and Pearson R, and write the (dropout_rate, RMSD, mAD, R) table to dropout_benchmark_results.csv. Rationale: MuSiC's gene weighting (W-NNLS) should make deconvolution robust to reference sparsity, so metrics should stay near the zero-dropout baseline (RMSD 0.09881, mAD 0.06357, R 0.9378) up to ~30% dropout.

## Related tools

- **MuSiC** (Provides music_prop for weighted deconvolution at each dropout level, bulk_construct and relative.ab for the artificial bulk benchmark, and Eval_multi for RMSD/mAD/R scoring) — https://github.com/xuranw/MuSiC
- **R** (Implementation environment; MuSiC is an R/Bioconductor-style package installed from GitHub via devtools) — https://github.com/xuranw/MuSiC

## Examples

```
library(MuSiC); Est.prop.Xin <- music_prop(bulk.eset = Bulk.counts, sc.eset = EMTAB.eset.dropout, clusters = 'cellType', samples = 'sampleID', select.ct = c('alpha','beta','delta','gamma'))
```

## Evaluation signals

- RMSD and mAD at ~30% dropout stay close to the zero-dropout baseline (RMSD 0.09881, mAD 0.06357, R 0.9378), reproducing Supplementary Figure 8a-b
- Pearson correlation R between estimated and real proportions remains high (baseline R = 0.9378) across the low-to-moderate dropout grid
- Estimated proportions sum to 1 for every bulk sample at every dropout level (NNLS simplex constraint)
- Results table dropout_benchmark_results.csv contains one row per dropout rate with non-missing dropout_rate, RMSD, mAD and R columns
- Zero-dropout run (no dropout applied) reproduces the reported XinT2D benchmark metrics before any degradation should appear

## Limitations

- Robustness is verified only up to ~30% dropout; accuracy beyond that rate is not guaranteed
- Benchmark is limited to the 4 endocrine cell types (alpha, beta, delta, gamma) of pancreatic islets; closely related cell types are harder to resolve due to collinearity
- Most scRNA-seq data do not reflect true cell type proportions in intact tissues, so the artificial bulk benchmark reflects simulated rather than biological composition
- Cell type proportions cannot be estimated with TPM as the input; count-level inputs are required

## Evidence

- [methods] artificial bulk benchmark: "Benchmark dataset is constructed by summing up single cell data from `XinT2D.eset`."
- [methods] music_prop estimation: "The cell type proportions is estimated by function `music_prop`"
- [methods] real proportions computation: "XinT2D.construct.full$prop.real = relative.ab(XinT2D.construct.full$num.real, by.col = FALSE)"
- [supplementary] dropout robustness finding: [evidence span withheld — anchor status: not_found]
- [readme] R package installation: "devtools::install_github('xuranw/MuSiC')"
- [discussion] TPM limitation: [evidence span withheld — anchor status: not_found]
