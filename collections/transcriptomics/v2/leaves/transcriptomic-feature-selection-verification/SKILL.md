---
name: transcriptomic-feature-selection-verification
description: Use when reproducing a published nested-CV glmnet model on RNA-Seq data
  and you need to verify which genes the automatically refitted final model retains
  and that their ranked coefficients match the published output.
when_to_use_negative:
- The dataset is not high-dimensional (P >> n) transcriptomic data with limited samples
  — conventional train/test splits may suffice
- You need unbiased accuracy estimation only; nested CV outer-loop performance is
  the target, not refitting a final model to the whole dataset
- Predictors are mixed continuous/categorical factors requiring one_hot() encoding
  beyond a simple gene-expression matrix
edam_operation: http://edamontology.org/operation_3936
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3518
tools:
- name: nestedcv
  role: Implements the fully nested k x l-fold CV with embedded feature-selection
    filters and refits the final model on the whole dataset
  repo: https://CRAN.R-project.org/package=nestedcv
- name: glmnet
  role: Fits the elastic-net regularized logistic model whose lambda/alpha are tuned
    and whose final nonzero coefficients are extracted and ranked
  repo: https://CRAN.R-project.org/package=nestedcv
- name: R
  role: Execution environment for the nestedcv pipeline and CSV export
  repo: https://CRAN.R-project.org/package=nestedcv
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbad048
    title: 'nestedcv: an R package for fast implementation of nested cross-validation
      with embedded feature selection designed for transcriptomics and high-dimensional
      data'
  source_content_hash: sha256:0bbb25934123a8bb7c82bc0f44ff04fa944b89f69a2ad30ea728536e5d845db3
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - glmnet
  - nestedcv
  edam_operation: http://edamontology.org/operation_3936
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.875
n_claims: 8
---

# transcriptomic-feature-selection-verification

## Summary

Reproduce and verify the final fitted glmnet model from nested cross-validation on high-dimensional transcriptomic data (e.g. the R4RA rituximab subset of E-MTAB-11611), extracting and ranking the retained nonzero coefficients against the published vignette output.

## When to use

Use when reproducing a published nested-CV glmnet model on RNA-Seq data and you need to verify which genes the automatically refitted final model retains and that their ranked coefficients match the published output.

## When NOT to use

- The dataset is not high-dimensional (P >> n) transcriptomic data with limited samples — conventional train/test splits may suffice
- You need unbiased accuracy estimation only; nested CV outer-loop performance is the target, not refitting a final model to the whole dataset
- Predictors are mixed continuous/categorical factors requiring one_hot() encoding beyond a simple gene-expression matrix

## Inputs

- Raw RNA-Seq count/expression matrix from ArrayExpress E-MTAB-11611 (R4RA rheumatoid arthritis)
- Response labels for the rituximab-treated subset (binary outcome vector y)
- nestedcv R package (nestcv.glmnet, coef, plot_alphas, plot_lambdas)
- glmnet elastic-net regression backend

## Outputs

- Fitted final nestcv.glmnet model with 21 retained genes plus intercept
- Ranked nonzero-coefficient table written to final_model_coefficients.csv
- Nested CV performance metrics (e.g. outer-loop AUC 0.7648)

## How to apply

Load the raw RNA-Seq matrix (ArrayExpress E-MTAB-11611 for the R4RA example) and attach the outcome labels for the subset of interest (rituximab-treated). Run nestcv.glmnet from the nestedcv R package with 10 x 10-fold nested CV, letting inner folds tune lambda and elastic net alpha, and optionally embed a feature-selection filter via filterFUN (options passed as filter_options) so filtering occurs inside the outer loop to avoid information leakage. After nested CV, perform the final round of CV on the whole dataset so hyperparameters are re-determined and the final model is fitted to the full data. Extract nonzero coefficients from the final fit with coef(), rank them by coefficient value (e.g. 21 genes plus intercept for the rituximab subset, AUC 0.7648, topped by AC016582.3, PCBP3, TMEM170B, EIF4E3, SEC14L6, CEP85, APLF, down to DIRAS3), write the ranked table to final_model_coefficients.csv, and compare gene set and rank order exactly against the published vignette output.

## Related tools

- **nestedcv** (Implements the fully nested k x l-fold CV with embedded feature-selection filters and refits the final model on the whole dataset) — https://CRAN.R-project.org/package=nestedcv
- **glmnet** (Fits the elastic-net regularized logistic model whose lambda/alpha are tuned and whose final nonzero coefficients are extracted and ranked) — https://CRAN.R-project.org/package=nestedcv
- **R** (Execution environment for the nestedcv pipeline and CSV export) — https://CRAN.R-project.org/package=nestedcv

## Examples

```
library(nestedcv); res <- nestcv.glmnet(y, x, family = "binomial", cv.cores = cores); coef(res)
```

## Evaluation signals

- The final model retains exactly 21 genes plus intercept when no filter is applied to the rituximab subset
- Ranked coefficient order matches the published vignette: AC016582.3, PCBP3, TMEM170B, EIF4E3, SEC14L6, CEP85, APLF ... down to DIRAS3
- final_model_coefficients.csv gene set and rank order exactly match the published output
- Outer-loop nested CV AUC reproduces the reported value 0.7648
- plot_alphas(res) and plot_lambdas(res) show deviance tuning across alpha and lambda as in the vignette

## Limitations

- Nested CV fold assignments and results can vary; exact coefficient reproduction depends on matching seed/fold settings used in the vignette
- Filtering on the whole dataset before CV creates bias — filters must be nested within the outer CV loop
- Biomedical datasets with P >> n are too small for a conventional 2/3-1/3 split; performance estimates depend on the full nested CV procedure

## Evidence

- [intro] final model refit on whole dataset: "A final round of CV is performed on the whole dataset to determine
hyperparameters to fit the final model to the whole data"
- [abstract] filters nested to avoid leakage: "the package ensures that filters are nested within the outer CV loop to avoid information leakage from performance test sets"
- [intro] default 10x10 folds stratified: "`nestedcv` partitions the dataset into outer and inner folds (default 10 x 10
folds)."
- [readme] coef() on final model: "Use `summary()` to see full information from the nested model fitting. `coef()`
can be used to show the coefficients of the final fitted model."
- [intro] raw RNA-Seq data location: "# Raw RNA-Seq data for this example is located at:
# https://www.ebi.ac.uk/arrayexpress/experiments/E-MTAB-11611/"
