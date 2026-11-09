---
name: gene-regulatory-network-inference
description: Use when you have pseudotime-ordered single-cell expression data (a .mat
  file with X, ptime, gene_list, optional regix/branches) and need a ranked list of
  directed regulator-gene interactions.
when_to_use_negative:
- Data lack meaningful pseudotemporal ordering — uninformative (shuffled) pseudotimes
  degrade GRN reconstruction, as shown by the pseudotime-variant experiments against
  SINCERITIES and SCODE
- lambda=0 is required (a complete non-sparse graph is desired) — the article warns
  this produces a non-sparse solution, is impractical over thousands of genes, and
  increases segmentation-violation risk in glmnet_matlab
- You need time-series bulk data or non-transcriptomic networks — SINGE is designed
  for ordered single-cell gene expression data
edam_operation: http://edamontology.org/operation_3927
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: SINGE
  role: 'Main algorithm: SINGE GLG runs kernel-based Granger causality regression
    per hyperparameter/replicate; SINGE Aggregate merges partial networks with modified
    Borda count'
  repo: https://github.com/gitter-lab/SINGE
- name: glmnet_matlab
  role: Backend Lasso regression package used by GLG tests; can hit memory segmentation
    violations on large data or lambda=0
- name: MATLAB R2018a
  role: Runtime for the standalone MATLAB mode and the compiled Linux executables
    (R2018a runtime)
  repo: https://github.com/gitter-lab/SINGE
- name: dynverse
  role: Used to infer trajectories/pseudotimes for cells before running SINGE on a
    new dataset
- name: dynwrap
  role: Provides add_pseudotime to assign pseudotemporal values after the fact when
    the chosen trajectory method does not supply them
provenance:
  source_task_ids:
  - task_003
  - task_004
  - task_005
  - task_007
  source_papers:
  - doi: 10.1016/j.celrep.2022.110333
    title: Network inference with Granger causality ensembles on single-cell transcriptomics
  source_content_hash: sha256:d53ea5fce5f837804b944b47233c484e24b97f864f287a92f7110bcea63d68eb
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gitter-lab/SINGE
  tools:
  - MATLAB R2018a
  - SINGE
  - dynverse
  - dynwrap
  - glmnet_matlab
  edam_operation: http://edamontology.org/operation_3927
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.45454545454545453
n_claims: 11
---

# gene-regulatory-network-inference

## Summary

Reconstruct gene regulatory networks from pseudotemporally ordered single-cell gene expression data using SINGE, which runs kernel-based Granger causality regression (GLG) across hyperparameter combinations and subsampled replicates, then aggregates partial networks with a modified Borda count into a ranked regulator-target edge list.

## When to use

Use when you have pseudotime-ordered single-cell expression data (a .mat file with X, ptime, gene_list, optional regix/branches) and need a ranked list of directed regulator-gene interactions.

## When NOT to use

- Data lack meaningful pseudotemporal ordering — uninformative (shuffled) pseudotimes degrade GRN reconstruction, as shown by the pseudotime-variant experiments against SINCERITIES and SCODE
- lambda=0 is required (a complete non-sparse graph is desired) — the article warns this produces a non-sparse solution, is impractical over thousands of genes, and increases segmentation-violation risk in glmnet_matlab
- You need time-series bulk data or non-transcriptomic networks — SINGE is designed for ordered single-cell gene expression data

## Inputs

- Ordered single-cell expression .mat file with sparse matrix X, pseudotime array ptime, optional regulator indices regix, and optional branching matrix branches (e.g. data1/X_SCODE_data.mat for a linear trajectory or data_bifurcated/X_data_bifurcated.mat for a branching trajectory)
- gene_list .mat file with gene names corresponding to rows of X
- Hyperparameter file listing GLG hyperparameter combinations (e.g. default_hyperparameters.txt, generable via scripts/generate_hyperparameters.sh)
- Optional MATLAB runtime directory when running compiled executables

## Outputs

- SINGE_Ranked_Edge_List.txt — ranked regulator-target edges by SINGE score
- SINGE_Gene_Influence.txt — genes ranked by SINGE influence
- Per-hyperparameter/per-replicate partial GLG network outputs in the outdir (for parallel GLG mode)

