---
name: gene-signature-reproducibility-checking
description: Use when a published nested-CV transcriptomic model reports a final fitted
  gene signature (ranked nonzero coefficients) and you need to verify your refit retains
  the same genes in the same order.
when_to_use_negative:
- You only need unbiased performance estimation — nested CV on held-out folds suffices;
  do not conflate outer-fold performance with the final refit coefficients used for
  signature checking.
- The predictors are low-dimensional with n >> p — nested CV with embedded feature
  selection is designed for P >> n transcriptomic data.
- You plan to filter features on the whole dataset before CV — the article shows this
  biases accuracy estimates; filters must run inside the outer loop.
edam_operation: http://edamontology.org/operation_2939
edam_topics:
- http://edamontology.org/topic_3518
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2269
tools:
- name: nestedcv
  role: Runs nestcv.glmnet for nested k x l-fold CV, final whole-data refit, and coef()
    extraction of the final model signature
  repo: https://CRAN.R-project.org/package=nestedcv
- name: glmnet
  role: Fits the elastic-net regularized logistic model whose nonzero coefficients
    define the retained gene signature
- name: caret
  role: Provides createFolds() for stratified outer/inner fold assignment on the outcome
    variable
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1093/bioadv/vbad048
    title: 'nestedcv: an R package for fast implementation of nested cross-validation
      with embedded feature selection designed for transcriptomics and high-dimensional
      data'
  source_content_hash: sha256:7047ac42c40a3b70ed5bd511b09ec0080a9c2bbd744bd310ebbc908782210b6e
schema_version: 0.2.0
metadata:
  license_tier: open
  tools:
  - R
  - caret
  - glmnet
  - nestedcv
  edam_operation: http://edamontology.org/operation_2939
  edam_topics:
  - http://edamontology.org/topic_3518
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2269
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5555555555555556
n_claims: 9
---

# gene-signature-reproducibility-checking

## Summary

Reproduce a published glmnet gene signature (e.g. the 22-feature rituximab response model from the R4RA E-MTAB-11611 RNA-Seq cohort) by refitting the final nestcv.glmnet model on the whole dataset and diffing the ranked nonzero coefficients against the published output.

## When to use

Use when a published nested-CV transcriptomic model reports a final fitted gene signature (ranked nonzero coefficients) and you need to verify your refit retains the same genes in the same order.

## When NOT to use

- You only need unbiased performance estimation — nested CV on held-out folds suffices; do not conflate outer-fold performance with the final refit coefficients used for signature checking.
- The predictors are low-dimensional with n >> p — nested CV with embedded feature selection is designed for P >> n transcriptomic data.
- You plan to filter features on the whole dataset before CV — the article shows this biases accuracy estimates; filters must run inside the outer loop.

## Inputs

- Raw RNA-Seq expression matrix from ArrayExpress E-MTAB-11611 (R4RA rheumatoid arthritis cohort)
- Response labels for the rituximab-treated subset (binary outcome vector y)
- Published reference signature (ranked gene list / coefficients, e.g. AC016582.3 ... DIRAS3)

## Outputs

- final_model_coefficients.csv: ranked nonzero coefficient table from the final whole-data glmnet fit (21 genes plus intercept for the rituximab vignette)
- Outer-CV performance metrics (e.g. AUC) from the nestcv.glmnet result object

## How to apply

Load the raw RNA-Seq matrix from ArrayExpress experiment E-MTAB-11611 and attach response labels for the rituximab-treated R4RA subset. Run nested cross-validation with nestcv.glmnet (10 x 10-fold by default, stratified on the outcome via caret's createFolds), letting inner folds tune lambda and elastic-net alpha with any embedded filterFUN nested inside the outer loop to avoid information leakage. After nested CV, perform the final round of cross-validation on the whole dataset — this is how nestedcv determines hyperparameters for the final fit — then extract nonzero coefficients from the final glmnet fit with coef(). Rank the retained genes by coefficient value and compare the gene set and rank order against the published signature (for the rituximab cohort, 21 genes plus intercept, topped by AC016582.3, PCBP3, TMEM170B, EIF4E3, SEC14L6, CEP85, APLF, down to DIRAS3; outer AUC 0.7648). Write the ranked table to final_model_coefficients.csv.

## Related tools

- **nestedcv** (Runs nestcv.glmnet for nested k x l-fold CV, final whole-data refit, and coef() extraction of the final model signature) — https://CRAN.R-project.org/package=nestedcv
- **glmnet** (Fits the elastic-net regularized logistic model whose nonzero coefficients define the retained gene signature)
- **caret** (Provides createFolds() for stratified outer/inner fold assignment on the outcome variable)

## Examples

```
library(nestedcv); res <- nestcv.glmnet(y, x, family = "binomial", filterFUN = NULL, cv.cores = cores); coef(res); write.csv(data.frame(gene = names(cf), coef = cf), "final_model_coefficients.csv")
```

## Evaluation signals

- Nonzero-coefficient gene set in final_model_coefficients.csv exactly matches the published vignette list (AC016582.3, PCBP3, TMEM170B, EIF4E3, SEC14L6, CEP85, APLF, ..., DIRAS3).
- Ranked coefficient order matches the published ranking, and the model contains 21 genes plus intercept for the rituximab subset.
- Outer-CV AUC is consistent with the reported 0.7648 for the no-filter rituximab model.
- summary(res) reports completed outer folds with no test-set leakage (filters executed within outer folds, not on the full dataset).
- coef() on the final fit returns an intercept plus coefficients only for the expected retained genes (all others exactly zero).

## Limitations

- Feature selection or filtering performed on the whole dataset before CV biases accuracy estimates, so signature reproduction must re-run the full nested procedure rather than refit on pre-filtered data.
- Exact gene-set reproduction depends on identical fold seeds, filter settings, and package versions; nested CV results can vary without fixed fold indices.
- The final whole-data model is fitted for prediction on external data — its coefficients are not themselves an unbiased performance measure.

## Evidence

- [intro] final round of CV on the whole dataset: "A final round of CV is performed on the whole dataset to determine hyperparameters to fit the final model to the whole data"
- [abstract] filters nested in outer loop: "the package ensures that filters are nested within the outer CV loop to avoid information leakage from performance test sets"
- [readme] coef on final model: "Use `summary()` to see full information from the nested model fitting. `coef()` can be used to show the coefficients of the final fitted model."
- [abstract] P >> n motivation: "Nested CV with embedded feature selection is especially suited to biomedical data where the sample size is frequently limited, but the number of predictors may be significantly larger (P ≫ n)"
- [intro] R4RA dataset location: "# Raw RNA-Seq data for this example is located at:
# https://www.ebi.ac.uk/arrayexpress/experiments/E-MTAB-11611/"
