---
name: technical-confounder-regression-scrna-seq
description: Use when raw UMI scRNA-seq count matrices must be normalized for downstream
  association testing (DE, CRISPR screen, co-expression) and nonlinear library-size
  effects need removal at mean and variance levels.
when_to_use_negative:
- Hypothesis testing of arbitrary nonlinear associations between covariates and expression
  — Normalisr is explicitly not designed for this
- Full-length (non-UMI) scRNA-seq platforms — only UMI-based datasets were validated
  and nonlinear cellular summary covariates require revalidation there
- Integrative studies spanning multiple datasets where batch effects need explicit
  correction — Normalisr does not address batch effects
edam_operation: http://edamontology.org/operation_3435
edam_topics:
- http://edamontology.org/topic_3170
- http://edamontology.org/topic_0203
tools:
- name: Normalisr
  role: Implements lcpm/scaling_factor (MMSE Bayesian log expression), normcov (restricted
    forward stepwise covariate selection), compute_var (per-cell variance), and normvar
    (variance normalization)
  repo: https://github.com/lingfeiwang/normalisr
- name: sctransform
  role: Alternative mean-level normalization method cited for comparison, not used
    in this skill
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1038/s41467-021-26682-1
    title: Single-cell normalization and association testing unifying CRISPR screen
      and gene co-expression analyses with Normalisr
  source_content_hash: sha256:c9016460cb30b2819d83dc9553db025d227e25c2f72fd77c2da072584540fefa
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/lingfeiwang/normalisr
  tools:
  - Normalisr
  - sctransform
  edam_operation: http://edamontology.org/operation_3435
  edam_topics:
  - http://edamontology.org/topic_3170
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# technical-confounder-regression-scrna-seq

## Summary

Normalisr's normalization strategy for scRNA-seq: detect and remove nonlinear library-size confounders from UMI read counts at both the mean (Bayesian log expression via MMSE estimator plus normcov covariate selection) and variance (compute_var plus normvar) levels before linear association testing.

## When to use

Use when raw UMI scRNA-seq count matrices must be normalized for downstream association testing (DE, CRISPR screen, co-expression) and nonlinear library-size effects need removal at mean and variance levels.

## When NOT to use

- Hypothesis testing of arbitrary nonlinear associations between covariates and expression — Normalisr is explicitly not designed for this
- Full-length (non-UMI) scRNA-seq platforms — only UMI-based datasets were validated and nonlinear cellular summary covariates require revalidation there
- Integrative studies spanning multiple datasets where batch effects need explicit correction — Normalisr does not address batch effects

## Inputs

- Raw scRNA-seq UMI read count matrix (tsv or Cell Ranger sparse mtx)
- Cell and gene name lists (one per line, text files)
- gRNA assignment matrix (for CRISPR screen mode, e.g. GSE120861 highmoi)

## Outputs

- Variance-normalized Bayesian log expression matrix (e.g. normalized_expression.tsv)
- Per-cell variance estimates from compute_var

## How to apply

Load the raw UMI count matrix (e.g. GSE120861 highmoi subset of 30 gRNAs and ~7000 cells, tsv or Cell Ranger sparse mtx). Compute Bayesian log expression with Normalisr's lcpm/scaling_factor functions, which apply the MMSE (digamma-based) posterior estimator for log relative gene expression. Run restricted forward stepwise selection of nonlinear cellular summary covariates (log total read count, its square, and the number of genes with zero reads) via normcov to remove mean-level confounding. Estimate per-cell variance with compute_var, then apply normvar to regress out variance-level covariate effects; outlier cells are removed iteratively by modeling inverse cell variance as normal, with prior outlier proportion bound 2% and Bonferroni P ≤ 10^-10. Mean-level effects of selected nonlinear covariates are deliberately left to be accounted for in the final linear association hypothesis testing rather than fully regressed out here. Output the variance-normalized expression matrix (e.g. normalized_expression.tsv).

## Related tools

- **Normalisr** (Implements lcpm/scaling_factor (MMSE Bayesian log expression), normcov (restricted forward stepwise covariate selection), compute_var (per-cell variance), and normvar (variance normalization)) — https://github.com/lingfeiwang/normalisr
- **sctransform** (Alternative mean-level normalization method cited for comparison, not used in this skill)

## Examples

```
import normalisr.normalisr as norm; ?norm.de
```

## Evaluation signals

- Output matrix rows/columns match input genes and retained cells; format is tsv with gzipped files auto-recognized by '.gz' suffix
- Nonlinear covariates selected by normcov are limited to the restricted set: log total read count, its square, and number of genes with zero reads
- Outlier removal respects prior bound of 2% outlier proportion and Bonferroni P ≤ 10^-10 cutoff
- P values from downstream linear association tests remain unbiased (uniform under the null), the stated design goal of two-level normalization
- Normalization runs parameter-free except QC thresholds (e.g. cells with ≥500 reads and 100 non-zero genes; genes non-zero in ≥50 cells and ≥2% of cells)

## Limitations

- Only UMI-based datasets analyzed; full-length platforms not validated and covariates may need revalidation
- Not designed for hypothesis testing of arbitrary nonlinear associations
- Does not reconstruct causal regulatory networks or handle batch effects in multi-dataset integrative studies
- Robustness against read count distribution distortions requires enough (>100) cells

## Evidence

- [abstract] By systematically detecting and removing nonlinear confounders arising from library size at mean and variance levels: "By systematically detecting and removing nonlinear confounders arising from library size at mean and variance levels, Normalisr achieves high sensitivity, specificity, speed"
- [discussion] Restricted forward stepwise selection: "We established a restricted forward stepwise selection process to systematically dissect nonlinear effects of known confounders"
- [discussion] MMSE Bayesian log expression: [evidence span withheld — anchor status: not_found]
- [discussion] Outlier removal thresholds: "With the prior bound for outlier proportion as 2%, low-variance outlier cells with Bonferroni P ≤ 10−10 were removed"
- [methods] Normalization function autosummary: "lcpm
	normcov
	scaling_factor
	compute_var
	normvar"
- [readme] README usage notes: [evidence span withheld — anchor status: external_doc]
