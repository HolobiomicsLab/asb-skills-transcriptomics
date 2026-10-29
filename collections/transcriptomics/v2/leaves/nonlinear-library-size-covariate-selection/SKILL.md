---
name: nonlinear-library-size-covariate-selection
description: Use when raw scRNA-seq UMI count matrices must be normalized so that
  library-size-dependent nonlinear confounding is detected and regressed out prior
  to linear association testing (DE, CRISPR screen, co-expression).
when_to_use_negative:
- Input is already normalized or is not UMI-based scRNA-seq (the paper only focused
  on UMI-based datasets; full-length platforms may need revalidated covariates)
- The goal is hypothesis testing of arbitrary nonlinear associations between covariates
  and expression
- Integrative analysis across multiple datasets where batch effects, which Normalisr
  does not address, dominate
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Normalisr
  role: Provides the normalization functions lcpm, scaling_factor, normcov, compute_var,
    and normvar used for Bayesian log expression, covariate selection, and variance
    normalization
  repo: github.com/lingfeiwang/normalisr
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-26682-1
    title: Single-cell normalization and association testing unifying CRISPR screen
      and gene co-expression analyses with Normalisr
  source_content_hash: sha256:34789b7baf705c11be0d5b504872a7b0c4310e63a344e9a55cda25468bc9a86d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Normalisr
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# nonlinear-library-size-covariate-selection

## Summary

Restricted forward stepwise selection of nonlinear cellular summary covariates (log total read count, its square, number of genes with zero reads) in Normalisr, to detect and remove library-size confounding from scRNA-seq Bayesian log expression at both mean and variance levels before association testing.

## When to use

Use when raw scRNA-seq UMI count matrices must be normalized so that library-size-dependent nonlinear confounding is detected and regressed out prior to linear association testing (DE, CRISPR screen, co-expression).

## When NOT to use

- Input is already normalized or is not UMI-based scRNA-seq (the paper only focused on UMI-based datasets; full-length platforms may need revalidated covariates)
- The goal is hypothesis testing of arbitrary nonlinear associations between covariates and expression
- Integrative analysis across multiple datasets where batch effects, which Normalisr does not address, dominate

## Inputs

- Raw scRNA-seq UMI read count matrix (tsv or Cell Ranger sparse mtx, optionally gzipped), e.g. GSE120861 highmoi subset (30 gRNAs, ~7000 cells)
- Cell and gene name files, one per line
- Optional gRNA assignment matrix for downstream association testing

## Outputs

- Covariate-selected, variance-normalized Bayesian log expression matrix (e.g. normalized_expression.tsv)
- Per-cell variance estimates from compute_var

## How to apply

Load the raw UMI read count matrix (e.g. the GSE120861 highmoi subset of 30 gRNAs and ~7000 cells) and compute the Bayesian log expression with Normalisr's lcpm and scaling_factor functions, which apply the MMSE (digamma-based) posterior estimator for log relative gene expression. Then run normalisr.norm's normcov, which performs a restricted forward stepwise selection over candidate nonlinear cellular summary covariates (log total read count, its square, and the number of genes with zero reads) to remove mean-level library-size confounding while keeping covariates fixed-effect-adjustable. Estimate per-cell variance with compute_var, then apply normvar to regress variance-level covariate effects out of gene expression variances; per the article's architecture, mean-level confounding of the selected covariates is instead accounted for later in the linear association hypothesis testing. The rationale is non-parametric normalization: aside from QC and final cutoffs, Normalisr has no adjustable parameters, and systematic nonlinear confounder removal yields unbiased P-value estimation with high sensitivity and specificity.

## Related tools

- **Normalisr** (Provides the normalization functions lcpm, scaling_factor, normcov, compute_var, and normvar used for Bayesian log expression, covariate selection, and variance normalization) — github.com/lingfeiwang/normalisr

## Examples

```
pip install normalisr; python3 -m normalisr -h
```

## Evaluation signals

- normcov output should show a restricted forward stepwise selection path over log total read count, its square, and the number of genes with zero reads
- Final expression matrix is saved in tsv format with row/column names consistent with the input cell and gene lists
- Downstream association P-values should be unbiased (uniform under the null, e.g. for non-targeting controls / NTCs)
- Pipeline runs parameter-free: only QC thresholds (e.g. cells with at least 500 reads and 100 genes with non-zero reads) and final DE/co-expression cutoffs are adjustable
- Variance normalization should precede association testing; variance-level covariate effects must be regressed out via normvar before linear models are fit

## Limitations

- Only UMI-based datasets were analyzed; full-length platforms may produce different read count distributions and require revalidating the nonlinear cellular summary covariates
- Normalisr is not designed for hypothesis testing of arbitrary nonlinear associations
- Integrative studies spanning multiple datasets may require additional consideration of batch effects, which are not addressed

## Evidence

- [abstract] By systematically detecting and removing nonlinear confounders arising from library size at mean and variance levels: "By systematically detecting and removing nonlinear confounders arising from library size at mean and variance levels"
- [discussion] We established a restricted forward stepwise selection process to systematically dissect nonlinear effects of known confounders: "We established a restricted forward stepwise selection process to systematically dissect nonlinear effects of known confounders"
- [discussion] Covariates may confound (Bayesian log) expression at the variance level in addition to the mean: "Covariates may confound (Bayesian log) expression at the variance level in addition to the mean"
- [methods] lcpm normcov scaling_factor compute_var normvar (normalization module autosummary): "lcpm
	normcov
	scaling_factor
	compute_var
	normvar"
- [results] Only a subset of 30 gRNAs and ~7000 cells are extracted from GSE120861 highmoi dataset for this demonstration.: "Only a subset of 30 gRNAs and ~7000 cells are extracted from GSE120861 highmoi dataset for this demonstration."
- [readme] Normalisr uses tsv (tab separated values) file format for input and output matrices: [evidence span withheld — anchor status: external_doc]