## How to apply

Rescale the pseudotimes to the 0-100 range required by the SINGE Gaussian kernel smoothing, then run the complete workflow (SINGE GLG followed by SINGE Aggregate). For each hyperparameter combination in the grid from default_hyperparameters.txt (lambda, dT, num-lags, kernel-width, family, prob-zero-removal=0, prob-remove-samples=0.2) and its ten subsampled replicates, GLG performs kernel-based Lasso Granger regression via glmnet_matlab; avoid lambda=0 since it yields a non-sparse complete graph and risks glmnet_matlab memory segmentation violations. For count-based data, log-transform as X1 = log(1+X) and use --family gaussian instead of poisson. Finally, SINGE Aggregate merges all partial GLG ranked edge lists with the modified Borda count weighting (stability-selection-style ensembling) into SINGE_Ranked_Edge_List.txt and SINGE_Gene_Influence.txt. For branching trajectories, either supply the branches matrix so results are combined across branches, or store per-branch GLG outputs separately and aggregate for branch-specific networks.

## Related tools

- **SINGE** (Main algorithm: SINGE GLG runs kernel-based Granger causality regression per hyperparameter/replicate; SINGE Aggregate merges partial networks with modified Borda count) — https://github.com/gitter-lab/SINGE
- **glmnet_matlab** (Backend Lasso regression package used by GLG tests; can hit memory segmentation violations on large data or lambda=0)
- **MATLAB R2018a** (Runtime for the standalone MATLAB mode and the compiled Linux executables (R2018a runtime)) — https://github.com/gitter-lab/SINGE
- **dynverse** (Used to infer trajectories/pseudotimes for cells before running SINGE on a new dataset)
- **dynwrap** (Provides add_pseudotime to assign pseudotemporal values after the fact when the chosen trajectory method does not supply them)

## Examples

```
docker run -v $(pwd):/SINGE -w /SINGE agitter/singe:0.5.1 standalone data1/X_SCODE_data.mat data1/gene_list.mat Output default_hyperparameters.txt
```

## Evaluation signals

- Every hyperparameter combination in the hyperparameter file contributed a partial GLG network before aggregation
- Aggregated output is a single ranked edge list (SINGE_Ranked_Edge_List.txt) whose scores decrease monotonically by rank
- Re-running with identical inputs and the same --date reproduces the prior result, since subsampling and zero-removal seeds derive from the hyperparameters including the date
- Generated example output matches the reference output in tests/reference/latest (tests/compare_example_output.sh), with permissive threshold on macOS for OS-specific differences
- No GLG run used lambda=0 producing a complete graph; predictions restricted to regix regulators if a regulator list was supplied

## Limitations

- Subsampling and zero-removal stages involve pseudo-random sample removals; results are only reproducible when the same inputs and date are provided
- glmnet_matlab encounters memory segmentation violations for large datasets (mitigated by log-transforming counts and using --family gaussian, and avoiding lambda=0)
- Performance is poor for individual regulators and under uninformative pseudotimes; quality of pseudotime assignments directly affects inference
- For branching trajectories, v0.5.0 combines results across branches by default — branch-specific networks require storing per-branch GLG outputs and aggregating separately
- The master branch of the repository may be unstable; use a versioned release for stable analysis

## Evidence

- [abstract] SINGE uses kernel-based Granger causality regression to smooth irregular pseudotimes and missing expression values.: "SINGE uses kernel-based Granger causality regression to smooth irregular pseudotimes and missing expression values."
- [methods] Ensemble aggregation motivation: "SINGE's ensembling and modified Borda aggregation is motivated by stability selection from these networks."
- [readme] Input file specification: [evidence span withheld — anchor status: external_doc]
- [readme] Reproducibility via date-seeded randomness: [evidence span withheld — anchor status: external_doc]
- [readme] Output file description: [evidence span withheld — anchor status: external_doc]
- [abstract] Caveat about pseudotime quality: "However, a more detailed examination reveals caveats about poor performance for individual regulators and uninformative pseudotimes."
