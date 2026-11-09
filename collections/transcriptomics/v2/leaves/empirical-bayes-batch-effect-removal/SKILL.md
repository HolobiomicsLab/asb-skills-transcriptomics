---
name: empirical-bayes-batch-effect-removal
description: Use when combining multi-study microarray data (e.g. GSE12929 and GSE14004
  from curatedAdipoArray) where study of origin or platform confounds the analysis,
  and you need to verify that a biological variable separates samples after correction.
when_to_use_negative:
- The data come from a single study/series with no known batch variable (data series,
  platform) to adjust for — ComBat-style removal would remove real signal
- The matrix is already batch-corrected, or the goal is differential expression or
  gene set enrichment on the curated data directly without first quantifying batch
  dominance
- Working with the RNA-Seq or ChIP-Seq companion datasets (curatedAdipoRNA/curatedAdipoChIP),
  whose preprocessing pipeline differs from the microarray limma/sva workflow
tools:
- name: sva
  role: Empirical Bayes (ComBat-style) removal of known batch effects (data series
    and platform) from the normalized expression matrix
- name: limma
  role: Log transformation and cross-study normalization of the expression data prior
    to batch adjustment
- name: curatedAdipoArray
  role: Source of the curated microarray SummarizedExperiment data and metadata (stages
    of differentiation, study of origin) that the correction is applied to
  repo: https://github.com/MahShaaban/curatedAdipoArray
- name: ExperimentHub
  role: Provides access to the clean and processed SummarizedExperiment objects used
    as input
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1080/21623945.2020.1829852
    title: Curated gene expression dataset of differentiating 3T3-L1 adipocytes under
      pharmacological and genetic perturbations
  source_content_hash: sha256:d3cb163382ef5cb4f9013c010c022f3080bc9dd846b8da36e13fa4d4a0fa2639
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MahShaaban/curatedAdipoArray
  tools:
  - ExperimentHub
  - R
  - R/Bioconductor
  - curatedAdipoArray
  - limma
  - sva
  edam_operation: null
  edam_topics: []
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7272727272727273
n_claims: 11
---

# Empirical Bayes batch-effect removal

## Summary

Removes known batch effects (data series and platform) from the curated 3T3-L1 adipocyte microarray data using sva's empirical Bayes (ComBat-style) adjustment, so that biological variables such as stage of differentiation — rather than study of origin — dominate downstream analyses like PCA.

## When to use

Use when combining multi-study microarray data (e.g. GSE12929 and GSE14004 from curatedAdipoArray) where study of origin or platform confounds the analysis, and you need to verify that a biological variable separates samples after correction.

## When NOT to use

- The data come from a single study/series with no known batch variable (data series, platform) to adjust for — ComBat-style removal would remove real signal
- The matrix is already batch-corrected, or the goal is differential expression or gene set enrichment on the curated data directly without first quantifying batch dominance
- Working with the RNA-Seq or ChIP-Seq companion datasets (curatedAdipoRNA/curatedAdipoChIP), whose preprocessing pipeline differs from the microarray limma/sva workflow

## Inputs

- curatedAdipoArray SummarizedExperiment object (limma-normalized, probe-to-gene-collapsed log expression matrix) accessed through ExperimentHub
- Sample metadata (colData) identifying study/series of origin, platform, and stage of differentiation
- Batch variable assignments: data series and platform per sample

## Outputs

- Batch-corrected gene expression matrix for the selected subset
- Variance-explained and sample-correlation table per principal component, before vs after adjustment
- Two PCA plots (before and after batch removal)

## How to apply

Load the curatedAdipoArray SummarizedExperiment object via ExperimentHub and subset to the perturbation series of interest (here the Pparg-knockdown studies GSE12929, n=48 and GSE14004, n=9), retaining stage-of-differentiation metadata. Work on the log-transformed expression matrix that was normalized across studies with limma and probe-to-gene collapsed with collapseRows. Apply sva's empirical Bayes batch removal on the subset with the known batch variables set to data series and platform. Recompute PCA on both the raw and batch-corrected matrices, recording per-principal-component variance explained and correlations of samples with study of origin and stage of differentiation; the correction is successful when study of origin no longer dominates the variance and stage of differentiation separates samples on PC2/PC3.

## Related tools

- **sva** (Empirical Bayes (ComBat-style) removal of known batch effects (data series and platform) from the normalized expression matrix)
- **limma** (Log transformation and cross-study normalization of the expression data prior to batch adjustment)
- **curatedAdipoArray** (Source of the curated microarray SummarizedExperiment data and metadata (stages of differentiation, study of origin) that the correction is applied to) — https://github.com/MahShaaban/curatedAdipoArray
- **ExperimentHub** (Provides access to the clean and processed SummarizedExperiment objects used as input)

## Examples

```
if (!requireNamespace("BiocManager", quietly = TRUE)) install.packages("BiocManager"); BiocManager::install("curatedAdipoArray")
```

## Evaluation signals

- Before adjustment, study of origin explains the most variance (dominates PC1) in the PCA of the combined GSE12929 + GSE14004 subset
- After empirical Bayes adjustment, stage of differentiation corresponds to the second and third principal components and separates the samples
- Per-principal-component variance-explained and correlation values are recorded for both the pre- and post-correction matrices
- The corrected matrix retains the biological pattern reported in Figure 3 (batch removal did not just shuffle variance but swapped the dominant variable)
- The two PCA plots (before vs after) and the variance/correlation table are reproducible from the scripts stored in the package's inst/scripts

## Limitations

- Batch removal requires known batch variables (data series and platform); the effect of adjustment should be verified by re-running PCA, since removal may not make the biological variable dominant for all subsets
- The correction was demonstrated only on a subset of the curated data (two Pparg-knockdown series, n=48 and n=9); behavior on other series or the full 47-dataset resource is not established by this analysis
- The upstream data quality depends on the curation filters: datasets with missing probe intensities or probe annotations were excluded (one and two studies respectively), and missing/low-intensity genes were dropped before normalization

## Evidence

- [intro] the known batch effects (data series and platform) were removed using `sva`.: [evidence span withheld — anchor status: not_found]
- [abstract] Finally, we used a subset of the data to effectively remove batch effects and reproduce biological observations.: "Finally, we used a subset of the data to effectively remove batch effects and reproduce biological observations."
- [intro] missing and low intensity genes were removed. Then the gene intensities were log transformed and normalized across studies using `limma`.: "missing and low intensity genes were removed. Then the gene intensities were log transformed and normalized across studies using `limma`."
- [results] Before batch removal, study of origin explained the most variance; after applying empirical Bayes adjustment, stage of differentiation corresponded to the second and third principal components: [evidence span withheld — anchor status: not_found]
- [readme] Access to the clean and the processed data `SummarizedExperiment` objects through `ExperimentHub`.: "Access to the clean and the processed data `SummarizedExperiment` objects
through `ExperimentHub`."
