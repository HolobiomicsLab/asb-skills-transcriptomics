---
name: missing-value-imputation-omics
description: Use when an rROMA (or similar PCA-based gene set activity) analysis is
  run on an expression matrix that contains missing values, and PC1-based module scores
  require a complete global matrix.
when_to_use_negative:
- The input dataset already has missing values imputed beforehand — the README recommends
  using datasets for which missing values have already been imputed rather than re-imputing
- The expression matrix is not normalized — rRoma works with normalized data (e.g.
  cpm from edgeR), and imputation should follow normalization in the pipeline
edam_operation: http://edamontology.org/operation_3557
edam_topics:
- http://edamontology.org/topic_0091
tools:
- name: rROMA
  role: R package whose rRoma.R call invokes the imputation step on the global expression
    matrix before computing module PC1s; exposes FillNAMethod to pass parameters to
    mice
  repo: https://github.com/sysbio-curie/rROMA
- name: mice
  role: Provides the iterative PCA method used to impute missing values in the global
    expression matrix
- name: irlba
  role: Computes only the first principal component (PC1) per module from the completed
    (imputed) matrix
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1038/s41540-024-00331-x
    title: Representation and quantification of module activity from omics data with
      rROMA
  source_content_hash: sha256:7a4a7223d46efc32037139ce5446942bb963b836a7e6a852eea4ea0707ba8865
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/sysbio-curie/rROMA
  tools:
  - R
  - irlba
  - mice
  - rROMA
  edam_operation: http://edamontology.org/operation_3557
  edam_topics:
  - http://edamontology.org/topic_0091
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 8
---

# missing-value-imputation-omics

## Summary

Imputes missing values in omics expression matrices using the iterative PCA (low-rank approximation) method implemented in the R mice package, as the preprocessing step inside the rROMA module-activity workflow before PC1 computation.

## When to use

Use when an rROMA (or similar PCA-based gene set activity) analysis is run on an expression matrix that contains missing values, and PC1-based module scores require a complete global matrix.

## When NOT to use

- The input dataset already has missing values imputed beforehand — the README recommends using datasets for which missing values have already been imputed rather than re-imputing
- The expression matrix is not normalized — rRoma works with normalized data (e.g. cpm from edgeR), and imputation should follow normalization in the pipeline

## Inputs

- Gene expression matrix (rows = gene/protein names, columns = sample IDs) containing missing values; e.g. the 9771 proteins x 122 treatment-naive primary breast cancer mass-spectrometry proteomic dataset
- Optional rRoma.R FillNAMethod parameters for the underlying mice call

## Outputs

- A complete (imputed, lower-rank approximated) global expression matrix subsequently centered and used for per-module PC1 computation
- rRoma module activity scores and ModuleMatrix (shift and L1 p/q values) computed on the imputed matrix

## How to apply

rROMA starts by centering the data of the global expression matrix; if missing values are present, they are imputed using the iterative PCA method implemented in the mice package, which approximates the data matrix containing missing values with a full matrix of lower rank. The imputation is performed on the global matrix (all genes/samples) before any per-module PC1 computation with irlba, so that every module's first principal component is computed on a complete matrix. Parameters can be forwarded to the underlying mice call via the FillNAMethod option of rRoma.R. The README cautions that imputation via mice only needs to be installed/used when the analyzed dataset contains missing values, and recommends preferentially working on datasets whose missing values have already been imputed (e.g. mass-spectrometry proteomics matrices).

## Related tools

- **rROMA** (R package whose rRoma.R call invokes the imputation step on the global expression matrix before computing module PC1s; exposes FillNAMethod to pass parameters to mice) — https://github.com/sysbio-curie/rROMA
- **mice** (Provides the iterative PCA method used to impute missing values in the global expression matrix)
- **irlba** (Computes only the first principal component (PC1) per module from the completed (imputed) matrix)

## Examples

```
install.packages("mice"); rRoma.output <- rRoma.R(MatData, AllHall, FillNAMethod = list(...))
```

## Evaluation signals

- Confirm no NA/missing values remain in the global matrix after the imputation step before module PC1 computation
- Imputation must occur on the global matrix after centering and before per-module irlba PC1 computation, matching the rROMA algorithm order
- FillNAMethod parameters should reach the mice call (verifiable by tracing rRoma.R options)
- Downstream validity: the breast cancer use case should still yield 34 shifted and 6 overdispersed Hallmark pathways at p < 0.05 from rRoma.output$ModuleMatrix

## Limitations

- The README recommends using datasets for which missing values have already been imputed; mice-based imputation is only invoked when the analyzed dataset contains missing values
- The article reports default hyperparameter values (e.g. imputation rank, FillNAMethod defaults) are not stated, so reproduction requires checking rRoma.R documentation
- Imputation by low-rank approximation can be computationally costly on very large matrices (mice must be installed separately if the dataset contains missing values)

## Evidence

- [discussion] rRoma utilizes the iterative PCA method implemented in the mice package40 on the global expression matrix, imputing any missing values within the dataset.: [evidence span withheld — anchor status: not_found]
- [discussion] they can be imputed using an approximation of the data matrix with missing values by a full matrix of lower rank.: "they can be imputed using an approximation of the data matrix with missing values by a full matrix of lower rank."
- [methods] Roma starts by imputing them using mice. More parameters can be passed to the mice function using `FillNAMethod`: "Roma starts by imputing them using mice. More parameters can be passed to the mice function using `FillNAMethod`"
- [readme] rRoma uses the `mice` package to fill missing values. This package needs to be installed if analysed datasets contain missing values.: "rRoma uses the `mice` package to fill missing values. This package needs to be installed if analysed datasets contain missing values. However, we recommend using on datasets for which missing values"
- [discussion] The rROMA algorithm therefore starts by centering the data of the global matrix.: "The rROMA algorithm therefore starts by centering the data of the global matrix."
