---
name: gene-regulatory-network-inference-from-pseudotemporal-single-cell-data
description: Use when input is pseudotemporal single-cell gene expression (a .mat
  file with sparse matrix X, ptime array, optional regix regulator indices) and the
  goal is a ranked list of transcriptional regulator-target interactions.
when_to_use_negative:
- The expression data have no pseudotime or trajectory ordering — SINGE's kernel-based
  Granger causality regression requires ordered single-cell data
- 'lambda = 0 must be excluded from the hyperparameter grid: it yields a non-sparse
  (complete graph) solution rather than a ranked, sparse network'
- Data are count-based and left untransformed with --family poisson for large datasets
  — glmnet_matlab encounters memory segmentation violations in this setting; log-transform
  and use --family gaussian instead
edam_operation: http://edamontology.org/operation_2436
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: SINGE
  role: 'Core algorithm: runs GLG (kernel-based Granger causality) tests and Aggregate
    (modified Borda) stages to produce the ranked network'
  repo: https://github.com/gitter-lab/SINGE
- name: glmnet_matlab
  role: MATLAB dependency used by the GLG regression stage; can trigger memory segmentation
    violations with count-based settings
  repo: http://web.stanford.edu/~hastie/glmnet_matlab/download.html
- name: dynverse/dynwrap
  role: Used to infer trajectories and generate or assign pseudotimes (add_pseudotime)
    for datasets lacking them
  repo: https://rdrr.io/github/dynverse/dynwrap/man/add_pseudotime.html
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.celrep.2022.110333
    title: Network inference with Granger causality ensembles on single-cell transcriptomics
  source_content_hash: sha256:072ef65c858c9ec99b817938c99d21f606e7478ee3a7df79b82dcc714d3796f0
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gitter-lab/SINGE
  tools:
  - MATLAB
  - SINGE
  - dynverse/dynwrap
  - glmnet_matlab
  edam_operation: http://edamontology.org/operation_2436
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7
n_claims: 10
---

# gene-regulatory-network-inference-from-pseudotemporal-single-cell-data

## Summary

Reconstruct gene regulatory networks from ordered single-cell expression data using SINGE, which applies kernel-based Granger causality regression (GLG) across an ensemble of hyperparameter and subsample replicates, then compiles a ranked regulator-target edge list via a modified Borda aggregation.

## When to use

Use when input is pseudotemporal single-cell gene expression (a .mat file with sparse matrix X, ptime array, optional regix regulator indices) and the goal is a ranked list of transcriptional regulator-target interactions.

## When NOT to use

- The expression data have no pseudotime or trajectory ordering — SINGE's kernel-based Granger causality regression requires ordered single-cell data
- lambda = 0 must be excluded from the hyperparameter grid: it yields a non-sparse (complete graph) solution rather than a ranked, sparse network
- Data are count-based and left untransformed with --family poisson for large datasets — glmnet_matlab encounters memory segmentation violations in this setting; log-transform and use --family gaussian instead

## Inputs

- .mat file with sparse expression matrix X, pseudotime array ptime, optional regulator indices regix, and optional branches matrix
- gene_list.mat file listing gene names corresponding to rows of X
- hyperparameter file (e.g., default_hyperparameters.txt) with GLG combinations of lambda, dT, num-lags, kernel-width, replicate, family, prob-zero-removal, prob-remove-samples, and date

## Outputs

- SINGE_Ranked_Edge_List.txt — ranked regulator-target edges by SINGE score
- SINGE_Gene_Influence.txt — genes ranked by SINGE influence

## How to apply

Load the expression matrix X, gene_list, and optional regulator index vector regix from the input .mat file (e.g., data1/X_SCODE_data.mat for a linear trajectory or data_bifurcated/X_data_bifurcated.mat for a branching one). For large count-based datasets, log-transform with X1 = log(1+X) and set the --family gaussian hyperparameter to avoid glmnet_matlab memory segmentation violations. Run SINGE in GLG mode across the hyperparameter combinations in default_hyperparameters.txt (lambda, dT, num-lags, kernel-width, replicate indices), using ten subsampled replicates per combination and avoiding lambda = 0, which yields a non-sparse complete graph. Then run SINGE Aggregate, which applies a modified Borda aggregation motivated by stability selection to combine the ensemble of partial network estimates into SINGE_Ranked_Edge_List.txt and SINGE_Gene_Influence.txt. For branching trajectories, results are combined across branches by default, or branch-specific GLG outputs can be aggregated separately. Reproducibility depends on supplying the same inputs including the --date hyperparameter, which seeds the pseudo-random subsampling and zero-removal stages.

## Related tools

- **SINGE** (Core algorithm: runs GLG (kernel-based Granger causality) tests and Aggregate (modified Borda) stages to produce the ranked network) — https://github.com/gitter-lab/SINGE
- **glmnet_matlab** (MATLAB dependency used by the GLG regression stage; can trigger memory segmentation violations with count-based settings) — http://web.stanford.edu/~hastie/glmnet_matlab/download.html
- **dynverse/dynwrap** (Used to infer trajectories and generate or assign pseudotimes (add_pseudotime) for datasets lacking them) — https://rdrr.io/github/dynverse/dynwrap/man/add_pseudotime.html

## Examples

```
bash SINGE.sh PATH_TO_RUNTIME standalone data1/X_SCODE_data.mat data1/gene_list.mat Output default_hyperparameters.txt
```

## Evaluation signals

- Output is a ranked (non-complete) edge list — confirm lambda = 0 was avoided so the solution is sparse
- Re-running with identical inputs including the same --date value reproduces the reference outputs in tests/reference/latest (tests/compare_example_output.sh)
- On the mouse ESC-to-endoderm differentiation datasets, the ranked network evaluated against ESCAPE reproduces SINGE's reported outperformance, with Pou5f1 prediction MSE averaged over 10 GLG runs with independent in-sample/out-sample splits
- SINGE_Ranked_Edge_List.txt and SINGE_Gene_Influence.txt are both produced by the Aggregate stage
- Docker/GitHub Actions test comparison of generated vs. reference adjacency matrices passes (macOS uses a more permissive threshold due to OS-specific differences)

## Limitations

- SINGE shows poor performance for individual regulators and with uninformative pseudotimes
- The glmnet_matlab package encounters memory segmentation violations with count-based (Poisson) settings on large datasets
- The master branch of the repository may be unstable; use a versioned release (Zenodo DOI 10.5281/zenodo.2549817) for stable analysis
- Branching trajectories combine all branches by default in v0.5.0 outputs; branch-specific networks require storing GLG results per branch and re-running Aggregate
- No compiled executables for Windows — use Docker if a compatible MATLAB environment is unavailable

## Evidence

- [abstract] SINGE uses kernel-based Granger causality regression to smooth irregular pseudotimes and missing expression values.: "SINGE uses kernel-based Granger causality regression to smooth irregular pseudotimes and missing expression values."
- [methods] Ensembling and aggregation: "SINGE's ensembling and modified Borda aggregation is motivated by stability selection from these networks."
- [methods] Log-transform recommendation: "we recommend log-transforming the count-based data (`X1 = log(1+X)`) and using the `--family gaussian` hyperparameter instead."
- [readme] Lambda zero warning: [evidence span withheld — anchor status: external_doc]
- [readme] Output files: [evidence span withheld — anchor status: external_doc]
- [readme] Reproducibility note: [evidence span withheld — anchor status: external_doc]
