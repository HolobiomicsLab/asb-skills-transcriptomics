---
name: hyperparameter-grid-configuration
description: Use when preparing a hyperparameter file for SINGE's GLG tests on pseudotemporal
  single-cell expression data, choosing lambda, dT, num-lags, kernel-width, family,
  and replicate settings for the ensemble.
when_to_use_negative:
- The dataset has more than 1000 genes and the grid would include lambda=0, which
  yields a non-sparse solution and is to be avoided
- A single fixed-network run is desired rather than an ensemble of subsampled GLG
  tests over multiple hyperparameter combinations
- glmnet_matlab segmentation violations occur with --family poisson; switch the grid
  to --family gaussian on log(1+X)-transformed data instead
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: SINGE
  role: Runs the GLG tests for every hyperparameter combination and replicate, then
    aggregates with modified Borda into a ranked network
  repo: https://github.com/gitter-lab/SINGE
- name: glmnet_matlab
  role: Underlying lasso/elastic-net regression package invoked by GLG; family choice
    (gaussian vs poisson) in the hyperparameter file controls its behavior
- name: MATLAB R2018a
  role: Runtime environment in which the compiled SINGE_GLG_Test and SINGE_Aggregate
    executables were built and tested
provenance:
  source_task_ids:
  - task_006
  source_papers:
  - doi: 10.1016/j.celrep.2022.110333
    title: Network inference with Granger causality ensembles on single-cell transcriptomics
  source_content_hash: sha256:ad4d89f647acb2bb05a6cb9da976d2cce0f221aaac180024d9a8f6b992101d9f
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gitter-lab/SINGE
  tools:
  - MATLAB R2018a
  - SINGE
  - glmnet_matlab
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7333333333333333
n_claims: 15
---

# hyperparameter-grid-configuration

## Summary

Construct and apply the grid of Generalized Lasso Granger (GLG) hyperparameter combinations (lambda, dT, num-lags, kernel-width, family, prob-zero-removal, prob-remove-samples) that drives SINGE's ensemble of regression tests and its modified Borda aggregation.

## When to use

Use when preparing a hyperparameter file for SINGE's GLG tests on pseudotemporal single-cell expression data, choosing lambda, dT, num-lags, kernel-width, family, and replicate settings for the ensemble.

## When NOT to use

- The dataset has more than 1000 genes and the grid would include lambda=0, which yields a non-sparse solution and is to be avoided
- A single fixed-network run is desired rather than an ensemble of subsampled GLG tests over multiple hyperparameter combinations
- glmnet_matlab segmentation violations occur with --family poisson; switch the grid to --family gaussian on log(1+X)-transformed data instead

## Inputs

- .mat file with sparse expression matrix X, pseudotime array ptime, optional regulator index array regix, and optional branches matrix
- gene_list.mat listing gene names corresponding to rows of X
- Hyperparameter text file (e.g. default_hyperparameters.txt) with one GLG combination per line
- Value files scripts/lambda.txt, kernel.txt, time.txt, probzeroremoval.txt, probremovesample.txt for grid generation

## Outputs

- Hyperparameter file consumed by SINGE.m / SINGE_GLG_Test.m for all GLG runs
- SINGE_Ranked_Edge_List.txt with edges ranked by SINGE score after aggregation
- SINGE_Gene_Influence.txt with genes ranked by SINGE influence

## How to apply

Enumerate GLG hyperparameter combinations in a text file (e.g. default_hyperparameters.txt), assigning each set a unique --ID plus a --replicate index, since each combination is run over subsampled replicates (ten per combination in the dyngen evaluation, or two to five replicates to reduce runtime with only moderate precision-recall degradation). Set --lambda (sparsity), --dT (time resolution), --num-lags, --kernel-width (Gaussian kernel), --prob-zero-removal, and --prob-remove-samples (default 0.2); avoid lambda=0 for datasets with more than 1000 genes. Use --family gaussian on log(1+X)-transformed counts if glmnet_matlab encounters memory segmentation violations with --family poisson. Generate custom grids with scripts/generate_hyperparameters.sh, which pulls values from scripts/lambda.txt, kernel.txt, time.txt, probzeroremoval.txt, and probremovesample.txt. Include a valid --date because SINGE derives its random seed from the hyperparameters including date, enabling exact reproducibility of subsampling and zero-removal stages.

## Related tools

- **SINGE** (Runs the GLG tests for every hyperparameter combination and replicate, then aggregates with modified Borda into a ranked network) — https://github.com/gitter-lab/SINGE
- **glmnet_matlab** (Underlying lasso/elastic-net regression package invoked by GLG; family choice (gaussian vs poisson) in the hyperparameter file controls its behavior)
- **MATLAB R2018a** (Runtime environment in which the compiled SINGE_GLG_Test and SINGE_Aggregate executables were built and tested)

## Examples

```
docker run -v $(pwd):/SINGE -w /SINGE agitter/singe:0.5.1 standalone data1/X_SCODE_data.mat data1/gene_list.mat Output default_hyperparameters.txt
```

## Evaluation signals

- Every hyperparameter set has a unique --ID across all combinations and replicate indices
- The grid excludes lambda=0 for datasets with more than 1000 genes
- Re-running with the same hyperparameter file and the same --date input reproduces identical subsampling and zero-removal results
- Two-to-five-replicate ensembles show only moderate precision-recall degradation relative to the ten-replicate reference in the dyngen evaluation
- Generated output matches the tests/reference adjacency matrices within the permissive threshold used in the GitHub Actions tests

## Limitations

- Subsampling and zero-removal stages are pseudo-random, so results depend on the seed derived from hyperparameters including the --date input
- lambda=0 produces a non-sparse solution, which is impractical for large gene sets
- The glmnet_matlab package can encounter memory segmentation violations with --family poisson
- SINGE v0.5.0 combines GLG results across all branches for branching trajectories; branch-specific inference requires storing per-branch GLG outputs separately before calling SINGE Aggregate

## Evidence

- [readme] GLG hyperparameters list: [evidence span withheld — anchor status: not_found]
- [readme] generate hyperparameters: [evidence span withheld — anchor status: external_doc]
- [readme] random seed: [evidence span withheld — anchor status: external_doc]
- [methods] gaussian family recommendation: "we recommend log-transforming the count-based data (`X1 = log(1+X)`) and using the `--family gaussian` hyperparameter instead."
- [methods] ensemble rationale: "SINGE's ensembling and modified Borda aggregation is motivated by stability selection from these networks."
