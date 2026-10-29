---
name: zinb-model-fitting-count-data
description: Use when raw scRNA-seq count data show zero inflation (dropouts) and
  over-dispersion, and you need model-based estimates of mean counts, zero probabilities,
  and dispersion, or a goodness-of-fit comparison of ZINB vs NB/hurdle count models.
when_to_use_negative:
- The goal is supervised differential expression — this capability is not yet available
  in zinbwave and is only envisioned for a future version
- You need a fast exploratory low-dimensional embedding and cannot afford the computational
  cost of optimizing a non-convex likelihood function
- Data are bulk RNA-seq or otherwise not zero-inflated single-cell counts, where a
  plain negative binomial model (e.g., edgeR) suffices
edam_operation: http://edamontology.org/operation_3664
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0203
tools:
- name: zinbwave
  role: Implements the zero-inflated negative binomial model; zinbFit fits the ZINB
    model to raw counts with gene-level and sample-level intercepts
  repo: https://bioconductor.org/packages/zinbwave
- name: edgeR
  role: Fits the gene-by-gene negative binomial model on FQ normalized counts as the
    non-zero-inflated comparator
- name: MAST
  role: Fits the hurdle model on log2(TPM+1) transformed values as the third competing
    count model
- name: scRNAseq
  role: Provides the single-cell count data sets (e.g., the V1 data set) used as model
    input
- name: Bioconductor (>=3.6)
  role: Required platform and version constraint for installing zinbwave
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-017-02554-5
    title: A general and flexible method for signal extraction from single-cell RNA-seq
      data
  source_content_hash: sha256:2d134fc6b60a844f3dcfdbd2978207e90c270881f7076d689bd1f3a069f9bdff
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
  edam_operation: http://edamontology.org/operation_3664
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# zinb-model-fitting-count-data

## Summary

Fit a zero-inflated negative binomial (ZINB-WaVE) model to raw single-cell RNA-seq counts via zinbFit, extracting estimated mean counts, genewise dispersions, and dropout (zero) probabilities, and benchmark goodness-of-fit against a gene-by-gene edgeR negative binomial model and the MAST hurdle model.

## When to use

Use when raw scRNA-seq count data show zero inflation (dropouts) and over-dispersion, and you need model-based estimates of mean counts, zero probabilities, and dispersion, or a goodness-of-fit comparison of ZINB vs NB/hurdle count models.

## When NOT to use

- The goal is supervised differential expression — this capability is not yet available in zinbwave and is only envisioned for a future version
- You need a fast exploratory low-dimensional embedding and cannot afford the computational cost of optimizing a non-convex likelihood function
- Data are bulk RNA-seq or otherwise not zero-inflated single-cell counts, where a plain negative binomial model (e.g., edgeR) suffices

## Inputs

- Raw single-cell RNA-seq gene-by-cell count matrix (e.g., V1 data set from scRNAseq) as a SummarizedExperiment/SingleCellExperiment
- FQ (upper-quartile) normalized counts for the edgeR negative binomial comparison
- log2(TPM+1) transformed values for the MAST hurdle model comparison

## Outputs

- ZINB model fits: per-gene/cell estimated mean counts, genewise dispersion estimates, and zero (dropout) probabilities from zinbFit
- edgeR negative binomial mean and dispersion estimates per gene
- PNG diagnostic figures: mean-difference plots (estimated minus observed mean counts and zero probabilities) and dispersion-vs-zero-frequency plots

## How to apply

Install zinbwave with BiocManager under R (>=3.4) and Bioconductor (>=3.6), then load the single-cell count data (e.g., the V1 data set via scRNAseq). Call zinbFit on the raw counts including sample-level and gene-level intercepts — a column of ones in the gene-level covariate matrix makes the cell-level intercept a global-scaling normalization factor, so no prior normalization is needed — and request genewise dispersion. Extract estimated mean counts, dispersions, and zero (dropout) probabilities from the fit. Fit competing models on the same data: a gene-by-gene negative binomial with edgeR on FQ (upper-quartile) normalized counts, and the MAST hurdle model on log2(TPM+1) transformed values. For each gene, plot estimated vs observed mean counts and zero probabilities and produce mean-difference (estimated minus observed) plots plus dispersion-vs-zero-frequency plots as PNG diagnostics to judge relative fit.

## Related tools

- **zinbwave** (Implements the zero-inflated negative binomial model; zinbFit fits the ZINB model to raw counts with gene-level and sample-level intercepts) — https://bioconductor.org/packages/zinbwave
- **edgeR** (Fits the gene-by-gene negative binomial model on FQ normalized counts as the non-zero-inflated comparator)
- **MAST** (Fits the hurdle model on log2(TPM+1) transformed values as the third competing count model)
- **scRNAseq** (Provides the single-cell count data sets (e.g., the V1 data set) used as model input)
- **Bioconductor (>=3.6)** (Required platform and version constraint for installing zinbwave)

## Examples

```
BiocManager::install("zinbwave")
```

## Evaluation signals

- ZINB estimated mean counts track observed mean counts more closely than edgeR-NB and MAST in the per-gene estimated-vs-observed and mean-difference diagnostic plots
- ZINB estimated zero probabilities track observed zero frequencies more closely than the competing models in the diagnostic plots
- ZINB genewise dispersion estimates are more stable across the range of zero frequencies than the NB/hurdle alternatives (dispersion-vs-zero-frequency plot)
- The sample-level intercept acts as a global-scaling normalization factor, confirming raw counts were modeled without prior normalization
- Session satisfies the version constraints R (>=3.4) and Bioconductor (>=3.6), and to reproduce the paper exactly, zinbwave version 0.1.1

## Limitations

- Fitting requires numerical optimization of a non-convex likelihood function, incurring computational cost
- Supervised differential expression is not yet available in zinbwave; only envisioned for a future version
- Supervised/differential capability aside, if unwanted technical variation is not accounted for, the inferred low-dimensional signal may capture confounding effects rather than biology
- Reproducing the paper's analyses exactly requires zinbwave version 0.1.1 (installable from drisso/archive-zinbwave@v0.1.1)

## Evidence

- [methods] Version constraints for zinbwave installation: "Note that `zinbwave` requires R (>=3.4) and Bioconductor (>=3.6)."
- [discussion] Cell-level intercept as normalization factor for raw counts: "the corresponding cell-level intercept acts as a global-scaling normalization factor, allowing the modeling of raw count data, with no need for prior normalization"
- [results] Goodness-of-fit comparison tools from the article index: [evidence span withheld — anchor status: not_found]
- [readme] README: exact zinbwave version needed for reproducibility: [evidence span withheld — anchor status: external_doc]
