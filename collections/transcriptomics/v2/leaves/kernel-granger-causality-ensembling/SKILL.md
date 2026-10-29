---
name: kernel-granger-causality-ensembling
description: Use when you have pseudotime-ordered single-cell gene expression (sparse
  matrix X plus ptime, optional regix/branches in a .mat file) and need to infer a
  ranked directed regulator-target network with Granger causality ensembling.
when_to_use_negative:
- Data are unordered bulk time series or lack pseudotime values — Granger regression
  here requires pseudotemporal ordering of single-cell expression
- 'You need a fully dense (complete) weighted graph: lambda=0 produces a non-sparse
  solution and is avoided because it is impractical and raises segmentation-violation
  risk on large datasets'
- A single, non-ensemble Granger regression is sufficient and the ensemble/aggregation
  overhead is unwarranted
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: SINGE
  role: Implements the GLG (SINGE_GLG_Test) and aggregation (SINGE_Aggregate) stages
    of the Granger ensemble; runs in standalone, parallel, Docker, or compiled-MATLAB
    modes
  repo: https://github.com/gitter-lab/SINGE
- name: glmnet_matlab
  role: Dependency used by GLG for the generalized lasso regression; must be unzipped
    in the root directory or code subdirectory, and is prone to memory segmentation
    violations with large data unless log-transformed gaussian family is used
  repo: http://web.stanford.edu/~hastie/glmnet_matlab/download.html
- name: MATLAB R2018a
  role: Runtime for the compiled Linux executables and compilation of SINGE_GLG_Test/SINGE_Aggregate
    via mcc
  repo: https://www.mathworks.com/products/compiler/matlab-runtime.html
provenance:
  source_task_ids:
  - task_003
  - task_002
  - task_004
  - task_005
  source_papers:
  - doi: 10.1016/j.celrep.2022.110333
    title: Network inference with Granger causality ensembles on single-cell transcriptomics
  source_content_hash: sha256:7ce1a0cef89b199762d639812afc4004ffa5856d90fa955430a6a2d4f13a1aab
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gitter-lab/SINGE
  tools:
  - MATLAB
  - MATLAB R2018a
  - SINGE
  - dynverse
  - dynwrap
  - glmnet_matlab
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.75
n_claims: 12
---

# kernel-granger-causality-ensembling

## Summary

Infers gene regulatory networks from pseudotemporal single-cell expression data by running an ensemble of kernel-based Generalized Lasso Granger (GLG) tests across hyperparameters and subsampled replicates, then compiling the partial networks into a ranked regulator-target list via modified Borda aggregation (SINGE).

## When to use

Use when you have pseudotime-ordered single-cell gene expression (sparse matrix X plus ptime, optional regix/branches in a .mat file) and need to infer a ranked directed regulator-target network with Granger causality ensembling.

## When NOT to use

- Data are unordered bulk time series or lack pseudotime values — Granger regression here requires pseudotemporal ordering of single-cell expression
- You need a fully dense (complete) weighted graph: lambda=0 produces a non-sparse solution and is avoided because it is impractical and raises segmentation-violation risk on large datasets
- A single, non-ensemble Granger regression is sufficient and the ensemble/aggregation overhead is unwarranted

## Inputs

- MAT file with ordered single-cell expression data: sparse matrix X, pseudotime array ptime, optional regulator index array regix, optional branching matrix branches (e.g. data1/X_SCODE_data.mat, data_bifurcated/X_data_bifurcated.mat)
- gene_list file (e.g. data1/gene_list.mat) with gene names corresponding to rows of X
- Hyperparameter file listing GLG combinations (e.g. default_hyperparameters.txt) with --ID, --lambda, --dT, --num-lags, --kernel-width, --replicate, --family, --prob-zero-removal, --prob-remove-samples, --date

## Outputs

- SINGE_Ranked_Edge_List.txt - ranked regulator-target edge list by SINGE scores
- SINGE_Gene_Influence.txt - genes ranked by SINGE influence
- Intermediate per-hyperparameter/per-replicate GLG partial networks in the output directory

## How to apply

