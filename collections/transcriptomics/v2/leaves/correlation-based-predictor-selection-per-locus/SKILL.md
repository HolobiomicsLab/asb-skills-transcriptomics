---
name: correlation-based-predictor-selection-per-locus
description: Use when reconstructing BIRD's per-locus prediction model from ENCODE
  RNA-seq training data, i.e., when building a binary model file that predicts DNase
  I hypersensitivity at ~1M genomic loci from genome-wide gene expression.
when_to_use_negative:
- You only want to run prediction with an existing prebuilt model (use BIRD_predict
  with human_hg38_model.bin; no model building needed)
- You want BIRD to discover new DHS locations not represented in the training loci
  — BIRD will not discover new locations of DHSs when applied to a new gene expression
  sample
- Input is exon array or other platform data without a matching trained model — BIRD
  must be extended/trained on the appropriate platform when training data are available
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_3518
tools:
- name: BIRD
  role: Software framework in which the locus-level regression models BIR(X-bar, Y)
    are built and serialized into a binary model file
  repo: https://github.com/WeiqiangZhou/BIRD
- name: BIRD_predict
  role: Loads the locus-level model via the -l flag to validate the model file and
    generate predictions (bounded 0 to 14, log2(x+1) scale)
  repo: https://github.com/WeiqiangZhou/BIRD
- name: R
  role: Required runtime (Rscript executable) for loading BIRD-data training .rds
    files and running BIRD's R-backed modeling code
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-017-01188-x
    title: Genome-wide prediction of DNase I hypersensitivity using gene expression
  source_content_hash: sha256:3a31040b5edc17186703f45c8aed0ba32fd92ccefc24fd2824f3e23ebbcea3a7
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/WeiqiangZhou/BIRD
  tools:
  - BIRD
  - BIRD_predict
  - R
  edam_operation: http://edamontology.org/operation_3659
  edam_topics:
  - http://edamontology.org/topic_3169
  - http://edamontology.org/topic_3518
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# correlation-based-predictor-selection-per-locus

## Summary

BIRD's locus-level model BIR(X-bar, Y) builds a separate regression for each DHS locus: co-expressed genes are compressed into ~1500 k-means cluster-average predictors (X-bar), and per locus only the top N=7 most correlated cluster predictors are retained to fit the regression, reducing dimensionality and co-linearity.

## When to use

Use when reconstructing BIRD's per-locus prediction model from ENCODE RNA-seq training data, i.e., when building a binary model file that predicts DNase I hypersensitivity at ~1M genomic loci from genome-wide gene expression.

## When NOT to use

- You only want to run prediction with an existing prebuilt model (use BIRD_predict with human_hg38_model.bin; no model building needed)
- You want BIRD to discover new DHS locations not represented in the training loci — BIRD will not discover new locations of DHSs when applied to a new gene expression sample
- Input is exon array or other platform data without a matching trained model — BIRD must be extended/trained on the appropriate platform when training data are available

## Inputs

- ENCODE RNA-seq training expression matrix (FPKM, gene ensembl ids in first column) from BIRD-data
- DNase-seq training DH data (200bp-window loci with genomic location columns) from BIRD-data
- Gene cluster definitions from k-means clustering (K=1500)

## Outputs

- Binary locus-level model file (e.g., locus_level_model.bin) containing cluster definitions, selected predictor indices, and per-locus regression coefficients
- Per-locus fitted linear regression models BIR(X-bar, Y)

## How to apply

Load the ENCODE RNA-seq training expression matrix and DH training data from the BIRD-data repository (e.g., BIRD_data_ENCODE.zip with the 167-sample DNase/RNA-seq set) following the build_prediction_model.md recipe. Standardize the training expression matrix and apply BIRD's robust quantile normalization, then run k-means clustering of genes with K=1500 and compute each cluster's average expression (X-bar) as a compressed predictor; clustering reduces predictor dimension, mitigates co-linearity, and makes predictors less sensitive to measurement noise. For each DHS locus (200bp window), compute correlations between the 1500 cluster predictors and the locus DH response and retain the top N=7 most correlated clusters as that locus's predictors. Fit the per-locus linear regressions BIR(X-bar, Y) in the R-backed BIRD modeling code, then serialize cluster definitions, selected predictor indices, and regression coefficients into a binary model file (e.g., locus_level_model.bin) loadable by BIRD_predict with the -l locus-level flag.

## Related tools

- **BIRD** (Software framework in which the locus-level regression models BIR(X-bar, Y) are built and serialized into a binary model file) — https://github.com/WeiqiangZhou/BIRD
- **BIRD_predict** (Loads the locus-level model via the -l flag to validate the model file and generate predictions (bounded 0 to 14, log2(x+1) scale)) — https://github.com/WeiqiangZhou/BIRD
- **R** (Required runtime (Rscript executable) for loading BIRD-data training .rds files and running BIRD's R-backed modeling code)

## Examples

```
path_to_BIRD/BIRD_predict -b locus_level_model.bin -i FPKM_data_matrix.txt -o output_file.txt -l
```

## Evaluation signals

- locus_level_model.bin exists and is loadable by BIRD_predict with the -l locus-level flag
- Predictions produced by the locus-level model fall within the default bounded range of 0 to 14 on the log2(x+1) scale (upper bound adjustable via -u)
- Every DHS locus model uses exactly the top N=7 most correlated gene-cluster predictors out of the 1500 clusters
- The serialized model retains cluster definitions, selected predictor indices, and regression coefficients so it reproduces per-locus regressions from training X-bar values

## Limitations

- BIRD will not discover new locations of DHSs when applied to analyze a new gene expression sample — only loci present in the training DH data get models
- Whether gene expression can similarly predict other functional genomic data types remains an open question
- BIRD should be extended to other gene expression platforms with training data before reuse; locus-level mode is useful when you are not sure if the cluster-level model works

## Evidence

- [methods] locus-level model groups correlated predictors: [evidence span withheld — anchor status: not_found]
- [results] cluster rationale: [evidence span withheld — anchor status: not_found]
- [readme] -l flag semantics: "Users can also use -l option to perform prediction using the locus-level model rather than using the full model."
- [readme] bounded predictions: "The predicted values are now bounded from 0 to 14. Users can use the -u option to change the upper bound when using their own prediction model."
- [discussion] limitation on new DHSs: "one limitation of BIRD is that it will not discover new locations of DHSs when applied to analyze a new gene expression sample"
