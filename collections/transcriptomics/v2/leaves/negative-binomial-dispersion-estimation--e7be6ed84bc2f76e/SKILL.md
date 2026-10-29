---
name: negative-binomial-dispersion-estimation
description: Use when modeling scRNA-seq count data requires comparing goodness-of-fit
  of zero-inflated NB vs negative binomial (edgeR) vs hurdle (MAST) models in terms
  of mean count, zero (dropout) probability, and dispersion estimation.
when_to_use_negative:
- Input data are already normalized low-dimensional embeddings or transformed expressions
  where count-based dispersion estimation is not meaningful (ZINB-WaVE's cell-level
  intercept global-scaling normalization is intended for raw count data)
- The analysis goal is supervised differential expression — this capability was not
  yet available and only envisioned for a future zinbwave version
- Bulk RNA-seq data without zero inflation/dropouts, where a plain negative binomial
  model (e.g., edgeR/DESeq2) alone is sufficient
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: zinbwave
  role: Implements the zero-inflated negative binomial model; zinbFit with sample-level
    and gene-level intercepts and genewise dispersion yields mean counts, dispersions,
    and zero (dropout) probabilities
  repo: https://bioconductor.org/packages/zinbwave
- name: edgeR
  role: Fits the gene-by-gene negative binomial model on FQ (upper-quartile) normalized
    counts as the non-zero-inflated competing model
- name: MAST
  role: Fits the hurdle model on log2(TPM+1) transformed values as the third competing
    count model
- name: scRNAseq
  role: Provides the V1 single-cell RNA-seq count dataset used for the goodness-of-fit
    comparison
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1038/s41467-017-02554-5
    title: A general and flexible method for signal extraction from single-cell RNA-seq
      data
  source_content_hash: sha256:fa3bcd98ddb9c816d197fe3a54a79ee883c028299de75370d891ed11f05e8ce7
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5454545454545454
n_claims: 11
---

# negative-binomial-dispersion-estimation

## Summary

Fit and compare count-model dispersion and zero-inflation estimates for single-cell RNA-seq data using a zero-inflated negative binomial model (ZINB-WaVE zinbFit), a gene-by-gene negative binomial model (edgeR), and a MAST hurdle model, benchmarking estimated mean counts, zero probabilities, and genewise dispersions against observed values.

## When to use

Use when modeling scRNA-seq count data requires comparing goodness-of-fit of zero-inflated NB vs negative binomial (edgeR) vs hurdle (MAST) models in terms of mean count, zero (dropout) probability, and dispersion estimation.

## When NOT to use

- Input data are already normalized low-dimensional embeddings or transformed expressions where count-based dispersion estimation is not meaningful (ZINB-WaVE's cell-level intercept global-scaling normalization is intended for raw count data)
- The analysis goal is supervised differential expression — this capability was not yet available and only envisioned for a future zinbwave version
- Bulk RNA-seq data without zero inflation/dropouts, where a plain negative binomial model (e.g., edgeR/DESeq2) alone is sufficient

## Inputs

- V1 single-cell RNA-seq raw count dataset (e.g., via the scRNAseq Bioconductor package)
- FQ (upper-quartile) normalized counts for the edgeR negative binomial fit
- log2(TPM+1) transformed values for the MAST hurdle model fit

## Outputs

- ZINB-estimated gene-level mean counts, dispersions, and zero (dropout) probabilities from zinbFit
- edgeR gene-by-gene negative binomial mean and dispersion estimates
- MAST hurdle model fits on log2(TPM+1) values
- PNG diagnostic figures: mean-difference (estimated minus observed) plots and dispersion-vs-zero-frequency plots

## How to apply

Install the zinbwave package with BiocManager under R (>=3.4) and Bioconductor (>=3.6), load the V1 single-cell RNA-seq count dataset, and fit the zero-inflated negative binomial model with zinbFit including sample-level and gene-level intercepts — including a column of ones in the gene-level covariate matrix makes the cell-level intercept a global-scaling normalization factor so raw counts can be modeled without prior normalization — extracting estimated mean counts, dispersions, and zero (dropout) probabilities. Fit a gene-by-gene negative binomial model with edgeR on FQ (upper-quartile) normalized counts to obtain comparable mean and dispersion estimates. Fit the MAST hurdle model on log2(TPM+1) transformed values as the third competing count model. For each gene, compare estimated vs observed mean counts and zero probabilities across the three models and generate mean-difference (estimated minus observed) plots plus dispersion-vs-zero-frequency plots as PNG diagnostic figures. The ZINB model is judged to fit better when it tracks observed mean counts and zero probabilities more closely and yields more stable genewise dispersion estimates across the zero-frequency range.

## Related tools

- **zinbwave** (Implements the zero-inflated negative binomial model; zinbFit with sample-level and gene-level intercepts and genewise dispersion yields mean counts, dispersions, and zero (dropout) probabilities) — https://bioconductor.org/packages/zinbwave
- **edgeR** (Fits the gene-by-gene negative binomial model on FQ (upper-quartile) normalized counts as the non-zero-inflated competing model)
- **MAST** (Fits the hurdle model on log2(TPM+1) transformed values as the third competing count model)
- **scRNAseq** (Provides the V1 single-cell RNA-seq count dataset used for the goodness-of-fit comparison)

## Examples

```
BiocManager::install("zinbwave")
```

## Evaluation signals

- Diagnostic mean-difference plots: ZINB-estimated mean counts track observed mean counts more closely than edgeR-NB and MAST
- Diagnostic zero-probability plots: ZINB-estimated zero probabilities match observed dropout frequencies better than the competing models
- Dispersion-vs-zero-frequency plots show ZINB genewise dispersion estimates are more stable across the zero-frequency range
- zinbwave environment meets version requirements: R (>=3.4) and Bioconductor (>=3.6); exact paper reproduction used zinbwave version 0.1.1

## Limitations

- There is no guarantee that the low-dimensional signal extracted by ZINB-WaVE is biologically relevant
- If unwanted technical variation affects the data and is not accounted for in the model or prior normalization, the inferred low-rank matrix W will capture such confounding effects
- Better performance comes at a computational cost from numerically optimizing a non-convex likelihood function
- Supervised differential expression capability is not yet available in zinbwave

## Evidence

- [methods] Note that `zinbwave` requires R (>=3.4) and Bioconductor (>=3.6).: "Note that `zinbwave` requires R (>=3.4) and Bioconductor (>=3.6)."
- [intro] includes a sample-level intercept, which serves as a global-scaling normalization factor: "includes a sample-level intercept, which serves as a global-scaling normalization factor"
- [discussion] the corresponding cell-level intercept acts as a global-scaling normalization factor, allowing the modeling of raw count data: "the corresponding cell-level intercept acts as a global-scaling normalization factor, allowing the modeling of raw count data, with no need for prior normalization"
- [readme] To exactly reproduce the analyses of the paper, version `0.1.1` of the `zinbwave` package is required.: [evidence span withheld — anchor status: external_doc]
