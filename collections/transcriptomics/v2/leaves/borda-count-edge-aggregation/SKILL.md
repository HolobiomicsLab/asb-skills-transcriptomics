---
name: borda-count-edge-aggregation
description: Use when multiple ranked directed networks from ensemble Granger regression
  runs (e.g., SINGE GLG outputs across lambda, kernel-width, lag, family, and replicate
  settings) must be merged into one ranked edge list.
when_to_use_negative:
- Only one network inference run is available (a single GLG output) — the ensemble/Borda
  aggregation has nothing to combine and stability-selection motivation does not apply
- The trajectory has no pseudotemporal ordering of single-cell expression data, so
  kernel-based Granger regression and its ensemble are not appropriate
- lambda=0 hyperparameter sets were run — these produce non-sparse solutions and should
  be excluded before aggregation
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_2258
tools:
- name: SINGE
  role: Provides the SINGE Aggregate mode that applies the modified Borda count to
    merge ranked GLG edge lists into the final network
  repo: https://github.com/gitter-lab/SINGE
- name: glmnet_matlab
  role: Performs the kernel-based Lasso Granger (GLG) regression whose per-hyperparameter
    ranked outputs feed the Borda aggregation
  repo: http://web.stanford.edu/~hastie/glmnet_matlab/download.html
- name: MATLAB R2018a
  role: Runtime environment for running SINGE_GLG_Test.m and SINGE_Aggregate.m standalone
    or as compiled executables
provenance:
  source_task_ids:
  - task_007
  source_papers:
  - doi: 10.1016/j.celrep.2022.110333
    title: Network inference with Granger causality ensembles on single-cell transcriptomics
  source_content_hash: sha256:848984e951e88c3af4108670b0cd5b919730c9e39706f4c906cdf15591f0a56d
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
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_2258
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.42857142857142855
n_claims: 7
---

# borda-count-edge-aggregation

## Summary

SINGE's aggregation stage (SINGE Aggregate) merges the ranked regulator-target edge lists produced by every GLG test across hyperparameter combinations and subsampled replicates into a single final network using a modified Borda count weighting rule motivated by stability selection.

## When to use

Use when multiple ranked directed networks from ensemble Granger regression runs (e.g., SINGE GLG outputs across lambda, kernel-width, lag, family, and replicate settings) must be merged into one ranked edge list.

## When NOT to use

- Only one network inference run is available (a single GLG output) — the ensemble/Borda aggregation has nothing to combine and stability-selection motivation does not apply
- The trajectory has no pseudotemporal ordering of single-cell expression data, so kernel-based Granger regression and its ensemble are not appropriate
- lambda=0 hyperparameter sets were run — these produce non-sparse solutions and should be excluded before aggregation

## Inputs

- Directory of partial GLG test outputs (ranked edge lists / adjacency matrices) from all hyperparameter combinations and replicates
- .mat file with sparse expression matrix X, pseudotime array ptime, optional regulator indices regix, optional branches matrix
- gene_list.mat gene names file corresponding to rows of X
- Hyperparameter file (e.g., default_hyperparameters.txt) with unique --ID per set/replicate

## Outputs

- SINGE_Ranked_Edge_List.txt — regulator-target edges ranked by SINGE score
- SINGE_Gene_Influence.txt — genes ranked by SINGE influence
- Optionally saved as singe_ranked_edges.csv for downstream use

## How to apply

Run SINGE GLG over all hyperparameter combinations (lambda, dT, num-lags, kernel-width, family, prob-zero-removal, prob-remove-samples) with per-gene independent subsampling of pseudotimestamps across ten replicates, saving each partial adjacency output to the GLG output directory. Confirm every hyperparameter set (avoiding lambda=0, which yields a non-sparse solution) contributed a partial GLG network before aggregation. Then run SINGE Aggregate over all GLG outputs, which applies the modified Borda count to combine the ranked edge lists; for branching trajectories, it combines results from all branches unless branch-specific folders are aggregated separately. The aggregated output is SINGE_Ranked_Edge_List.txt (edges ranked by SINGE score) and SINGE_Gene_Influence.txt (genes ranked by SINGE influence). Reproducibility depends on the pseudo-random subsampling seeds derived from the hyperparameters, including the date field, so identical inputs and date reproduce identical results.

## Related tools

- **SINGE** (Provides the SINGE Aggregate mode that applies the modified Borda count to merge ranked GLG edge lists into the final network) — https://github.com/gitter-lab/SINGE
- **glmnet_matlab** (Performs the kernel-based Lasso Granger (GLG) regression whose per-hyperparameter ranked outputs feed the Borda aggregation) — http://web.stanford.edu/~hastie/glmnet_matlab/download.html
- **MATLAB R2018a** (Runtime environment for running SINGE_GLG_Test.m and SINGE_Aggregate.m standalone or as compiled executables)

## Examples

```
bash SINGE.sh PATH_TO_RUNTIME Aggregate data1/X_SCODE_data.mat data1/gene_list.mat Output
```

## Evaluation signals

- Every hyperparameter combination in the grid contributed a partial GLG network before aggregation (no missing GLG outputs)
- SINGE_Ranked_Edge_List.txt and SINGE_Gene_Influence.txt exist and are ordered by SINGE score/influence
- Re-running with identical inputs and the same date hyperparameter reproduces the same aggregated ranked edge list
- Aggregated output matches reference output within the test comparison threshold (tests/compare_example_output.sh against tests/reference/latest)
- For branching data, edge rankings reflect combination of GLG results across all branches, or branch-specific aggregates are produced when branch folders are aggregated separately

## Limitations

- Subsampling and zero-removal stages use pseudo-random sample removals; results are only reproducible if the same inputs and date hyperparameter are supplied
- On branching trajectories, default v0.5.0 output combines all branches rather than giving branch-specific networks unless branch results are stored and aggregated separately
- SINGE aggregation has caveats about poor performance for individual regulators and uninformative pseudotimes
- lambda=0 hyperparameter sets yield non-sparse solutions and glmnet_matlab can encounter memory segmentation violations with certain settings

## Evidence

- [methods] SINGE produces its final ranked network by running GLG tests across all hyperparameter combinations and subsampled replicates, then aggregating the resulting ranked edge lists using a modification of the Borda count.: [evidence span withheld — anchor status: not_found]
- [readme] The full SINGE pipeline runs multiple Generalized Lasso Granger (GLG) tests to infer different directed networks for different hyperparameters and subsamples of the data. These directed networks are then aggregated into a final predicted network.: [evidence span withheld — anchor status: external_doc]
- [methods] SINGE's ensembling and modified Borda aggregation is motivated by stability selection from these networks.: "SINGE's ensembling and modified Borda aggregation is motivated by stability selection from these networks."
- [readme] the SINGE_Ranked_Edge_List.txt and SINGE_Gene_Influence.txt are calculated for the entire branching process by combining the results of the individual GLG tests from all branches.: [evidence span withheld — anchor status: external_doc]
- [readme] Because the subsampling and zero-removal stages involve pseudo-random sample removals, SINGE generates a random seed using input hyperparameters, including the date input.: [evidence span withheld — anchor status: external_doc]
