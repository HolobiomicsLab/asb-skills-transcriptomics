---
name: locus-level-linear-regression-model-fitting
description: Use when building a custom BIRD prediction model from training RNA-seq
  expression and DNase-seq DH data, or when cluster-level model performance is uncertain
  and the locus-level model must be used via BIRD_predict -l.
when_to_use_negative:
- You only need to apply an existing prebuilt model (e.g., human_hg38_model.bin) to
  new expression data — use standard BIRD_predict without retraining
- The full/cluster-level model already performs well; the README notes -l is mainly
  useful when 'you are not sure if the cluster-level model works or not'
- Goal is discovering new DHS locations in a new sample — BIRD will not discover new
  DHS locations beyond its training loci
edam_operation: http://edamontology.org/operation_3659
edam_topics:
- http://edamontology.org/topic_3169
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0622
tools:
- name: BIRD
  role: Source codebase containing the R-backed modeling routines for clustering,
    predictor selection, and per-locus regression fitting
  repo: https://github.com/WeiqiangZhou/BIRD
- name: BIRD_predict
  role: Loads the fitted locus-level model via the -l flag to validate loadability
    and generate predictions
  repo: https://github.com/WeiqiangZhou/BIRD
- name: R
  role: Required runtime (Rscript executable) for loading training data and running
    the model-building scripts
  repo: https://www.r-project.org
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-017-01188-x
    title: Genome-wide prediction of DNase I hypersensitivity using gene expression
  source_content_hash: sha256:43199467fb3d883ac701d9da925f48e5fdca441bda74db84d64f0d0f7df79dc0
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
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0622
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5
n_claims: 6
---

# locus-level-linear-regression-model-fitting

## Summary

Fits BIRD's locus-level model BIR(X-bar, Y), which compresses co-expressed genes into k-means gene-cluster predictors (X-bar), selects the most informative cluster predictors per DHS locus, and fits a per-locus linear regression predicting DNase I hypersensitivity (DH) from genome-wide gene expression.

## When to use

Use when building a custom BIRD prediction model from training RNA-seq expression and DNase-seq DH data, or when cluster-level model performance is uncertain and the locus-level model must be used via BIRD_predict -l.

## When NOT to use

- You only need to apply an existing prebuilt model (e.g., human_hg38_model.bin) to new expression data — use standard BIRD_predict without retraining
- The full/cluster-level model already performs well; the README notes -l is mainly useful when 'you are not sure if the cluster-level model works or not'
- Goal is discovering new DHS locations in a new sample — BIRD will not discover new DHS locations beyond its training loci

## Inputs

- ENCODE RNA-seq training expression matrix (FPKM, Ensembl gene ids) from BIRD-data (e.g., BIRD_data_ENCODE.zip, 167 samples)
- DNase-seq DH training data over 200bp-window genomic loci (e.g., DNase_data_167_cells.rds)
- Number of gene clusters K=1500 and number of selected predictors per locus N=7

## Outputs

- Binary locus-level model file (e.g., locus_level_model.bin) containing cluster definitions, selected predictor indices, and per-locus regression coefficients
- Fitted per-locus linear regression models BIR(X-bar, Y) for ~1M DHS loci, loadable by BIRD_predict -l

## How to apply

Load the ENCODE RNA-seq training expression matrix and DNase-seq DH training data (e.g., 167-sample BIRD_data_ENCODE.zip) into R via readRDS, following the build_prediction_model.md recipe. Standardize the expression matrix and apply BIRD's robust quantile normalization, then run k-means clustering of genes with K=1500 and compute per-cluster average expression (X-bar) to obtain the compressed predictor set — this reduces predictor dimension, mitigates co-linearity, and makes predictors less sensitive to measurement noise. For each DHS locus, compute correlations between the 1500 gene-cluster predictors and the locus DH response and retain the top N=7 most correlated clusters as that locus's predictors. Fit the per-locus linear regressions BIR(X-bar, Y) in the R-backed BIRD modeling code, then serialize cluster definitions, selected predictor indices, and regression coefficients into a binary model file loadable by BIRD_predict with the -l locus-level flag.

## Related tools

- **BIRD** (Source codebase containing the R-backed modeling routines for clustering, predictor selection, and per-locus regression fitting) — https://github.com/WeiqiangZhou/BIRD
- **BIRD_predict** (Loads the fitted locus-level model via the -l flag to validate loadability and generate predictions) — https://github.com/WeiqiangZhou/BIRD
- **R** (Required runtime (Rscript executable) for loading training data and running the model-building scripts) — https://www.r-project.org

## Examples

```
path_to_BIRD/BIRD_predict -b locus_level_model.bin -i FPKM_data_matrix.txt -o output_file.txt -l
```

## Evaluation signals

- locus_level_model.bin exists and is loadable by BIRD_predict with the -l flag
- Predictions produced by the locus-level model fall within the default bounded range 0 to 14 (log2(x+1) scale); upper bound adjustable via -u
- Gene clustering yields exactly K=1500 cluster predictors whose per-cluster averages form the compressed X-bar predictor set
- Each fitted locus model uses exactly N=7 selected gene-cluster predictors with stored indices and regression coefficients
- Input gene ids match model genes: BIRD_predict outputs the matched matrix as input_file.match.txt (Ensembl version suffix ignored unless -e is set)

## Limitations

- BIRD will not discover new locations of DHSs when applied to analyze a new gene expression sample
- Predicted values are bounded (default upper bound 14 on the log2(x+1) scale), which caps extreme predictions unless -u is changed
- The locus-level model is a fallback for when cluster-level model quality is uncertain; the current hg38 model is trained on only 174 ENCODE samples

## Evidence

- [methods] Locus-level model design: [evidence span withheld — anchor status: not_found]
- [results] Rationale for clustering: [evidence span withheld — anchor status: not_found]
- [readme] Locus-level flag in BIRD_predict: "Users can also use -l option to perform prediction using the locus-level model rather than using the full model."
- [readme] Prediction bounds: "The predicted values are now bounded from 0 to 14. Users can use the -u option to change the upper bound"
- [abstract] Whole-transcriptome predictor rationale: "gene expression predicts DH, and information useful for prediction is contained in the whole transcriptome rather than limited to a regulatory element's neighboring genes"
- [readme] Training data availability: [evidence span withheld — anchor status: external_doc]
