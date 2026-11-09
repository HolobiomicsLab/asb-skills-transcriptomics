---
name: single-cell-umi-count-normalization
description: Use when raw UMI read count matrices from scRNA-seq (e.g., GSE120861
  highmoi Perturb-seq) must be normalized at both mean and variance levels before
  linear association testing such as DE, CRISPR screen, or co-expression inference.
when_to_use_negative:
- 'The data are full-length (non-UMI) platform reads: only UMI-based datasets were
  analyzed and the nonlinear cellular summary covariates would require revalidation'
- The goal is hypothesis testing of arbitrary nonlinear associations, not linear association
  testing after library-size confounder removal
- An integrative multi-dataset study where unaddressed batch effects dominate, which
  Normalisr does not explicitly handle
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: Normalisr
  role: Implements lcpm, scaling_factor, normcov, compute_var, and normvar functions
    for Bayesian log expression, covariate selection, variance estimation, and variance
    normalization
  repo: github.com/lingfeiwang/normalisr
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-26682-1
    title: Single-cell normalization and association testing unifying CRISPR screen
      and gene co-expression analyses with Normalisr
  source_content_hash: sha256:83cd62cf426c6e0acdb9295316d4f7546b7912af2f7ab4da8589495121f17d6e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Normalisr
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8181818181818182
n_claims: 11
---

# single-cell-umi-count-normalization

## Summary

Normalize raw scRNA-seq UMI read counts with Normalisr by computing Bayesian (MMSE digamma-based) log expression, selecting nonlinear cellular covariates (log total read count, its square, zero-read gene count) via restricted forward stepwise selection, and regressing out variance-level confounding with per-cell variance normalization.

## When to use

Use when raw UMI read count matrices from scRNA-seq (e.g., GSE120861 highmoi Perturb-seq) must be normalized at both mean and variance levels before linear association testing such as DE, CRISPR screen, or co-expression inference.

## When NOT to use

- The data are full-length (non-UMI) platform reads: only UMI-based datasets were analyzed and the nonlinear cellular summary covariates would require revalidation
- The goal is hypothesis testing of arbitrary nonlinear associations, not linear association testing after library-size confounder removal
- An integrative multi-dataset study where unaddressed batch effects dominate, which Normalisr does not explicitly handle

## Inputs

- Raw scRNA-seq UMI read count matrix (tsv, gzipped tsv, or Cell Ranger sparse mtx)
- Cell and gene name files (one per line)
- gRNA assignment matrix (for CRISPR screen context, e.g., GSE120861 highmoi subset of 30 gRNAs and ~7000 cells)

## Outputs

- Bayesian log expression matrix (MMSE posterior estimate of log relative gene expression)
- Per-cell variance estimates (compute_var output)
- Variance-normalized expression matrix (e.g., normalized_expression.tsv)
- Selected nonlinear cellular covariates from restricted forward stepwise selection

## How to apply

Load the raw UMI count matrix (tsv or Cell Ranger sparse mtx) and quality-control it, e.g., cells with at least 500 reads and 100 genes with non-zero reads, genes non-zero in at least 50 cells and 2% of cells for 10x data. Compute the Bayesian log expression using Normalisr's lcpm/scaling_factor functions, which apply the MMSE digamma-based posterior estimator for log relative gene expression. Run restricted forward stepwise selection of nonlinear cellular summary covariates (log total read count, its square, and the number of genes with zero reads) via normalisr.norm's normcov to remove mean-level library-size confounding. Estimate per-cell variance with compute_var, remove low-variance outlier cells modeled on inverse cell variance (prior bound 2%, Bonferroni P ≤ 10−10), then apply normvar to regress variance-level covariate effects out of gene expression variances. The rationale is that library size produces nonlinear confounding at both mean and variance levels, which must be removed for unbiased p-value estimation in the downstream linear association step.

## Related tools

- **Normalisr** (Implements lcpm, scaling_factor, normcov, compute_var, and normvar functions for Bayesian log expression, covariate selection, variance estimation, and variance normalization) — github.com/lingfeiwang/normalisr

## Examples

```
import normalisr.normalisr as norm; ?norm.de
```

## Evaluation signals

- The output matrix contains Bayesian log expression (digamma-based MMSE estimates) rather than naive log-CPM values
- Selected covariates from normcov include the expected nonlinear terms: log total read count, its square, and the number of genes with zero reads
- Low-variance outlier cells were iteratively removed with prior outlier proportion bound 2% and Bonferroni P ≤ 10−10
- QC thresholds were enforced: cells with ≥500 reads and ≥100 genes with non-zero reads; genes non-zero in ≥50 cells (10x) and ≥2% of cells
- Downstream association testing on the normalized matrix yields unbiased p-value estimation (uniform p-values under null/non-targeting controls)

## Limitations

- Only UMI-based datasets were validated; full-length platforms may produce a different read count distribution and require revalidating the nonlinear cellular summary covariates
- Normalisr is not designed for hypothesis testing of arbitrary nonlinear associations
- Batch effects in multi-dataset integrative studies are not addressed and may require additional consideration
- Existing single-cell methods like scVI do not support P value-based FDR estimation, and ScImpute/ZINB-WaVE cannot accommodate large data sizes, so Normalisr's normalization is positioned specifically for scalable association testing

## Evidence

- [abstract] By systematically detecting and removing nonlinear confounders arising from library size at mean and variance levels, Normalisr achieves high sensitivity, specificity, speed, and generalizability: "By systematically detecting and removing nonlinear confounders arising from library size at mean and variance levels, Normalisr achieves high sensitivity, specificity, speed, and generalizability"
- [discussion] The MMSE estimator for log relative gene expression (here named Bayesian log expression) is yij≡Elng~ij: [evidence span withheld — anchor status: not_found]
- [discussion] we restricted our analyses to cells with at least 500 reads and 100 genes with non-zero reads: "we restricted our analyses to cells with at least 500 reads and 100 genes with non-zero reads"
- [methods] lcpm normcov scaling_factor compute_var normvar: "Normalization
==============
.. autosummary::
	:toctree:
	:nosignatures:

	lcpm
	normcov
	scaling_factor
	compute_var
	normvar"
- [readme] Normalisr uses tsv (tab separated values) file format for input and output matrices: [evidence span withheld — anchor status: external_doc]
