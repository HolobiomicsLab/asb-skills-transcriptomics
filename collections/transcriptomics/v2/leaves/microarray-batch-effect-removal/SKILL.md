---
name: microarray-batch-effect-removal
description: Use when a normalized gene-level expression matrix aggregated across
  multiple GEO studies or platforms shows study-of-origin batch structure, and downstream
  comparison requires homogenized batch distributions while preserving perturbation/differentiation
  signal.
when_to_use_negative:
- When the data come from a single study/series and platform, so no known batch covariate
  exists to adjust
- When only raw probe intensities are available — missing/low-intensity filtering,
  probe-to-gene-symbol collapsing (collapseRows) and limma normalization must be done
  first
- When the goal is to quantify, not remove, batch structure; this skill applies a
  known-covariate adjustment, not de novo latent-factor discovery (e.g. sva's surrogate
  variable estimation)
edam_operation: http://edamontology.org/operation_3656
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0203
tools:
- name: sva
  role: Implements the parametric empirical Bayes (ComBat-style) batch-removal adjustment
    applied to the normalized merged matrix using the study-of-origin batch covariate
- name: curatedAdipoArray
  role: Provides the curated SummarizedExperiment objects whose homogenized metadata
    supply the batch covariate (data series and platform)
  repo: https://github.com/MahShaaban/curatedAdipoArray
- name: limma
  role: Upstream normalization of log-transformed gene intensities across studies,
    producing the input matrix on which batch removal is performed
- name: SummarizedExperiment
  role: Container format in which the curated data and metadata are packaged and loaded
    for batch correction
- name: ExperimentHub
  role: Access route to the clean and processed SummarizedExperiment objects used
    as inputs
provenance:
  source_task_ids:
  - task_001
  - task_004
  source_papers:
  - doi: 10.1080/21623945.2020.1829852
    title: Curated gene expression dataset of differentiating 3T3-L1 adipocytes under
      pharmacological and genetic perturbations
  source_content_hash: sha256:b3daa2627cfe4167346f622d086030fc240e1c942dcbe2a972e2c5ee541d488e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/MahShaaban/curatedAdipoArray
  tools:
  - Docker
  - ExperimentHub
  - GEOquery
  - R/Bioconductor
  - SummarizedExperiment
  - collapseRows
  - curatedAdipoArray
  - limma
  - sva
  edam_operation: http://edamontology.org/operation_3656
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.9166666666666666
n_claims: 12
---

# microarray-batch-effect-removal

## Summary

Removes known batch effects (data series and platform) from a normalized merged microarray expression matrix using the parametric empirical Bayes adjustment implemented in sva, so that multi-study gene-expression data (e.g. the curated 3T3-L1 adipocyte datasets) can be compared while retaining biological signal.

## When to use

Use when a normalized gene-level expression matrix aggregated across multiple GEO studies or platforms shows study-of-origin batch structure, and downstream comparison requires homogenized batch distributions while preserving perturbation/differentiation signal.

## When NOT to use

- When the data come from a single study/series and platform, so no known batch covariate exists to adjust
- When only raw probe intensities are available — missing/low-intensity filtering, probe-to-gene-symbol collapsing (collapseRows) and limma normalization must be done first
- When the goal is to quantify, not remove, batch structure; this skill applies a known-covariate adjustment, not de novo latent-factor discovery (e.g. sva's surrogate variable estimation)

## Inputs

- Normalized gene-level expression matrix (probes collapsed to gene symbols via collapseRows, missing/low-intensity genes removed, log transformed and limma-normalized across studies)
- Homogenized sample metadata (study-of-origin batch factors: data series and platform) from the curatedAdipoArray SummarizedExperiment object accessed via ExperimentHub

## Outputs

- Batch-corrected expression matrix saved as batch_corrected_matrix.rds, with the same dimensions and gene/sample order as the input matrix

## How to apply

Load the normalized gene-level expression matrix produced by upstream limma normalization together with curated sample metadata (e.g. from the curatedAdipoArray SummarizedExperiment accessed via ExperimentHub). Assemble the batch covariate from the known study-of-origin factors recorded in the homogenized metadata — in this paper, data series and platform. Apply the parametric empirical Bayes (ComBat-style) adjustment in sva with that batch covariate as the adjustment variable, so biological signal such as differentiation time courses and perturbation responses is retained. Save the corrected matrix (e.g. batch_corrected_matrix.rds) and verify dimensions and gene/sample order are unchanged. Rationale: study of origin was the dominant source of variance between samples, so removing this known batch is what allowed the paper to reproduce published biological observations from a subset of the data.

## Related tools

- **sva** (Implements the parametric empirical Bayes (ComBat-style) batch-removal adjustment applied to the normalized merged matrix using the study-of-origin batch covariate)
- **curatedAdipoArray** (Provides the curated SummarizedExperiment objects whose homogenized metadata supply the batch covariate (data series and platform)) — https://github.com/MahShaaban/curatedAdipoArray
- **limma** (Upstream normalization of log-transformed gene intensities across studies, producing the input matrix on which batch removal is performed)
- **SummarizedExperiment** (Container format in which the curated data and metadata are packaged and loaded for batch correction)
- **ExperimentHub** (Access route to the clean and processed SummarizedExperiment objects used as inputs)

## Examples

```
Rscript -e 'mat <- readRDS("normalized_matrix.rds"); batch <- colData(se)$series_platform; corrected <- sva::ComBat(dat=mat, batch=batch); saveRDS(corrected, "batch_corrected_matrix.rds")'
```

## Evaluation signals

- Corrected matrix has the same dimensions and gene/sample order as the input normalized matrix
- Batch distributions across data series and platforms are homogenized after adjustment
- Published biological observations (differentiation time course and perturbation responses) remain reproducible in the corrected data
- Batch covariate assembled from metadata covers all samples, matching the known study-of-origin factors (data series and platform)

## Limitations

- Only known batch effects (data series and platform) are removed; unmodeled sources of variation are not addressed by this adjustment
- Biological signal preservation depends on the batch covariate being correctly defined; errors in curated metadata would propagate into the corrected matrix
- Effectiveness was demonstrated on a subset of the data, so full-dataset behavior should still be validated by reproducing known biological observations

## Evidence

- [intro] Remove known batch effects (data series and platform) with sva: "the known batch effects (data series and platform) were removed using `sva`."
- [abstract] Batch effects were effectively removed and biological observations reproduced using a subset of the data: "Finally, we used a subset of the data to effectively remove batch effects and reproduce biological observations."
- [intro] Metadata homogenization supplies the batch covariate: "homogenized across studies using common vocabularies to describe the experimental designs"
- [readme] Input data access via ExperimentHub SummarizedExperiment objects: "Access to the clean and the processed data `SummarizedExperiment` objects through `ExperimentHub`."
