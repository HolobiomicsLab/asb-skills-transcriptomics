---
name: stability-selection-ensembling
description: Use when inferring gene regulatory networks from pseudotemporally ordered
  single-cell expression data (.mat with sparse X, ptime, optional regix/branches)
  where single regression runs are unstable and ensembling with modified Borda aggregation
  is needed.
when_to_use_negative:
- The dataset has no meaningful pseudotemporal ordering — the paper notes caveats
  about poor performance with uninformative pseudotimes.
- A single-network fit suffices and the ensemble/aggregation overhead (many GLG tests
  over lambda/kernel-width/lag grids and replicates) is unwarranted for a small dataset.
- The expression matrix has more than 1000 genes and no sparsity can be tolerated
  — lambda=0 must be avoided, so a non-sparse regression request is out of scope.
edam_operation: http://edamontology.org/operation_0564
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: SINGE
  role: Provides the GLG test and Aggregate modes whose ensemble of regressions is
    merged with modified Borda aggregation
  repo: https://github.com/gitter-lab/SINGE
- name: glmnet_matlab
  role: Backend for the kernel-based Lasso Granger regression run within each SINGE
    GLG test
  repo: http://web.stanford.edu/~hastie/glmnet_matlab/download.html
- name: MATLAB R2018a
  role: Runtime environment in which the standalone SINGE pipeline and compiled executables
    are tested
provenance:
  source_task_ids:
  - task_007
  - task_006
  source_papers:
  - doi: 10.1016/j.celrep.2022.110333
    title: Network inference with Granger causality ensembles on single-cell transcriptomics
  source_content_hash: sha256:f79f8cd47a21225bd2f10df67cf61199fdf7c25940e976b547ba122b411925d4
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gitter-lab/SINGE
  tools:
  - MATLAB R2018a
  - SINGE
  - glmnet_matlab
  edam_operation: http://edamontology.org/operation_0564
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# stability-selection-ensembling

## Summary

Runs SINGE's ensemble pipeline: many Generalized Lasso Granger (GLG) tests over hyperparameter combinations and subsampled pseudotime replicates, then merges the ranked edge lists with a modified Borda count to produce a stable ranked regulator-target network.

## When to use

Use when inferring gene regulatory networks from pseudotemporally ordered single-cell expression data (.mat with sparse X, ptime, optional regix/branches) where single regression runs are unstable and ensembling with modified Borda aggregation is needed.

## When NOT to use

- The dataset has no meaningful pseudotemporal ordering — the paper notes caveats about poor performance with uninformative pseudotimes.
- A single-network fit suffices and the ensemble/aggregation overhead (many GLG tests over lambda/kernel-width/lag grids and replicates) is unwarranted for a small dataset.
- The expression matrix has more than 1000 genes and no sparsity can be tolerated — lambda=0 must be avoided, so a non-sparse regression request is out of scope.

## Inputs

- .mat file with sparse expression matrix X, pseudotime array ptime, optional regulator indices regix, optional branches matrix (e.g., data1/X_SCODE_data.mat or data_bifurcated/X_data_bifurcated.mat)
- gene_list file with gene names matching rows of X (e.g., data1/gene_list.mat)
- hyperparameter file of GLG combinations (default_hyperparameters.txt) with --ID, --lambda, --dT, --num-lags, --kernel-width, --replicate, --family, --prob-zero-removal, --prob-remove-samples, --date
- output directory for individual GLG test results

## Outputs

- SINGE_Ranked_Edge_List.txt — ranked regulator-target edges with SINGE scores
- SINGE_Gene_Influence.txt — genes ranked by SINGE influence
- per-hyperparameter/per-replicate partial GLG adjacency outputs in the outdir

## How to apply

Load the expression matrix X, gene_list, and rescaled pseudotimes (0-100 range) from the input .mat file, optionally including regix candidate regulators and branches for bifurcated trajectories. For each hyperparameter combination in default_hyperparameters.txt (lambda, kernel-width, dT, num-lags, family, prob-zero-removal, prob-remove-samples), run SINGE GLG: kernel-based Lasso Granger regression via glmnet_matlab with per-gene independent subsampling of pseudotimestamps across ten replicates, avoiding lambda=0 (non-sparse). For count data that causes glmnet_matlab memory segmentation violations with --family poisson, log-transform with X1 = log(1+X) and use --family gaussian. Then run SINGE Aggregate over all GLG outputs, applying the modified Borda count — motivated by stability selection — to merge the ranked edge lists into a single regulatory network; for branching trajectories, GLG results from all branches are combined, or per-branch aggregation can be requested by storing each branch's GLG results separately.

## Related tools

- **SINGE** (Provides the GLG test and Aggregate modes whose ensemble of regressions is merged with modified Borda aggregation) — https://github.com/gitter-lab/SINGE
- **glmnet_matlab** (Backend for the kernel-based Lasso Granger regression run within each SINGE GLG test) — http://web.stanford.edu/~hastie/glmnet_matlab/download.html
- **MATLAB R2018a** (Runtime environment in which the standalone SINGE pipeline and compiled executables are tested)

## Examples

```
bash SINGE.sh PATH_TO_RUNTIME standalone data1/X_SCODE_data.mat data1/gene_list.mat Output default_hyperparameters.txt
```

## Evaluation signals

- Every hyperparameter combination in default_hyperparameters.txt contributed a partial GLG network before aggregation (check outdir contents).
- The aggregated output files SINGE_Ranked_Edge_List.txt and SINGE_Gene_Influence.txt exist and are ranked by SINGE score/influence.
- Rerunning with the same inputs and --date reproduces the same output, since the random seed derives from input hyperparameters including date.
- Docker-based test comparison of generated output against tests/reference/latest matches (macOS uses a more permissive threshold due to OS-specific differences).
- On dyngen benchmarks, two-to-five-replicate ensembles show only moderate precision-recall degradation versus the ten-replicate reference.

## Limitations

- Pseudo-random subsampling and zero-removal require the same hyperparameters (including --date) to reproduce results; the master branch may be unstable — use versioned releases.
- lambda=0 yields a non-sparse solution and must be avoided; glmnet_matlab can encounter memory segmentation violations with --family poisson, requiring log(1+X) counts with --family gaussian.
- SINGE can perform poorly for individual regulators and when pseudotimes are uninformative.
- On Windows, no compiled executables exist — Docker is recommended when a compatible MATLAB environment is unavailable.

## Evidence

- [methods] GLG tests across all hyperparameter combinations: [evidence span withheld — anchor status: not_found]
- [methods] modified Borda aggregation: "SINGE's ensembling and modified Borda aggregation is motivated by stability selection from these networks."
- [abstract] kernel-based Granger causality regression: "SINGE uses kernel-based Granger causality regression to smooth irregular pseudotimes and missing expression values."
- [readme] compiled with MATLAB R2018a runtime: [evidence span withheld — anchor status: external_doc]
- [methods] log-transform counts gaussian: "we recommend log-transforming the count-based data (`X1 = log(1+X)`) and using the `--family gaussian` hyperparameter instead."
- [readme] reproducibility via date: "SINGE generates a random seed using input hyperparameters, including the *date* input.
The results can be reproduced by providing the same inputs and date from a previous experiment."
