---
name: kernel-weighted-lasso-regression-ensembling
description: Use when you have single-cell gene expression data ordered along pseudotime
  (a .mat file with matrix X, ptime array, gene list) and need to reconstruct a gene
  regulatory network as ranked transcription-regulator-to-target interactions.
when_to_use_negative:
- Data are not ordered along a meaningful pseudotime — the paper warns of poor performance
  for uninformative pseudotimes
- The trajectory branches but you want branch-specific networks without splitting
  GLG results per branch (branch results are combined by default in SINGE v0.5.0)
- Expression data are not count-based and you plan to log-transform anyway — the --family
  gaussian + log-transform recommendation targets count data with glmnet_matlab memory
  issues
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: SINGE
  role: Implements the GLG tests (SINGE GLG) and modified Borda ensemble aggregation
    (SINGE Aggregate); run standalone, via compiled executables, or Docker
  repo: https://github.com/gitter-lab/SINGE
- name: glmnet_matlab
  role: Dependency used by GLG regression for the sparse lasso fits; avoid settings
    that trigger its memory segmentation violations
  repo: http://web.stanford.edu/~hastie/glmnet_matlab/download.html
- name: MATLAB R2018a
  role: Runtime environment for the SINGE MATLAB source and the Linux-compiled executables
    (R2020a runtime for macOS)
provenance:
  source_task_ids:
  - task_001
  source_papers:
  - doi: 10.1016/j.celrep.2022.110333
    title: Network inference with Granger causality ensembles on single-cell transcriptomics
  source_content_hash: sha256:9ec3987f4b66b515fa3ae2a3d7fc99f8ac43c1b356b40972d54a045003f5cf09
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
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6
n_claims: 10
---

# kernel-weighted-lasso-regression-ensembling

## Summary

Runs the SINGE workflow: kernel-based Generalized Lasso Granger (GLG) regression on pseudotime-ordered single-cell expression data, followed by modified Borda aggregation of an ensemble of subsampled GLG networks into a ranked regulator-target edge list.

## When to use

Use when you have single-cell gene expression data ordered along pseudotime (a .mat file with matrix X, ptime array, gene list) and need to reconstruct a gene regulatory network as ranked transcription-regulator-to-target interactions.

## When NOT to use

- Data are not ordered along a meaningful pseudotime — the paper warns of poor performance for uninformative pseudotimes
- The trajectory branches but you want branch-specific networks without splitting GLG results per branch (branch results are combined by default in SINGE v0.5.0)
- Expression data are not count-based and you plan to log-transform anyway — the --family gaussian + log-transform recommendation targets count data with glmnet_matlab memory issues

## Inputs

- .mat file with ordered single-cell expression data (sparse matrix X), pseudotime values (ptime), optional regulator indices (regix), optional branches matrix
- gene_list file (.mat) mapping gene names to rows of X
- hyperparameter file (e.g., default_hyperparameters.txt) listing GLG hyperparameter combinations (lambda, dT, num-lags, kernel-width, replicate, family, prob-zero-removal, prob-remove-samples, date)

## Outputs

- SINGE_Ranked_Edge_List.txt — ranked candidate regulator-target interactions by SINGE score
- SINGE_Gene_Influence.txt — genes ranked by SINGE influence
- Per-GLG-test output directories from individual ensemble members

## How to apply

Order the expression matrix by pseudotime (e.g., Monocle 2), then run SINGE GLG followed by SINGE Aggregate, either serially via SINGE.m or through the SINGE.sh/Docker wrappers. For count-based data, log-transform (X1 = log(1+X)) and set --family gaussian to avoid glmnet_matlab segmentation violations seen with other families. Build the hyperparameter file with generate_hyperparameters.sh, sweeping lambda (never lambda = 0, which gives a non-sparse solution), kernel-width, dT, and num-lags, with prob-remove-samples (default 0.2) generating ten subsampled replicates per combination; the --date input seeds the pseudo-random subsampling so results are reproducible. Aggregate the partial GLG networks with modified Borda aggregation into SINGE_Ranked_Edge_List.txt and SINGE_Gene_Influence.txt, then score the ranked edges against a gold-standard network (e.g., ESCAPE) using average precision and early precision.

## Related tools

- **SINGE** (Implements the GLG tests (SINGE GLG) and modified Borda ensemble aggregation (SINGE Aggregate); run standalone, via compiled executables, or Docker) — https://github.com/gitter-lab/SINGE
- **glmnet_matlab** (Dependency used by GLG regression for the sparse lasso fits; avoid settings that trigger its memory segmentation violations) — http://web.stanford.edu/~hastie/glmnet_matlab/download.html
- **MATLAB R2018a** (Runtime environment for the SINGE MATLAB source and the Linux-compiled executables (R2020a runtime for macOS))

## Examples

```
docker run -v $(pwd):/SINGE -w /SINGE agitter/singe:0.5.1 standalone data1/X_SCODE_data.mat data1/gene_list.mat Output default_hyperparameters.txt
```

## Evaluation signals

- Output directory contains SINGE_Ranked_Edge_List.txt and SINGE_Gene_Influence.txt with edges ranked by SINGE score
- Edge ranking against the ESCAPE gold standard yields average precision and early precision values exceeding baseline methods (GENIE3, SCODE, SINCERITIES, Jump3) on the retinoic acid mESC dataset
- Re-running with identical inputs, hyperparameters, and --date reproduces the reference adjacency matrices (as verified by tests/compare_example_output.sh)
- No lambda = 0 hyperparameter combination was used; every GLG run is sparse
- Ensemble contains the expected number of GLG tests (e.g., ten subsampled replicates per hyperparameter combination in default_hyperparameters.txt)

## Limitations

- glmnet_matlab encounters memory segmentation violations with certain settings, motivating the log-transform + --family gaussian workaround
- Detailed examination reveals caveats about poor performance for individual regulators and uninformative pseudotimes
- Branching trajectories are combined across all branches by default in v0.5.0; branch-specific networks require saving GLG results per branch in separate folders
- The master branch of the repository may be unstable; use a versioned release for reproducible analysis
- Results depend on the pseudo-random seed derived from hyperparameters including the date input

## Evidence

- [abstract] kernel-based Granger causality regression on pseudotimes: "SINGE uses kernel-based Granger causality regression to smooth irregular pseudotimes and missing expression values."
- [abstract] ensemble aggregation into ranked interactions: "It aggregates predictions from an ensemble of regression analyses to compile a ranked list of candidate interactions between transcriptional regulators and target genes."
- [methods] log-transform and gaussian family recommendation: "we recommend log-transforming the count-based data (`X1 = log(1+X)`) and using the `--family gaussian` hyperparameter instead."
- [methods] glmnet_matlab failure mode: "the glmnet_matlab package encounters memory segmentation violations with this setting"
- [readme] reproducibility via date-seeded random removals: [evidence span withheld — anchor status: external_doc]
- [methods] Borda aggregation motivation: "SINGE's ensembling and modified Borda aggregation is motivated by stability selection from these networks."
