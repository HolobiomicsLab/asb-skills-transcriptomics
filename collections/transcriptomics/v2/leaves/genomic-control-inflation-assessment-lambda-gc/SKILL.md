---
name: genomic-control-inflation-assessment-lambda-gc
description: Use when benchmarking DE or gene set enrichment methods on scRNA-seq
  summary statistics and you need to check p-value calibration via a genomic control
  factor (lambda_gc), as in the iDEA paper's Fig. 4 comparison.
when_to_use_negative:
- When the summary statistics come from inferred cell types whose DE test statistics
  may be inflated with artificially smaller standard errors — inflation may reflect
  a modeling artifact rather than method miscalibration
- When you only need enrichment power (gene set counts at FDR 5%) without any p-value
  calibration assessment
edam_operation: http://edamontology.org/operation_3223
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: iDEA
  role: Fits the hierarchical Bayesian model and produces GSE/DE statistics whose
    calibration is assessed by lambda_gc; iDEA.fit.null and iDEA.FDR supply the permuted-null
    FDR against which calibration is compared
  repo: https://github.com/xzhoulab/iDEA
- name: zingeR
  role: Generates the per-gene DE summary statistics (beta, beta variance) that feed
    the lambda_gc computation for each compared method
  repo: https://github.com/statOmics/zinbwaveZinger
- name: fGSEA
  role: Competing GSE method whose lambda_gc and FDR-5% gene set counts are compared
    against iDEA
- name: CAMERA
  role: Competing GSE method included in the lambda_gc calibration benchmark
- name: PAGE
  role: Competing GSE method included in the lambda_gc calibration benchmark
- name: GSEA
  role: Competing GSE method included in the lambda_gc calibration benchmark
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1038/s41467-020-15298-6
    title: Integrative differential expression and gene set enrichment analysis using
      summary statistics for scRNA-seq studies
  source_content_hash: sha256:c6315da6ed02e5d6be265403653f6b414612ca307faab79fd6bb6eafa2d91393
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
  - zingeR
  edam_operation: http://edamontology.org/operation_3223
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 12
---

# Genomic control inflation assessment (lambda_gc) for scRNA-seq DE/GSE p-values

## Summary

Computes the lambda_gc genomic control factor from DE or GSE summary statistics to quantify test-statistic inflation when benchmarking scRNA-seq methods such as iDEA, fGSEA, CAMERA, PAGE, GSEA and zingeR on the Chu et al. human ESC (DEC vs EC) dataset.

## When to use

Use when benchmarking DE or gene set enrichment methods on scRNA-seq summary statistics and you need to check p-value calibration via a genomic control factor (lambda_gc), as in the iDEA paper's Fig. 4 comparison.

## When NOT to use

- When the summary statistics come from inferred cell types whose DE test statistics may be inflated with artificially smaller standard errors — inflation may reflect a modeling artifact rather than method miscalibration
- When you only need enrichment power (gene set counts at FDR 5%) without any p-value calibration assessment

## Inputs

- Per-gene DE summary statistics from zingeR-DESeq2 (beta coefficient, beta variance) or z-scores derived from p-values and LogFC
- Per-method gene set enrichment p-values or test statistics (iDEA, fGSEA, CAMERA, PAGE, GSEA, hypergeometric/Fisher)
- Per-gene DE p-values or z-scores from each DE method being compared

## Outputs

- Per-method lambda_gc genomic control factor (scalar per DE or GSE method)
- Calibration column in idea_dec_ec_results.csv alongside FDR-based gene set counts, DE gene counts and Jaccard overlaps

## How to apply