Build an input .mat file containing the ordered expression matrix X (log-transform counts as X1 = log(1+X) and use --family gaussian for large count-based datasets to avoid glmnet_matlab segmentation violations), gene_list, and optional regix regulator indices and branches. Run the SINGE GLG stage over the hyperparameter combinations listed in a hyperparameter file (lambda, dT, num-lags, kernel-width, family, prob-zero-removal, prob-remove-samples with default 0.2, replicate), avoiding lambda=0 since it yields a non-sparse complete graph, with kernel-based Granger regression smoothing irregular pseudotimes and missing expression values. Then run SINGE Aggregate to combine the partial GLG networks with the modified Borda aggregation, motivated by stability selection, into SINGE_Ranked_Edge_List.txt and SINGE_Gene_Influence.txt. For branching trajectories, results are combined across all branches unless branch-specific GLG outputs are stored separately for branch-specific aggregation.

## Related tools

- **SINGE** (Implements the GLG (SINGE_GLG_Test) and aggregation (SINGE_Aggregate) stages of the Granger ensemble; runs in standalone, parallel, Docker, or compiled-MATLAB modes) — https://github.com/gitter-lab/SINGE
- **glmnet_matlab** (Dependency used by GLG for the generalized lasso regression; must be unzipped in the root directory or code subdirectory, and is prone to memory segmentation violations with large data unless log-transformed gaussian family is used) — http://web.stanford.edu/~hastie/glmnet_matlab/download.html
- **MATLAB R2018a** (Runtime for the compiled Linux executables and compilation of SINGE_GLG_Test/SINGE_Aggregate via mcc) — https://www.mathworks.com/products/compiler/matlab-runtime.html

## Examples

```
docker run -v $(pwd):/SINGE -w /SINGE agitter/singe:0.5.1 standalone data1/X_SCODE_data.mat data1/gene_list.mat Output default_hyperparameters.txt
```

## Evaluation signals

- Output contains a ranked (non-complete) regulator-target edge list, confirming lambda=0 was not used
- If regix was supplied (e.g. 149 AnimalTFDB 3.0 TFs), all predicted edges are restricted to those candidate regulators
- Re-running with identical inputs and the same --date reproduces the output, since seeds derive from input hyperparameters including date
- Generated output matches tests/reference files via tests/compare_example_output.sh (GitHub Actions compares Docker and macOS runs against reference adjacency matrices, macOS with a permissive threshold)
- Downstream AUROC/AUPRC and early precision against a reference network reproduce reported performance (e.g. modest degradation with 2-5 vs 10 subsampled replicates, no substantial gain beyond five)

## Limitations

- glmnet_matlab encounters memory segmentation violations for large count-based data unless counts are log-transformed (X1 = log(1+X)) and --family gaussian is used
- Subsampling and zero-removal are pseudo-random; reproducibility requires the same inputs including the date hyperparameter
- The master branch may be unstable; use a versioned release for stable data analysis
- Uninformative (shuffled) pseudotime values can degrade GRN inference performance; per-regulator performance can be poor for individual regulators
- No compiled executables exist for Windows; Docker is recommended there

## Evidence

- [abstract] SINGE uses kernel-based Granger causality regression to smooth irregular pseudotimes and missing expression values.: "SINGE uses kernel-based Granger causality regression to smooth irregular pseudotimes and missing expression values."
- [methods] SINGE's ensembling and modified Borda aggregation is motivated by stability selection from these networks.: "SINGE's ensembling and modified Borda aggregation is motivated by stability selection from these networks."
- [methods] we recommend log-transforming the count-based data (`X1 = log(1+X)`) and using the `--family gaussian` hyperparameter instead.: "we recommend log-transforming the count-based data (`X1 = log(1+X)`) and using the `--family gaussian` hyperparameter instead."
- [readme] The full SINGE pipeline runs multiple Generalized Lasso Granger (GLG) tests to infer different directed networks for different hyperparameters and subsamples of the data.: [evidence span withheld — anchor status: external_doc]
- [readme] SINGE generates a random seed using input hyperparameters, including the *date* input.: [evidence span withheld — anchor status: external_doc]
