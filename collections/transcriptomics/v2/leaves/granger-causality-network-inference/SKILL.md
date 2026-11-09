---
name: granger-causality-network-inference
description: Use when you have single-cell expression data ordered by pseudotime (e.g.,
  a .mat file with sparse X, ptime, optional regix/branches) and need to rank candidate
  regulator-target interactions as a directed network.
when_to_use_negative:
- Input expression data lack any pseudotemporal ordering — SINGE relies on ordered
  single-cell data by design
- The goal is gold-standard evaluation against indirect/transitive interactions only;
  the dyngen gold standard contains only direct regulator-to-target edges and extended
  standards must be built separately
- glmnet_matlab segmentation violations cannot be mitigated (e.g., no ability to log-transform
  counts or switch --family from poisson to gaussian)
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0602
- http://edamontology.org/topic_3308
tools:
- name: SINGE
  role: Runs the GLG Granger causality tests and modified Borda aggregation to produce
    the ranked regulatory network
  repo: https://github.com/gitter-lab/SINGE
- name: glmnet_matlab
  role: Dependency powering the generalized lasso Granger regression inside SINGE
    GLG; may hit segmentation violations with poisson family on large datasets
  repo: http://web.stanford.edu/~hastie/glmnet_matlab/download.html
- name: dynwrap add_pseudotime
  role: Assigns pseudotime values to cells after trajectory inference so SINGE has
    ordered input
  repo: https://rdrr.io/github/dynverse/dynwrap/man/add_pseudotime.html
provenance:
  source_task_ids:
  - task_006
  - task_001
  source_papers:
  - doi: 10.1016/j.celrep.2022.110333
    title: Network inference with Granger causality ensembles on single-cell transcriptomics
  source_content_hash: sha256:0811f1868a191485d688e165172fd2666ebb30699a97f9cad23a8fef2f9eb37b
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gitter-lab/SINGE
  tools:
  - MATLAB R2018a
  - SINGE
  - dynwrap add_pseudotime
  - glmnet_matlab
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0602
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6666666666666666
n_claims: 9
---

# granger-causality-network-inference

## Summary

Infers gene regulatory networks from pseudotemporal single-cell gene expression data using SINGE's kernel-based Granger causality regression (GLG) followed by modified Borda aggregation over an ensemble of subsampled regression analyses.

## When to use

Use when you have single-cell expression data ordered by pseudotime (e.g., a .mat file with sparse X, ptime, optional regix/branches) and need to rank candidate regulator-target interactions as a directed network.

## When NOT to use

- Input expression data lack any pseudotemporal ordering — SINGE relies on ordered single-cell data by design
- The goal is gold-standard evaluation against indirect/transitive interactions only; the dyngen gold standard contains only direct regulator-to-target edges and extended standards must be built separately
- glmnet_matlab segmentation violations cannot be mitigated (e.g., no ability to log-transform counts or switch --family from poisson to gaussian)

## Inputs

- .mat file with ordered single-cell expression data (sparse matrix X), pseudotime values (ptime array), optional regulator indices (regix), and optional branching matrix (branches)
- gene_list file (e.g., data1/gene_list.mat) mapping gene names to rows of X
- hyperparameter file listing GLG hyperparameter combinations (e.g., default_hyperparameters.txt)
- MATLAB runtime directory path when using compiled executables

## Outputs

- SINGE_Ranked_Edge_List.txt — ranked candidate regulator-target edges with SINGE scores
- SINGE_Gene_Influence.txt — genes ranked by SINGE influence
- Per-GLG-test outputs in the specified outdir, consumed by the aggregation step

## How to apply

