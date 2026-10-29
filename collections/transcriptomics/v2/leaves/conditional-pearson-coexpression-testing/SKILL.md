---
name: conditional-pearson-coexpression-testing
description: Use when you need gene-level co-expression P values or a gene regulatory
  network from UMI-based scRNA-seq counts, especially when expression-level normalization
  bias must be avoided so null P values remain uniform across gene expression bins.
when_to_use_negative:
- You need hypothesis testing of arbitrary nonlinear associations between genes —
  Normalisr is explicitly not designed for this
- You aim to reconstruct causal regulatory networks via v-structures, regularized
  regression, or Granger causality with longitudinal data
- Input is full-length (non-UMI) scRNA-seq, where the nonlinear cellular summary covariates
  were not validated and may need revalidation
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3170
tools:
- name: Normalisr
  role: 'Provides the normalization module (normalisr.norm: lcpm, normcov, scaling_factor,
    compute_var, normvar) and the conditional Pearson co-expression test (normalisr.coex)
    used for this skill'
  repo: github.com/lingfeiwang/normalisr
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41467-021-26682-1
    title: Single-cell normalization and association testing unifying CRISPR screen
      and gene co-expression analyses with Normalisr
  source_content_hash: sha256:b73ea2c70673d9f61b80c8501858a4316e7e3b43c9822f6d8e67c4c680214958
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Normalisr
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3170
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 4
---

# Conditional Pearson co-expression testing

## Summary

Normalisr's linear-model gene co-expression test that computes conditional Pearson correlation P values between gene pairs on normalized scRNA-seq expression, after nonlinear library-size confounders have been removed at the mean (lcpm, normcov) and variance (compute_var, normvar) levels, yielding exact, unbiased null P values without permutation.

## When to use

Use when you need gene-level co-expression P values or a gene regulatory network from UMI-based scRNA-seq counts, especially when expression-level normalization bias must be avoided so null P values remain uniform across gene expression bins.

## When NOT to use

- You need hypothesis testing of arbitrary nonlinear associations between genes — Normalisr is explicitly not designed for this
- You aim to reconstruct causal regulatory networks via v-structures, regularized regression, or Granger causality with longitudinal data
- Input is full-length (non-UMI) scRNA-seq, where the nonlinear cellular summary covariates were not validated and may need revalidation

## Inputs

- Raw UMI read count matrix in tsv or Cell Ranger sparse mtx format (cells x genes)
- Normalized (Bayesian log) expression matrix produced by normalisr.norm (lcpm/normcov/compute_var/normvar)
- Optional covariate annotations (batch labels, cellular summary covariates, gRNA assignments) as fixed effects

## Outputs

- Pairwise gene co-expression P values for all gene pairs (normalisr.coex output)
- Co-expression network edge list after FDR cutoff (e.g. Q ≤ 10−15)
- Diagnostic bin-pair KS statistics/P values against uniformity (e.g. bin_pair_ks_results.csv)

## How to apply

First normalize the raw UMI count matrix with Normalisr's normalization module (lcpm to estimate Bayesian log expression, then normcov/scaling_factor, compute_var and normvar) to remove nonlinear library-size effects at mean and variance levels; outlier cells are removed iteratively with a prior outlier proportion bound of 2% and Bonferroni P ≤ 10−10. Then run the conditional Pearson co-expression test from normalisr.coex on the normalized expression matrix, passing covariates (e.g. batches, house-keeping programs, untested gRNAs) as fixed effects to be natively regressed out. The test gives exact P-value estimation without permutation, so it scales to all gene pairs; apply a FDR cutoff such as Q ≤ 10−15 to prioritize direct gene interactions when building co-expression networks. Validate unbiasedness by splitting genes into 10 mean-expression bins (55 unique bin pairs) and checking, via two-sample Kolmogorov-Smirnov tests, that null P values do not depart from the uniform [0,1] distribution in any bin pair.

## Related tools

- **Normalisr** (Provides the normalization module (normalisr.norm: lcpm, normcov, scaling_factor, compute_var, normvar) and the conditional Pearson co-expression test (normalisr.coex) used for this skill) — github.com/lingfeiwang/normalisr

## Examples

```
import normalisr.normalisr as norm; ?norm.de
```

## Evaluation signals

- Null co-expression P values on a synthetic co-expression-free dataset are approximately uniform in every expression bin pair: two-sample KS tests show no significant departure from uniformity (reproduces Figs. 2d, e)
- Normalisr's P-value uniformity outperforms log(CPM+1) normalization with or without covariates, and no imputation method fully accounts for the technical confounding
- Resulting co-expression networks recapitulate known gene functions (e.g. GO/KEGG pathway enrichment on MARS-seq melanoma data)
- Quality-control filters were applied: cells with ≥500 reads and ≥100 genes with non-zero reads; genes with non-zero reads in ≥50 cells (10x) or ≥500 cells and ≥2% of cells
- Output matrices use Normalisr's tsv format with text files of one cell/gene name per line, as expected by the command-line interface

## Limitations

- Only UMI-based datasets were analyzed; the nonlinear cellular summary covariates may need revalidation on full-length platforms with different read count distributions
- Integrative studies spanning multiple datasets may require additional consideration for batch effects, which Normalisr does not address
- ScImpute and ZINB-WaVE comparisons were excluded because they could not accommodate the data size; ScVI does not recommend P-value-based FDR estimation
- Does not reconstruct causal regulatory networks or test arbitrary nonlinear associations

## Evidence

- [abstract] On conventional scRNA-seq, Normalisr recovers gene-level co-expression networks that recapitulated known gene functions.: "On conventional scRNA-seq, Normalisr recovers gene-level co-expression networks that recapitulated known gene functions."
- [methods] lcpm, normcov, scaling_factor, compute_var, normvar: "Normalization
==============
.. autosummary::
	:toctree:
	:nosignatures:

	lcpm
	normcov
	scaling_factor
	compute_var
	normvar"
- [discussion] For co-expression network in dysfunctional T cells, we used a strong cutoff (Q ≤ 10−15) to prioritize more direct gene interactions: "For co-expression network in dysfunctional T cells, we used a strong cutoff (Q ≤ 10−15) to prioritize more direct gene interactions"
- [readme] exact P-value estimation without permutation: "linear association testing provides a unified inferential framework with several advantages: (i) exact P-value estimation without permutation, (ii) native removal of covariates"
