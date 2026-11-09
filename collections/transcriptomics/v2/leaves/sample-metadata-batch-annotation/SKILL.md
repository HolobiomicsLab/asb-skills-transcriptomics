---
name: sample-metadata-batch-annotation
description: Use when a merged multi-study microarray matrix shows dominant variance
  driven by study-of-origin (data series/platform) and the known batch must be removed
  while retaining the biological signal (e.g. MDI differentiation time course).
when_to_use_negative:
- The input is a single-study dataset with no series/platform batch structure to remove
  (adjustment would remove signal with no batch to correct).
- The matrix has not yet been through gene collapsing (collapseRows) and limma normalization
  — this skill assumes a normalized gene-level matrix as input.
- The batch covariate is confounded with the biology of interest so that removal would
  eliminate the differentiation time course or perturbation response rather than only
  technical variance.
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: sva
  role: Provides the parametric empirical Bayes batch-removal adjustment (ComBat-style)
    applied to the normalized matrix using the batch covariate
- name: curatedAdipoArray
  role: Source of the curated 3T3-L1 microarray data and homogenized sample metadata
    defining the batch covariate
  repo: https://github.com/MahShaaban/curatedAdipoArray
- name: SummarizedExperiment
  role: Container from which the expression assay and sample metadata columns (data
    series, platform) are extracted
- name: ExperimentHub
  role: Access point for the clean and processed SummarizedExperiment objects
- name: limma
  role: 'Upstream step: log transformation and cross-study normalization that produces
    the input matrix to batch correction'
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1080/21623945.2020.1829852
    title: Curated gene expression dataset of differentiating 3T3-L1 adipocytes under
      pharmacological and genetic perturbations
  source_content_hash: sha256:c5284488b702118f21c2a96b166acd02c276161bb62c601f64246dae073678dd
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
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 8
---

# sample-metadata-batch-annotation

## Summary

Annotates samples with a known batch covariate (data series and platform, i.e. study of origin) from homogenized microarray metadata and removes that batch effect from the normalized gene-level expression matrix using the parametric empirical Bayes adjustment implemented in sva.

## When to use

Use when a merged multi-study microarray matrix shows dominant variance driven by study-of-origin (data series/platform) and the known batch must be removed while retaining the biological signal (e.g. MDI differentiation time course).

## When NOT to use

- The input is a single-study dataset with no series/platform batch structure to remove (adjustment would remove signal with no batch to correct).
- The matrix has not yet been through gene collapsing (collapseRows) and limma normalization — this skill assumes a normalized gene-level matrix as input.
- The batch covariate is confounded with the biology of interest so that removal would eliminate the differentiation time course or perturbation response rather than only technical variance.

## Inputs

- Normalized gene-level expression matrix from the upstream limma normalization step
- curatedAdipoArray SummarizedExperiment object with homogenized sample metadata (data series, platform) accessed via ExperimentHub
- Batch covariate assembled from known study-of-origin factors (data series and platform)

## Outputs

- batch_corrected_matrix.rds — batch-corrected expression matrix with the same dimensions and gene/sample order as the input normalized matrix

## How to apply

Load the normalized gene-level expression matrix produced by the upstream limma normalization step, together with the curated sample metadata from the curatedAdipoArray SummarizedExperiment object accessed via ExperimentHub. Assemble the batch covariate from the known study-of-origin factors — the data series and platform — recorded in the homogenized metadata (43 published studies / 47 datasets were curated, so series and platform identify the batch structure). Apply the parametric empirical Bayes batch-removal method in the sva package (ComBat-style adjustment) to the normalized matrix, using the batch covariate as the adjustment variable, because the goal is to homogenize batch distributions while preserving the differentiation time course and perturbation responses. Save the batch-corrected matrix as batch_corrected_matrix.rds. Validate by checking that dimensions and gene/sample order are unchanged from the input and that published biological observations are reproducible, consistent with the paper's reported result that batch effects were effectively removed.

## Related tools

- **sva** (Provides the parametric empirical Bayes batch-removal adjustment (ComBat-style) applied to the normalized matrix using the batch covariate)
- **curatedAdipoArray** (Source of the curated 3T3-L1 microarray data and homogenized sample metadata defining the batch covariate) — https://github.com/MahShaaban/curatedAdipoArray
- **SummarizedExperiment** (Container from which the expression assay and sample metadata columns (data series, platform) are extracted)
- **ExperimentHub** (Access point for the clean and processed SummarizedExperiment objects)
- **limma** (Upstream step: log transformation and cross-study normalization that produces the input matrix to batch correction)

## Examples

```
batch <- se$series; corrected <- sva::ComBat(assay(se), batch); saveRDS(corrected, "batch_corrected_matrix.rds")
```

## Evaluation signals

- Corrected matrix has the same dimensions and gene/sample order as the input normalized matrix.
- Batch distributions across data series and platforms are homogenized after adjustment.
- Published biological observations (MDI differentiation time course and perturbation responses) remain reproducible after correction.
- The finding that study of origin was the dominant variance source prior to correction is reproducible in the pre-correction matrix.
- Output file batch_corrected_matrix.rds exists and loads as a matrix matching the input gene-level schema.

## Limitations

- Only known batch effects (data series and platform) are removed; unmodeled/unknown batch variation is not addressed by this parametric empirical Bayes adjustment.
- A subset of the data was used to demonstrate effective batch removal, so results depend on which datasets are included in the merged matrix.
- Two datasets were excluded for missing probe annotations and one for missing probe intensities during curation, so the available batch structure is limited to studies passing those filters.

## Evidence

- [intro] the known batch effects (data series and platform) were removed using `sva`.: "the known batch effects (data series and platform) were removed using `sva`."
- [abstract] batch effects removed and biological observations reproduced: "Finally, we used a subset of the data to effectively remove batch effects and reproduce biological observations."
- [intro] missing and low intensity genes were removed; log transform and normalize with limma: "missing and low intensity genes were removed. Then the gene intensities were log transformed and normalized across studies using `limma`."
- [readme] Access to the clean and the processed data SummarizedExperiment objects through ExperimentHub: "Access to the clean and the processed data `SummarizedExperiment` objects through `ExperimentHub`."
- [intro] metadata homogenized across studies: "homogenized across studies using common vocabularies to describe the experimental designs"