Run the complete SINGE workflow: SINGE GLG applies kernel-based Granger causality regression over hyperparameter combinations from default_hyperparameters.txt (lambda, dT, num-lags, kernel-width, --family, replicate indices), using ten subsampled replicates per combination (or two-to-five replicates to reduce runtime) with --prob-remove-samples defaulting to 0.2. SINGE Aggregate then compiles a single ranked network per dataset using modified Borda aggregation over all GLG test outputs, motivated by stability selection. For large count-based datasets (e.g., the retinoic acid scRNA-seq matrix), log-transform counts (X1 = log(1+X)) and use --family gaussian to avoid glmnet_matlab memory segmentation violations, and avoid lambda=0 for datasets with more than 1000 genes since it yields a non-sparse solution. For branching trajectories, either let SINGE v0.5.0 combine results across branches or store per-branch GLG outputs separately and call SINGE Aggregate for branch-specific inference. Reproducibility is achieved by fixing the --date hyperparameter, since subsampling and zero-removal draw pseudo-random samples seeded from the hyperparameters.

## Related tools

- **SINGE** (Runs the GLG Granger causality tests and modified Borda aggregation to produce the ranked regulatory network) — https://github.com/gitter-lab/SINGE
- **glmnet_matlab** (Dependency powering the generalized lasso Granger regression inside SINGE GLG; may hit segmentation violations with poisson family on large datasets) — http://web.stanford.edu/~hastie/glmnet_matlab/download.html
- **dynwrap add_pseudotime** (Assigns pseudotime values to cells after trajectory inference so SINGE has ordered input) — https://rdrr.io/github/dynverse/dynwrap/man/add_pseudotime.html

## Examples

```
docker run -v $(pwd):/SINGE -w /SINGE agitter/singe:0.5.1 standalone data1/X_SCODE_data.mat data1/gene_list.mat Output default_hyperparameters.txt
```

## Evaluation signals

- SINGE_Ranked_Edge_List.txt and SINGE_Gene_Influence.txt are produced in the outdir after aggregation
- On the retinoic acid case study, average precision and early precision against the ESCAPE gold standard should exceed the four baseline methods (GENIE3, SCODE, SINCERITIES, Jump3)
- Two-to-five-replicate ensembles show only moderate precision-recall degradation versus the ten-replicate reference on dyngen-simulated data
- Re-running with identical inputs and --date reproduces the same results, since seeds derive from the hyperparameters including date
- Docker-based test comparison of generated output against tests/reference passes (tests/compare_example_output.sh)

## Limitations

- glmnet_matlab encounters memory segmentation violations with the poisson family on large count datasets; use log(1+X) with --family gaussian instead
- lambda = 0 results in a non-sparse solution and should be avoided for datasets with more than 1000 genes
- More detailed examination reveals caveats about poor performance for individual regulators and uninformative pseudotimes
- SINGE may not work in all Windows environments; no compiled executables exist for Windows, so Docker is recommended there
- The master branch of the repository may be unstable; use a versioned release for stable analysis

## Evidence

- [abstract] SINGE uses kernel-based Granger causality regression to smooth irregular pseudotimes and missing expression values.: "SINGE uses kernel-based Granger causality regression to smooth irregular pseudotimes and missing expression values."
- [methods] we recommend log-transforming the count-based data (`X1 = log(1+X)`) and using the `--family gaussian` hyperparameter instead.: "we recommend log-transforming the count-based data (`X1 = log(1+X)`) and using the `--family gaussian` hyperparameter instead."
- [methods] SINGE's ensembling and modified Borda aggregation is motivated by stability selection from these networks.: "SINGE's ensembling and modified Borda aggregation is motivated by stability selection from these networks."
- [readme] The complete SINGE workflow includes the SINGE GLG and SINGE Aggregate modes.: [evidence span withheld — anchor status: external_doc]
- [readme] SINGE generates a random seed using input hyperparameters, including the date input, enabling reproducibility.: [evidence span withheld — anchor status: external_doc]
- [abstract] In two mouse embryonic stem cell differentiation datasets, SINGE outperforms other contemporary algorithms.: "In two mouse embryonic stem cell differentiation datasets, SINGE outperforms other contemporary algorithms."
