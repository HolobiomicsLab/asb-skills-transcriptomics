---
name: count-data-log-transformation-for-glmnet
description: Explains how to preprocess large count-based single-cell expression matrices
  (e.g. .mat X with gene_list and regix) before SINGE GLG Granger regression so that
  glmnet_matlab does not hit memory segmentation violations.
when_to_use_negative:
- The expression matrix is already log-transformed or continuous — applying log(1+X)
  again would distort values; the gaussian family is intended for the count-based
  setting described in the article
- The dataset is small enough that glmnet_matlab runs without memory segmentation
  violations under the poisson family
- Data lack pseudotime ordering, which SINGE's kernel-based Granger regression requires
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0199
- http://edamontology.org/topic_0203
tools:
- name: SINGE
  role: Runs the GLG Granger regression on the log-transformed matrix with the --family
    gaussian hyperparameter, then SINGE Aggregate compiles the ranked network
  repo: https://github.com/gitter-lab/SINGE
- name: glmnet_matlab
  role: Underlying regression package whose memory segmentation violations on large
    count data motivate the log-transform-plus-gaussian workaround
  repo: http://web.stanford.edu/~hastie/glmnet_matlab/download.html
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.celrep.2022.110333
    title: Network inference with Granger causality ensembles on single-cell transcriptomics
  source_content_hash: sha256:12f8f6dea1321cb6acdb9bc8fe611909b91cd79c0fb2cdcfd64ce239c2a9c2f0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gitter-lab/SINGE
  tools:
  - MATLAB
  - SINGE
  - glmnet_matlab
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0199
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6428571428571429
n_claims: 14
---

# Count-data log-transformation for glmnet

## Summary

Log-transform large count-based single-cell expression matrices with X1 = log(1+X) and run SINGE's GLG regression with the --family gaussian setting instead of poisson, because glmnet_matlab encounters memory segmentation violations on large count data with the default family.

## When to use

Explains how to preprocess large count-based single-cell expression matrices (e.g. .mat X with gene_list and regix) before SINGE GLG Granger regression so that glmnet_matlab does not hit memory segmentation violations.

## When NOT to use

- The expression matrix is already log-transformed or continuous — applying log(1+X) again would distort values; the gaussian family is intended for the count-based setting described in the article
- The dataset is small enough that glmnet_matlab runs without memory segmentation violations under the poisson family
- Data lack pseudotime ordering, which SINGE's kernel-based Granger regression requires

## Inputs

- Ordered single-cell count-based expression matrix (sparse matrix X) in a .mat file
- Pseudotime values (ptime array) in the same .mat file
- Optional regulator indices (regix array) and branching information (branches matrix)
- Gene list file (e.g. data1/gene_list.mat)
- GLG hyperparameter file (e.g. default_hyperparameters.txt) with --family set to gaussian

## Outputs

- Log-transformed expression matrix X1 = log(1+X) used for GLG regression
- SINGE_Ranked_Edge_List.txt with edges ranked by SINGE scores
- SINGE_Gene_Influence.txt with genes ranked by SINGE influence

## How to apply

Load the ordered single-cell expression data (sparse matrix X, pseudotime ptime, optional regix) from the input .mat file, then apply the log transform X1 = log(1+X) to the count-based matrix. Set the --family hyperparameter to gaussian in the GLG hyperparameter file (generated via scripts/generate_hyperparameters.sh) rather than poisson, since the article reports glmnet_matlab memory segmentation violations for large count-based datasets under that setting. Then run SINGE in GLG mode over the hyperparameter combinations (e.g. default_hyperparameters.txt) with the lambda sparsity parameter kept above 0, followed by SINGE Aggregate to produce the ranked edge list. Reproducibility depends on supplying the same hyperparameters, including the --date input, so reruns with identical inputs should regenerate identical subsampling results.

## Related tools

- **SINGE** (Runs the GLG Granger regression on the log-transformed matrix with the --family gaussian hyperparameter, then SINGE Aggregate compiles the ranked network) — https://github.com/gitter-lab/SINGE
- **glmnet_matlab** (Underlying regression package whose memory segmentation violations on large count data motivate the log-transform-plus-gaussian workaround) — http://web.stanford.edu/~hastie/glmnet_matlab/download.html

## Examples

```
bash SINGE.sh PATH_TO_RUNTIME standalone data1/X_SCODE_data.mat data1/gene_list.mat Output default_hyperparameters.txt
```

## Evaluation signals

- The transformed matrix X1 contains only non-negative values and equals log(1+X) of the original count matrix
- glmnet_matlab completes without memory segmentation violations on the transformed data
- Output SINGE_Ranked_Edge_List.txt is a ranked (non-complete) edge list, confirming lambda=0 was avoided
- Re-running with identical hyperparameters including the --date input reproduces the same subsampled GLG results
- The downstream SINGE Aggregate output matches reference outputs under the tests/compare_example_output.sh comparison

## Limitations

- The lambda = 0 setting yields a non-sparse complete-graph solution and must be avoided; only lambda > 0 gives a ranked candidate interaction list
- glmnet_matlab memory segmentation violations are specific to large count-based datasets run without the log-transform + gaussian workaround
- Compiled SINGE executables are not available for Windows; Docker is recommended if no compatible MATLAB environment exists

## Evidence

- [methods] log-transform recommendation: "we recommend log-transforming the count-based data (`X1 = log(1+X)`) and using the `--family gaussian` hyperparameter instead."
- [methods] glmnet_matlab memory issue: "the glmnet_matlab package encounters memory segmentation violations with this setting"
- [readme] family hyperparameter definition: [evidence span withheld — anchor status: not_found]
