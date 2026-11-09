---
name: microarray-principal-component-analysis
description: Use when working with curated microarray datasets (e.g. 3T3-L1 adipocyte
  series) where you need to check whether batch effects (data series, platform) dominate
  sample variance and whether a biological covariate like stage of differentiation
  separates samples after correction.
when_to_use_negative:
- The data are RNA-Seq or ChIP-Seq counts (curatedAdipoRNA/curatedAdipoChIP) rather
  than processed microarray probe intensities — those packages store RangedSummarizedExperiment
  count/peak objects
- The goal is differential expression or gene set enrichment rather than exploratory
  variance decomposition and batch-effect diagnosis
- The dataset has only one study/series or platform so there is no batch structure
  (data series and platform) for empirical Bayes removal to act on
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
tools:
- name: curatedAdipoArray
  role: Source of the curated, normalized microarray SummarizedExperiment data used
    as PCA input
  repo: https://github.com/MahShaaban/curatedAdipoArray
- name: ExperimentHub
  role: Access to the clean and processed data SummarizedExperiment objects
  repo: https://github.com/MahShaaban/curatedAdipoArray
- name: limma
  role: Log transformation and normalization of gene intensities across studies prior
    to PCA
- name: sva
  role: Empirical Bayes removal of known batch effects (data series and platform)
    before recomputing PCA
- name: GEOquery
  role: Original retrieval of probe intensities, probe annotation and metadata from
    GEO
- name: collapseRows
  role: Collapse probe intensities to gene symbols before PCA
provenance:
  source_task_ids:
  - task_005
  source_papers:
  - doi: 10.1080/21623945.2020.1829852
    title: Curated gene expression dataset of differentiating 3T3-L1 adipocytes under
      pharmacological and genetic perturbations
  source_content_hash: sha256:c374e09f8253daf32a16f4e39bff10f77737acbb3645e69193156f529b09d4e0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MahShaaban/curatedAdipoArray
  tools:
  - ExperimentHub
  - GEOquery
  - R
  - R/Bioconductor
  - collapseRows
  - curatedAdipoArray
  - limma
  - sva
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9090909090909091
n_claims: 11
---

# microarray-principal-component-analysis

## Summary

Compute PCA on log-normalized, probe-to-gene-collapsed microarray expression matrices (from curatedAdipoArray SummarizedExperiment objects) before and after empirical Bayes batch removal, to determine which variable (study of origin vs. stage of differentiation) dominates the principal components.

## When to use

Use when working with curated microarray datasets (e.g. 3T3-L1 adipocyte series) where you need to check whether batch effects (data series, platform) dominate sample variance and whether a biological covariate like stage of differentiation separates samples after correction.

## When NOT to use

- The data are RNA-Seq or ChIP-Seq counts (curatedAdipoRNA/curatedAdipoChIP) rather than processed microarray probe intensities — those packages store RangedSummarizedExperiment count/peak objects
- The goal is differential expression or gene set enrichment rather than exploratory variance decomposition and batch-effect diagnosis
- The dataset has only one study/series or platform so there is no batch structure (data series and platform) for empirical Bayes removal to act on

## Inputs

- curatedAdipoArray SummarizedExperiment object accessed through ExperimentHub
- Subset of MDI-induced 3T3-L1 microarray samples (e.g. GSE12929, n=48; GSE14004, n=9) with stage-of-differentiation metadata
- Log-normalized, limma-normalized, probe-to-gene-collapsed expression matrix

## Outputs

- Per-principal-component variance-explained and sample-correlation table (correlations with study of origin and stage of differentiation)
- Two PCA plots: one on the raw log-normalized matrix and one on the batch-corrected matrix

## How to apply

Load the curatedAdipoArray SummarizedExperiment object via ExperimentHub and subset to the series of interest (e.g. the Pparg-knockdown studies GSE12929, n=48 and GSE14004, n=9), retaining stage-of-differentiation metadata. Compute PCA on the log-transformed, limma-normalized, probe-to-gene-collapsed expression matrix (probe intensities were collapsed to gene symbols using `collapsRows`, after removing missing and low-intensity genes) and record per-principal-component variance explained plus sample correlations with study of origin and stage. Remove the known batch effects (data series and platform) using sva's empirical Bayes approach, then recompute PCA on the corrected matrix. Verify the expected pattern: before correction, study of origin explains the most variance; after correction, stage of differentiation corresponds to PC2 and PC3 and separates the samples. Save the variance-explained/correlation table alongside both PCA plots (pre- and post-correction).

## Related tools

- **curatedAdipoArray** (Source of the curated, normalized microarray SummarizedExperiment data used as PCA input) — https://github.com/MahShaaban/curatedAdipoArray
- **ExperimentHub** (Access to the clean and processed data SummarizedExperiment objects) — https://github.com/MahShaaban/curatedAdipoArray
- **limma** (Log transformation and normalization of gene intensities across studies prior to PCA)
- **sva** (Empirical Bayes removal of known batch effects (data series and platform) before recomputing PCA)
- **GEOquery** (Original retrieval of probe intensities, probe annotation and metadata from GEO)
- **collapseRows** (Collapse probe intensities to gene symbols before PCA)

## Examples

```
library(curatedAdipoArray); se <- curatedAdipoArray::adipo_array3; sub <- se[, se$series %in% c("GSE12929","GSE14004")]; mod <- model.matrix(~0+stage, colData(sub)); corrected <- sva::ComBat(lognorm(sub), batch=sub$series, mod); pca <- prcomp(t(assay(corrected)))
```

## Evaluation signals

- Before batch removal, study of origin explains the most variance in the principal components
- After empirical Bayes correction, stage of differentiation corresponds to the second and third principal components and successfully separates the samples
- The reported biological observation is reproduced on the subset (e.g. GSE12929 + GSE14004) after batch removal
- A variance-explained/correlation table and both pre- and post-correction PCA plots are produced
- The workflow is reproducible using the package scripts in inst/scripts and the bcmslab/adiporeg_array Docker image

## Limitations

- Batch effects (data series and platform) must be removed before biological covariates can be seen; without correction, study of origin masks the differentiation signal
- Two studies were excluded for missing probe intensities and missing probe annotations, so not all GEO series are usable
- The package contains no R functions; the analysis relies on user-side R scripts documented in inst/scripts

## Evidence

- [abstract] Stage of differentiation corresponds to PC2/PC3 after batch removal: "Finally, we used a subset of the data to effectively remove batch effects and reproduce biological observations."
- [intro] sva empirical Bayes removal of data series and platform batch effects: "the known batch effects (data series and platform) were removed using `sva`."
- [intro] limma normalization and low-intensity gene filtering precede PCA: "missing and low intensity genes were removed. Then the gene intensities were log transformed and normalized across studies using `limma`."
- [readme] ExperimentHub access to processed SummarizedExperiment objects: "Access to the clean and the processed data `SummarizedExperiment` objects through `ExperimentHub`."
- [intro] Probes collapsed to gene symbols before analysis: "collapsed using probe to gene symbol annotation using `collapsRows`"
