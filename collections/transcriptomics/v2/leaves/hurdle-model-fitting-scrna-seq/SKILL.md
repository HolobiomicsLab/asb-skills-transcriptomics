---
name: hurdle-model-fitting-scrna-seq
description: 'Use when benchmarking count models for scRNA-seq data with high dropout:
  e.g., when you need to decide whether zero inflation must be modeled explicitly
  or whether NB/hurdle models suffice for mean, zero-probability, and dispersion estimation.'
when_to_use_negative:
- The goal is dimensionality reduction or signal extraction (ZINB-WaVE's primary low-rank
  W-matrix use case) rather than model-fit comparison of mean/zero/dispersion estimation
- Input data are already log-transformed/normalized counts only — the hurdle-model
  comparison requires raw counts for the ZINB fit (zinbwave's intercept normalization
  allows modeling of raw count data)
- Data show negligible zero inflation, so the added complexity of zero-inflated models
  offers no fitting advantage over a plain NB
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: zinbwave
  role: Fits the zero-inflated negative binomial model (zinbFit) with sample/gene-level
    intercepts and genewise dispersion, yielding mean counts, dispersions, and dropout
    probabilities
  repo: https://bioconductor.org/packages/zinbwave
- name: edgeR
  role: Fits the gene-by-gene negative binomial competitor on FQ (upper-quartile)
    normalized counts for mean and dispersion comparison
- name: MAST
  role: Fits the hurdle model on log2(TPM+1) transformed values as the third competing
    count model
- name: R (>=3.4)
  role: Runtime environment required by zinbwave along with Bioconductor (>=3.6)
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-017-02554-5
    title: A general and flexible method for signal extraction from single-cell RNA-seq
      data
  source_content_hash: sha256:b9b2406a6c88aa0220f5001fcbcaf040aef208232c3b220e9cbd09e981fdeb8b
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - Bioconductor (>=3.6)
  - MAST
  - R (>=3.4)
  - edgeR
  - zinbwave
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# hurdle-model-fitting-scrna-seq

## Summary

Fit and compare count models for single-cell RNA-seq data — a MAST hurdle model, a gene-by-gene negative binomial model via edgeR, and a zero-inflated negative binomial model via zinbwave — to evaluate goodness of fit in terms of mean counts, zero (dropout) probabilities, and dispersion estimation.

## When to use

Use when benchmarking count models for scRNA-seq data with high dropout: e.g., when you need to decide whether zero inflation must be modeled explicitly or whether NB/hurdle models suffice for mean, zero-probability, and dispersion estimation.

## When NOT to use

- The goal is dimensionality reduction or signal extraction (ZINB-WaVE's primary low-rank W-matrix use case) rather than model-fit comparison of mean/zero/dispersion estimation
- Input data are already log-transformed/normalized counts only — the hurdle-model comparison requires raw counts for the ZINB fit (zinbwave's intercept normalization allows modeling of raw count data)
- Data show negligible zero inflation, so the added complexity of zero-inflated models offers no fitting advantage over a plain NB

## Inputs

- Single-cell RNA-seq raw count matrix (e.g., the V1 data set) as a SummarizedExperiment/sparse count matrix
- FQ (upper-quartile) normalized counts for the edgeR gene-by-gene NB fit
- log2(TPM+1) transformed expression values for the MAST hurdle model

## Outputs

- Estimated mean counts, genewise dispersions, and zero (dropout) probabilities per gene from the ZINB fit
- Gene-by-gene NB mean and dispersion estimates from edgeR
- Hurdle model estimates from MAST
- PNG diagnostic figures: mean-difference (estimated minus observed) plots and dispersion-vs-zero-frequency plots

## How to apply

Install zinbwave with BiocManager under R (>=3.4) and Bioconductor (>=3.6), and load the scRNA-seq count data set (e.g., the V1 data). Fit the zero-inflated negative binomial model with zinbFit, including sample-level and gene-level intercepts so the cell-level intercept acts as a global-scaling normalization factor, and use genewise dispersion; extract estimated mean counts, dispersions, and zero (dropout) probabilities. Fit the negative binomial competitor gene-by-gene with edgeR on FQ (upper-quartile) normalized counts, and fit the MAST hurdle model on log2(TPM+1)-transformed values. For each gene, compare estimated vs observed mean counts and zero probabilities across the three models and produce mean-difference (estimated minus observed) plots plus dispersion-vs-zero-frequency plots as PNG diagnostics; the ZINB fit is judged better when it tracks observed means and zero probabilities more closely and yields more stable genewise dispersion across the zero-frequency range.

## Related tools

- **zinbwave** (Fits the zero-inflated negative binomial model (zinbFit) with sample/gene-level intercepts and genewise dispersion, yielding mean counts, dispersions, and dropout probabilities) — https://bioconductor.org/packages/zinbwave
- **edgeR** (Fits the gene-by-gene negative binomial competitor on FQ (upper-quartile) normalized counts for mean and dispersion comparison)
- **MAST** (Fits the hurdle model on log2(TPM+1) transformed values as the third competing count model)
- **R (>=3.4)** (Runtime environment required by zinbwave along with Bioconductor (>=3.6))

## Examples

```
BiocManager::install("zinbwave")
```

## Evaluation signals

- Estimated mean counts from the ZINB fit track observed per-gene mean counts more closely than edgeR-NB and MAST in the mean-difference plots
- Estimated zero probabilities from the ZINB fit track observed zero frequencies more closely than the NB and hurdle competitors
- Genewise dispersion estimates from the ZINB model are more stable across the zero-frequency range than edgeR's
- Environment check: R (>=3.4) and Bioconductor (>=3.6) with zinbwave installed via BiocManager
- Diagnostic figures (mean-difference and dispersion-vs-zero-frequency) are produced as PNG files covering all three fitted models

## Limitations

- The ZINB fit comes at a computational cost, as a non-convex likelihood function must be numerically optimized
- The better performance of ZINB-WaVE with respect to PCA-type approaches comes at this computational cost; fit comparisons depend on the normalization choices per model (FQ for edgeR, log2(TPM+1) for MAST)
- zinbwave version matters for reproducibility: reproducing the paper's analyses exactly requires zinbwave version 0.1.1

## Evidence

- [results] The ZINB model fit the data better than NB and hurdle models, achieving better fit of the overall mean count and zero probability and more stable estimators of the dispersion parameter.: [evidence span withheld — anchor status: not_found]
- [methods] zinbwave runtime environment requirement: "Note that `zinbwave` requires R (>=3.4) and Bioconductor (>=3.6)."
- [discussion] Cell-level intercept as normalization factor for raw counts: "the corresponding cell-level intercept acts as a global-scaling normalization factor, allowing the modeling of raw count data, with no need for prior normalization"
- [abstract] Model accounts for dropout and over-dispersion: "the model and its associated estimation procedure are able to give a more stable and accurate low-dimensional representation of the data than principal component analysis (PCA) and zero-inflated"
- [readme] zinb_analysis README reproducibility scope: [evidence span withheld — anchor status: external_doc]
