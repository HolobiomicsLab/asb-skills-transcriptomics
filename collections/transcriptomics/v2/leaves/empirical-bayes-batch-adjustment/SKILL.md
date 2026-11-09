---
name: empirical-bayes-batch-adjustment
description: Use when a merged, cross-study normalized expression matrix (e.g. GEO-derived
  microarray data collapsed to gene symbols) shows batch structure driven by known
  covariates such as data series or microarray platform that must be removed before
  downstream analyses.
when_to_use_negative:
- Input is raw probe intensities that have not yet been probe-to-gene collapsed (collapseRows)
  and cross-study normalized with limma — batch adjustment assumes an already normalized
  matrix
- The batch structure is unknown or driven by unmeasured confounders rather than known
  covariates like data series and platform — a different sva mode (e.g. latent factor
  estimation) would be needed
- The dataset comes from a single study/series with no platform variation, so no known
  batch effect exists to remove
edam_operation: http://edamontology.org/operation_3695
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0203
tools:
- name: sva
  role: Implements the parametric empirical Bayes batch-removal (ComBat-style) adjustment
    on the normalized matrix using the data-series/platform batch covariate
- name: limma
  role: Upstream tool that log-transforms and normalizes gene intensities across studies,
    producing the input matrix for batch adjustment
- name: curatedAdipoArray
  role: Bioconductor experimental data package supplying the curated 3T3-L1 expression
    matrix and homogenized sample metadata used to build the batch covariate
  repo: https://github.com/MahShaaban/curatedAdipoArray
- name: ExperimentHub
  role: Provides access to the clean and processed SummarizedExperiment data objects
    consumed by this step
- name: SummarizedExperiment
  role: Container format for the processed datasets from which the expression matrix
    and colData metadata are extracted
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1080/21623945.2020.1829852
    title: Curated gene expression dataset of differentiating 3T3-L1 adipocytes under
      pharmacological and genetic perturbations
  source_content_hash: sha256:d59e83d84d763f5fb019eaafe85ef808b16d7133e532c7d52b76e910db28a918
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MahShaaban/curatedAdipoArray
  tools:
  - ExperimentHub
  - R/Bioconductor
  - SummarizedExperiment
  - curatedAdipoArray
  - limma
  - sva
  edam_operation: http://edamontology.org/operation_3695
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# empirical-bayes-batch-adjustment

## Summary

Remove known batch effects (data series and platform) from a normalized multi-study gene expression matrix using the parametric empirical Bayes adjustment implemented in the sva package (ComBat-style), retaining biological signal so differentiation time-course and perturbation responses are reproduced.

## When to use

Use when a merged, cross-study normalized expression matrix (e.g. GEO-derived microarray data collapsed to gene symbols) shows batch structure driven by known covariates such as data series or microarray platform that must be removed before downstream analyses.

## When NOT to use

- Input is raw probe intensities that have not yet been probe-to-gene collapsed (collapseRows) and cross-study normalized with limma — batch adjustment assumes an already normalized matrix
- The batch structure is unknown or driven by unmeasured confounders rather than known covariates like data series and platform — a different sva mode (e.g. latent factor estimation) would be needed
- The dataset comes from a single study/series with no platform variation, so no known batch effect exists to remove

## Inputs

- Normalized gene-level expression matrix (genes x samples) produced by the upstream limma normalization of collapsed probe intensities
- Curated sample metadata from the curatedAdipoArray SummarizedExperiment object (accessed via ExperimentHub), containing homogenized study/series/sample metadata
- Batch covariate vector assembled from known study-of-origin factors (data series and platform)

## Outputs

- batch_corrected_matrix.rds — batch-corrected gene expression matrix with the same dimensions and gene/sample order as the input normalized matrix
- Homogenized batch distributions across data series and platforms, with biological observations preserved

## How to apply

Load the gene-level expression matrix produced by upstream limma normalization (after removing missing and low-intensity genes, log transform, and cross-study normalization) together with curated sample metadata from the curatedAdipoArray SummarizedExperiment object accessed via ExperimentHub. Assemble the batch covariate from the known study-of-origin factors recorded in the homogenized metadata, specifically the data series and platform of each sample. Apply the parametric empirical Bayes batch-removal method from the sva package (ComBat-style adjustment), specifying the batch covariate as the adjustment variable without including biological condition variables so that differentiation time-course and perturbation signal is retained. Save the corrected matrix (batch_corrected_matrix.rds) and verify dimensions and sample/gene order match the input. The paper's rationale is that batch effects from pooling 43 studies (47 datasets) obscure biological observations unless adjusted.

## Related tools

- **sva** (Implements the parametric empirical Bayes batch-removal (ComBat-style) adjustment on the normalized matrix using the data-series/platform batch covariate)
- **limma** (Upstream tool that log-transforms and normalizes gene intensities across studies, producing the input matrix for batch adjustment)
- **curatedAdipoArray** (Bioconductor experimental data package supplying the curated 3T3-L1 expression matrix and homogenized sample metadata used to build the batch covariate) — https://github.com/MahShaaban/curatedAdipoArray
- **ExperimentHub** (Provides access to the clean and processed SummarizedExperiment data objects consumed by this step)
- **SummarizedExperiment** (Container format for the processed datasets from which the expression matrix and colData metadata are extracted)

## Examples

```
batch_corrected_matrix <- sva::ComBat(dat = normalized_matrix, batch = colData(se)$series_platform); saveRDS(batch_corrected_matrix, 'batch_corrected_matrix.rds')
```

## Evaluation signals

- Corrected matrix has the same dimensions and gene/sample order as the input normalized matrix
- Batch distributions across data series and platforms are homogenized after adjustment (e.g. by PCA/clustering on samples grouped by batch)
- Published biological observations — the MDI-induced differentiation time course and perturbation responses — remain reproducible after correction
- Corrected matrix is saved as batch_corrected_matrix.rds and loads back as the expected matrix object in R

## Limitations

- Adjustment is limited to known batch covariates (data series and platform); unmeasured or latent batch effects are not addressed by this known-covariate procedure
- Two of the 47 examined datasets were excluded for missing probe annotations (and one for missing probe intensities), so those studies cannot contribute to the merged matrix being adjusted
- Improperly specified batch covariates risk removing biological signal — the adjustment must retain differentiation time-course and perturbation responses

## Evidence

- [intro] Remove known batch effects (data series and platform) with sva: "the known batch effects (data series and platform) were removed using `sva`."
- [abstract] Batch effects removed and biology reproduced: "Finally, we used a subset of the data to effectively remove batch effects and reproduce biological observations."
- [intro] Upstream normalization step: "missing and low intensity genes were removed. Then the gene intensities were log transformed and normalized across studies using `limma`."
- [intro] Data source and container for inputs: "Access to the clean and the processed data `SummarizedExperiment` objects through `ExperimentHub`."
