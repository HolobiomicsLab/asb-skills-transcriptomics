---
name: cell-variance-estimation-and-variance-normalization
description: Use when preparing scRNA-seq UMI count data for Normalisr association
  testing and mean-level (lcpm/normcov) normalization is done but per-cell variance
  effects of library-size covariates still confound expression.
when_to_use_negative:
- Data comes from full-length (non-UMI) platforms — the nonlinear cellular summary
  covariates were only validated on UMI-based datasets and 'may benefit from revalidating'
  there
- The goal is hypothesis testing of arbitrary nonlinear associations — 'Normalisr
  is not designed for hypothesis testing of arbitrary nonlinear associations'
- Integrative analysis across multiple datasets requiring explicit batch-effect correction
  — batch effects are not addressed by Normalisr
edam_operation: http://edamontology.org/operation_2990
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: Normalisr
  role: Provides normalisr.norm's compute_var for per-cell variance estimation and
    normvar for variance-level normalization with covariate regression; installed
    via pip install normalisr
  repo: github.com/lingfeiwang/normalisr
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-26682-1
    title: Single-cell normalization and association testing unifying CRISPR screen
      and gene co-expression analyses with Normalisr
  source_content_hash: sha256:2f1ba81be88ef29a14f9697f26e6ba7dff7825b7b2f465bbccce3b0f16bbddf0
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Normalisr
  edam_operation: http://edamontology.org/operation_2990
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8333333333333334
n_claims: 6
---

# cell-variance-estimation-and-variance-normalization

## Summary

Normalisr's variance-level normalization: estimate per-cell variance of the Bayesian log expression, then normalize gene expression variances with normvar to regress out variance-level confounding effects of nonlinear cellular covariates before association testing.

## When to use

Use when preparing scRNA-seq UMI count data for Normalisr association testing and mean-level (lcpm/normcov) normalization is done but per-cell variance effects of library-size covariates still confound expression.

## When NOT to use

- Data comes from full-length (non-UMI) platforms — the nonlinear cellular summary covariates were only validated on UMI-based datasets and 'may benefit from revalidating' there
- The goal is hypothesis testing of arbitrary nonlinear associations — 'Normalisr is not designed for hypothesis testing of arbitrary nonlinear associations'
- Integrative analysis across multiple datasets requiring explicit batch-effect correction — batch effects are not addressed by Normalisr

## Inputs

- Bayesian log expression matrix (output of lcpm/scaling_factor on the raw UMI read count matrix, e.g. GSE120861 highmoi 30 gRNAs / ~7000 cells)
- Selected nonlinear cellular summary covariates from normcov (log total read count, its square, number of zero-read genes)
- Raw UMI read count matrix (tsv or Cell Ranger sparse mtx)

## Outputs

- Per-cell variance estimates from compute_var
- Variance-normalized expression matrix (e.g. normalized_expression.tsv)

## How to apply

After computing Bayesian log expression via lcpm/scaling_factor and selecting nonlinear cellular summary covariates (log total read count, its square, and number of genes with zero reads) with normcov, call normalisr.norm's compute_var to estimate a variance for each cell. Then run normvar to normalize gene expression variances, linearly regressing out the confounding effects of the selected nonlinear cellular covariates at the log variance level; mean-level confounding is deliberately left for the final linear association hypothesis testing. The pipeline also supports iterative outlier cell removal in which the inverse of the estimated cell variance is modeled with a normal distribution, removing low-variance outlier cells with Bonferroni P ≤ 10−10 under a 2% prior bound on the outlier proportion. Because Normalisr is non-parametric, the only adjustable parameters here relate to quality control and outlier cutoffs.

## Related tools

- **Normalisr** (Provides normalisr.norm's compute_var for per-cell variance estimation and normvar for variance-level normalization with covariate regression; installed via pip install normalisr) — github.com/lingfeiwang/normalisr

## Examples

```
import normalisr.normalisr as norm; var = norm.compute_var(...); norm.normvar(...)
```

## Evaluation signals

- normvar completes on the lcpm-normalized matrix without error and outputs a matrix of the same genes × cells dimensions
- Variance-level covariate effects of the selected covariates (log total read count, its square, zero-read gene count) are regressed out at the log variance level
- Outlier removal (if run) removes only low-variance outlier cells with Bonferroni P ≤ 10−10 under the 2% prior bound
- Downstream association testing yields unbiased p-value estimation — the paper's stated criterion that confounders were correctly removed at both mean and variance levels

## Limitations

- Validated only on UMI-based datasets; full-length platforms may produce a different read count distribution requiring revalidation of the covariates
- Mean-level confounding is not removed here — it is accounted for in the final linear association hypothesis testing, so residual mean effects are expected after this step
- Only adjustable parameters are for quality control and final differential/co-expression cutoffs (Normalisr is non-parametric)

## Evidence

- [discussion] the confounding effects of the selected nonlinear cellular covariates are first linearly regressed out at the log variance level: [evidence span withheld — anchor status: not_found]
- [discussion] Covariates may confound (Bayesian log) expression at the variance level in addition to the mean: "Covariates may confound (Bayesian log) expression at the variance level in addition to the mean"
- [discussion] inverse of (estimated) cell variance was modeled with normal distribution: "For outlier removal, inverse of (estimated) cell variance was modeled with normal distribution"
- [methods] Normalization autosummary listing lcpm, normcov, scaling_factor, compute_var, normvar: "lcpm
	normcov
	scaling_factor
	compute_var
	normvar"
- [abstract] detecting and removing nonlinear confounders arising from library size at mean and variance levels: "By systematically detecting and removing nonlinear confounders arising from library size at mean and variance levels"