After generating per-gene DE summary statistics (e.g., zingeR-DESeq2 beta coefficients and beta variances, or z-scores via qnorm(p) with se = |beta/zscore|) and running each GSE method, collect the resulting test statistics or p-values per method. Compute lambda_gc as the ratio of the median of the observed chi-square statistics (or squared z-scores) to the expected median of the chi-square distribution with 1 degree of freedom (approximately 0.4549), as reported for each competing method. In the DEC vs EC human ESC comparison, the paper expects lambda_gc values around 1.13, 1.02, 0.96, 1.01, 0.29 for the compared methods, providing a quantitative calibration check alongside empirical FDR (permuted via iDEA.fit.null and iDEA.FDR) gene-set counts at 5% FDR and DE gene counts at 1% FDR. Write per-method lambda_gc values into the results table (idea_dec_ec_results.csv) alongside FDR-based counts and Jaccard overlaps so calibration and power can be jointly judged.

## Related tools

- **iDEA** (Fits the hierarchical Bayesian model and produces GSE/DE statistics whose calibration is assessed by lambda_gc; iDEA.fit.null and iDEA.FDR supply the permuted-null FDR against which calibration is compared) — https://github.com/xzhoulab/iDEA
- **zingeR** (Generates the per-gene DE summary statistics (beta, beta variance) that feed the lambda_gc computation for each compared method) — https://github.com/statOmics/zinbwaveZinger
- **fGSEA** (Competing GSE method whose lambda_gc and FDR-5% gene set counts are compared against iDEA)
- **CAMERA** (Competing GSE method included in the lambda_gc calibration benchmark)
- **PAGE** (Competing GSE method included in the lambda_gc calibration benchmark)
- **GSEA** (Competing GSE method included in the lambda_gc calibration benchmark)

## Examples

```
R: idea_obj <- iDEA.fit(CreateiDEAObject(beta, beta_var, annotation, max_var_beta=100, min_precent_annot=0.0025, num_core=10), init_tau=c(-2,0.5), min_degene=5, em_iter=15, mcmc_iter=1000, fit.tol=1e-5, modelVariant=FALSE); lambda_gc <- median(idea_obj@zscore^2)/qchisq(0.5,1)
```

## Evaluation signals

- lambda_gc values fall within reasonable numerical tolerance of the published Fig. 4 values (1.13, 1.02, 0.96, 1.01, 0.29) for the DEC vs EC comparison
- Genomic control factor near 1.0 indicates calibrated p-values; large deviations (e.g., hypergeometric ~0.29 or >1.1) flag miscalibration consistent with the paper's findings
- Calibration column is populated for every method in idea_dec_ec_results.csv and pairs with the FDR-5% gene set counts and FDR-1% DE gene counts
- Jaccard overlap trajectory across methods (0.10 -> 0.25) remains consistent when recomputed alongside lambda_gc
- Empirical FDR (via iDEA.fit.null / iDEA.FDR gene-label permutation) is used alongside lambda_gc rather than replacing it

## Limitations

- Genomic control assumes p-values are on a common scale across genes; z-score modeling in iDEA effectively assumes the prior distribution of true effect sizes depends on the standard errors and sample size, which can bias calibration estimates
- DE analysis on inferred cell types may lead to inflated DE test statistics with artificially smaller standard errors, distorting lambda_gc
- iDEA does not explicitly account for gene set overlap that may cause non-independence among gene sets; genomic control alone does not correct for this non-independence — permutation-based FDR (iDEA.fit.null/iDEA.FDR) is also required

## Evidence

- [results] Genomic control lambda_gc values reported for DEC vs EC human ESC comparison: [evidence span withheld — anchor status: not_found]
- [discussion] iDEA provides calibrated p-values consistent with other non-correlation-aware GSE methods: "GSE methods that do not explicitly account for gene-gene correlation (e.g. iDEA, PAGE, fGSEA and GSEA) appear to provide more calibrated p-values"
- [methods] Permutation-based FDR used in conjunction with genomic control: "The function iDEA.FDR calculate the estimated FDR for each gene set given the permuted null distribution of pvalues for gene sets."
- [readme] iDEA is installed and applied via the R package: "devtools::install_github('xzhoulab/iDEA')"
- [intro] Summary-statistics-based GSE is less susceptible to correlations and dropout, motivating calibration checks: "incorporating summary statistics from scRNA-seq DE analysis into GSE analysis under the framework of iDEA makes GSE analysis less susceptible to gene-gene correlations and other technical"
