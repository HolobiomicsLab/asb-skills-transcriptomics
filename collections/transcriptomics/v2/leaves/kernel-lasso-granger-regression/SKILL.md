---
name: kernel-lasso-granger-regression
description: Use when input is pseudotemporally ordered single-cell expression data
  (e.g., a .mat file with sparse matrix X and ptime) and the goal is to infer directed
  regulator-target interactions via kernel-smoothed Lasso Granger regression.
when_to_use_negative:
- Data lack pseudotemporal ordering or the pseudotimes are uninformative — the article
  warns SINGE performs poorly with uninformative pseudotimes
- You need branch-specific networks but did not separate per-branch GLG outputs before
  calling Aggregate on a branching trajectory
- lambda=0 is selected — the GLG test becomes non-sparse and no network edge ranking
  is produced
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: SINGE
  role: Implements the GLG tests (SINGE_GLG_Test.m) and standalone pipeline (SINGE.m)
    for kernel-weighted Lasso Granger regression on pseudotemporal single-cell data
  repo: https://github.com/gitter-lab/SINGE
- name: glmnet_matlab
  role: Dependency used by SINGE GLG to fit the Lasso (Generalized Lasso Granger)
    regression models; must be unzipped in the SINGE root or code subdirectory
  repo: http://web.stanford.edu/~hastie/glmnet_matlab/download.html
- name: MATLAB R2018a
  role: Runtime environment for the standalone MATLAB and compiled Linux GLG executables
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.celrep.2022.110333
    title: Network inference with Granger causality ensembles on single-cell transcriptomics
  source_content_hash: sha256:fc97ef3a99ac71841e691c2d1800e89fff52b54fb1f4397e7034d9278e7817d0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gitter-lab/SINGE
  tools:
  - MATLAB R2018a
  - SINGE
  - glmnet_matlab
  edam_operation: http://edamontology.org/operation_0279
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 12
---

# kernel-lasso-granger-regression

## Summary

SINGE's Generalized Lasso Granger (GLG) regression performs kernel-weighted, sparsity-controlled Granger causality testing on pseudotemporally ordered single-cell expression data, using Gaussian kernel smoothing over irregular pseudotimes and glmnet_matlab Lasso regression with per-gene subsampling across replicates.

## When to use

Use when input is pseudotemporally ordered single-cell expression data (e.g., a .mat file with sparse matrix X and ptime) and the goal is to infer directed regulator-target interactions via kernel-smoothed Lasso Granger regression.

## When NOT to use

- Data lack pseudotemporal ordering or the pseudotimes are uninformative — the article warns SINGE performs poorly with uninformative pseudotimes
- You need branch-specific networks but did not separate per-branch GLG outputs before calling Aggregate on a branching trajectory
- lambda=0 is selected — the GLG test becomes non-sparse and no network edge ranking is produced

## Inputs

- .mat file with sparse expression matrix X, pseudotime array ptime, optional regulator indices regix, optional branches matrix
- gene_list.mat file of gene names corresponding to rows of X
- hyperparameter file (e.g., default_hyperparameters.txt) with lambda, dT, num-lags, kernel-width, replicate, family, prob-zero-removal, prob-remove-samples, date

## Outputs

- Per-hyperparameter/replicate GLG partial adjacency outputs written to outdir
- SINGE_Ranked_Edge_List.txt with ranked regulator-target edges by SINGE score
- SINGE_Gene_Influence.txt with genes ranked by SINGE influence

## How to apply

Load the expression matrix X, gene_list, and pseudotimes from the input .mat file (e.g., data_bifurcated/X_data_bifurcated.mat or data1/X_SCODE_data.mat) and rescale pseudotimes to the 0-100 range expected by the SINGE kernel. For each hyperparameter combination from default_hyperparameters.txt (lambda sparsity parameter, dT time resolution, num-lags, kernel-width for the Gaussian kernel, and family='gaussian' for log-transformed count data X1=log(1+X) or 'poisson'), run SINGE GLG, avoiding lambda=0 which yields a non-sparse solution. Each GLG test applies Gaussian kernel smoothing over the irregular pseudotimes, then runs Lasso Granger regression via glmnet_matlab with per-gene independent subsampling of pseudotimestamps across ten replicates (prob-remove-samples default 0.2), optionally applying prob-zero-removal dropout-driven zero handling. Each hyperparameter set and replicate writes a partial adjacency output to the outdir, which SINGE Aggregate later merges with the modified Borda count. For branching trajectories, either combine branches in one run or store per-branch GLG results in separate folders and call SINGE Aggregate per branch.

## Related tools

- **SINGE** (Implements the GLG tests (SINGE_GLG_Test.m) and standalone pipeline (SINGE.m) for kernel-weighted Lasso Granger regression on pseudotemporal single-cell data) — https://github.com/gitter-lab/SINGE
- **glmnet_matlab** (Dependency used by SINGE GLG to fit the Lasso (Generalized Lasso Granger) regression models; must be unzipped in the SINGE root or code subdirectory) — http://web.stanford.edu/~hastie/glmnet_matlab/download.html
- **MATLAB R2018a** (Runtime environment for the standalone MATLAB and compiled Linux GLG executables)

## Examples

```
bash SINGE.sh PATH_TO_RUNTIME GLG data1/X_SCODE_data.mat data1/gene_list.mat Output default_hyperparameters.txt 2
```

## Evaluation signals

- Every hyperparameter combination in the hyperparameter file contributed a partial GLG network before aggregation
- Partial GLG outputs for all replicate indices are present in outdir before SINGE Aggregate is run
- Output matches the reference files in tests/reference (tests/compare_example_output.sh), or falls within the permissive threshold used for macOS OS-specific differences
- The final SINGE_Ranked_Edge_List.txt contains edges with SINGE scores and rows of X map to gene_list entries
- Re-running with identical inputs and the same --date hyperparameter reproduces the same result (seed derives from inputs including date)

## Limitations

- glmnet_matlab encounters memory segmentation violations with some settings, so lambda=0 (non-sparse solution) must be avoided
- Zero-removal and subsampling are pseudo-random; results are only reproducible when the same inputs, including the --date hyperparameter, are supplied
- SINGE can perform poorly for individual regulators and when pseudotimes are uninformative; branching trajectories are recommended to be split into sub-trajectories (branches) analyzed independently
- The master branch may be unstable; a versioned release should be used for stable analysis

## Evidence

- [abstract] Kernel-based Granger causality regression to smooth irregular pseudotimes and missing expression values: "SINGE uses kernel-based Granger causality regression to smooth irregular pseudotimes and missing expression values."
- [methods] Ensemble aggregation rationale: "SINGE's ensembling and modified Borda aggregation is motivated by stability selection from these networks."
- [methods] Family selection for count data: "we recommend log-transforming the count-based data (`X1 = log(1+X)`) and using the `--family gaussian` hyperparameter instead."
- [readme] GLG hyperparameters including lambda and kernel width: [evidence span withheld — anchor status: not_found]
- [readme] Reproducibility of subsampling via date input: [evidence span withheld — anchor status: external_doc]
