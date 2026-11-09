---
name: goodness-of-fit-diagnostic-visualization
description: Use when evaluating whether a zero-inflated count model (e.g., ZINB-WaVE
  on scRNA-seq counts) actually fits observed mean counts, dropout rates, and genewise
  dispersions better than simpler negative binomial or hurdle alternatives.
when_to_use_negative:
- The goal is low-dimensional signal extraction or dimensionality reduction (ZINB-WaVE's
  primary use, e.g., vs PCA/ZIFA) rather than goodness-of-fit comparison of count
  models
- The data are bulk RNA-seq or already normalized with no dropout structure, so zero-inflated
  negative binomial vs NB vs hurdle comparison is not meaningful
- Only one count model can be fit — the skill's core value is the three-way ZINB vs
  NB (edgeR) vs MAST hurdle comparison and its diagnostic plots
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: zinbwave
  role: Fit the zero-inflated negative binomial model (zinbFit) and extract estimated
    mean counts, dispersions, and zero probabilities on raw scRNA-seq counts
  repo: https://bioconductor.org/packages/zinbwave
- name: edgeR
  role: Fit a gene-by-gene negative binomial model on FQ normalized counts to obtain
    comparable mean and dispersion estimates
- name: MAST
  role: Fit the hurdle model on log2(TPM+1) transformed values as the third competing
    count model
- name: scRNAseq
  role: Provide the single-cell RNA-seq count data set (e.g., V1 data set) used for
    the model fits
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-017-02554-5
    title: A general and flexible method for signal extraction from single-cell RNA-seq
      data
  source_content_hash: sha256:39f27c065603b0f6d7b65f532881e00912a38ff9b1c16b6cdb358fe2510b683d
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bioconductor (>=3.6)
  - MAST
  - R (>=3.4)
  - edgeR
  - scRNAseq
  - zinbwave
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# goodness-of-fit-diagnostic-visualization

## Summary

A diagnostic-comparison skill for single-cell RNA-seq count models: fit a zero-inflated negative binomial (ZINB-WaVE) model alongside competing count models (edgeR negative binomial, MAST hurdle) and judge fit via estimated-vs-observed mean count, zero probability, and dispersion visualizations.

## When to use

Use when evaluating whether a zero-inflated count model (e.g., ZINB-WaVE on scRNA-seq counts) actually fits observed mean counts, dropout rates, and genewise dispersions better than simpler negative binomial or hurdle alternatives.

## When NOT to use

- The goal is low-dimensional signal extraction or dimensionality reduction (ZINB-WaVE's primary use, e.g., vs PCA/ZIFA) rather than goodness-of-fit comparison of count models
- The data are bulk RNA-seq or already normalized with no dropout structure, so zero-inflated negative binomial vs NB vs hurdle comparison is not meaningful
- Only one count model can be fit — the skill's core value is the three-way ZINB vs NB (edgeR) vs MAST hurdle comparison and its diagnostic plots

## Inputs

- Single-cell RNA-seq raw count matrix (e.g., V1 data set) loadable as a SummarizedExperiment via the scRNAseq package
- FQ (upper-quartile) normalized counts for the edgeR negative binomial fit
- log2(TPM+1) transformed values for the MAST hurdle model fit

## Outputs

- Estimated mean counts, genewise dispersions, and zero (dropout) probabilities from the ZINB (zinbFit) model
- Comparable gene-wise mean and dispersion estimates from edgeR (NB) and MAST hurdle models
- PNG diagnostic figures: estimated-vs-observed mean-count and zero-probability comparisons, mean-difference plots, and dispersion-vs-zero-frequency plots

## How to apply

Install zinbwave via BiocManager under R (>=3.4) and Bioconductor (>=3.6), then load a single-cell RNA-seq count data set such as the V1 data set. Fit the zero-inflated negative binomial model with zinbFit, including sample-level and gene-level intercepts so the cell-level intercept acts as a global-scaling normalization factor (allowing raw counts) plus genewise dispersion, and extract estimated mean counts, dispersions, and zero (dropout) probabilities. Fit a gene-by-gene negative binomial model with edgeR on FQ (upper-quartile) normalized counts, and fit the MAST hurdle model on log2(TPM+1) transformed values as the third competitor. For each gene, compare estimated versus observed mean counts and zero probabilities across the three models, and render mean-difference (estimated minus observed) plots plus dispersion-vs-zero-frequency plots as PNG diagnostic figures. Judgment: the ZINB fit should track observed mean counts and zero probabilities more closely than edgeR-NB and MAST, and its genewise dispersion estimates should be more stable across the zero-frequency range.

## Related tools

- **zinbwave** (Fit the zero-inflated negative binomial model (zinbFit) and extract estimated mean counts, dispersions, and zero probabilities on raw scRNA-seq counts) — https://bioconductor.org/packages/zinbwave
- **edgeR** (Fit a gene-by-gene negative binomial model on FQ normalized counts to obtain comparable mean and dispersion estimates)
- **MAST** (Fit the hurdle model on log2(TPM+1) transformed values as the third competing count model)
- **scRNAseq** (Provide the single-cell RNA-seq count data set (e.g., V1 data set) used for the model fits)

## Examples

```
BiocManager::install("zinbwave")
```

## Evaluation signals

- Diagnostic plots show ZINB-estimated mean counts tracking observed mean counts more closely than edgeR-NB and MAST estimates
- Diagnostic plots show ZINB-estimated zero probabilities matching observed zero frequencies better than the NB and hurdle alternatives
- Dispersion-vs-zero-frequency plot shows ZINB genewise dispersion estimates are more stable across the range of observed zero frequencies
- Model specification check: sample-level and gene-level intercepts were included so the cell-level intercept acts as a global-scaling normalization factor on raw counts
- Version check: zinbwave run under R (>=3.4) and Bioconductor (>=3.6); install succeeded via BiocManager::install

## Limitations

- Computing time scales with data size; ZINB-WaVE requires numerical optimization of a non-convex likelihood, which is a computational cost
- There is no guarantee that the low-dimensional signal extracted by ZINB-WaVE is biologically relevant, particularly if unwanted technical variation is unaccounted for
- Comparability across competing models is imperfect by construction: NB uses FQ-normalized counts and MAST uses log2(TPM+1) values, so differences also reflect normalization/transform choices

## Evidence

- [methods] zinbwave requires R (>=3.4) and Bioconductor (>=3.6): "Note that `zinbwave` requires R (>=3.4) and Bioconductor (>=3.6)."
- [discussion] cell-level intercept as global-scaling normalization factor: "the corresponding cell-level intercept acts as a global-scaling normalization factor, allowing the modeling of raw count data"
- [intro] zinbwave available through Bioconductor: "The approach is implemented in the open-source R package zinbwave, publicly available through the Bioconductor Project"
- [readme] repo designed to reproduce paper results and figures: [evidence span withheld — anchor status: external_doc]
