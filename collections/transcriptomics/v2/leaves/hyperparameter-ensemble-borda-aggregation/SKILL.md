---
name: hyperparameter-ensemble-borda-aggregation
description: Use when reconstructing gene regulatory networks from pseudotemporal
  single-cell expression data (.mat files with sparse X, ptime, optional regix/branches)
  and you need SINGE's GLG-then-Aggregate two-stage ensemble workflow with ranked
  edge output.
when_to_use_negative:
- The data lacks an ordering/pseudotime (no ptime array) — SINGE requires pseudotemporally
  ordered single-cell expression data
- A single fixed-hyperparameter Granger regression is sufficient and no ensemble/ranked
  network is needed — the ensemble and Borda aggregation would add unnecessary compute
- For branching trajectories where branch-specific networks are required without combining
  results, run Aggregate per-branch on separately stored GLG results rather than letting
  v0.5.0 combine all branches
edam_operation: null
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_0602
tools:
- name: SINGE
  role: Runs the GLG tests across hyperparameters/replicates and the Aggregate stage
    with modified Borda aggregation to produce the ranked edge list
  repo: https://github.com/gitter-lab/SINGE
- name: MATLAB
  role: Environment (tested R2014b, R2018a, R2020a) for running SINGE.m standalone
    or SINGE_GLG_Test.m plus SINGE_Aggregate.m separately
- name: glmnet_matlab
  role: Dependency used by the GLG regression stage; unzip into the root or code subdirectory
- name: Docker
  role: Alternative execution mode via agitter/singe images and the SINGE.sh wrapper
    for standalone, GLG, or Aggregate modes
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1016/j.celrep.2022.110333
    title: Network inference with Granger causality ensembles on single-cell transcriptomics
  source_content_hash: sha256:3135b437a57a1855219c5006f7e506ca150f0600185f9fc21682c5f383b6948e
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gitter-lab/SINGE
  tools:
  - Docker
  - MATLAB
  - SINGE
  - glmnet_matlab
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_0602
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# hyperparameter-ensemble-borda-aggregation

## Summary

SINGE's full pipeline runs multiple Generalized Lasso Granger (GLG) tests across hyperparameter combinations and subsampled replicates, then aggregates the ensemble of partial directed networks with a modified Borda method (motivated by stability selection) into a single ranked regulator-target edge list.

## When to use

Use when reconstructing gene regulatory networks from pseudotemporal single-cell expression data (.mat files with sparse X, ptime, optional regix/branches) and you need SINGE's GLG-then-Aggregate two-stage ensemble workflow with ranked edge output.

## When NOT to use

- The data lacks an ordering/pseudotime (no ptime array) — SINGE requires pseudotemporally ordered single-cell expression data
- A single fixed-hyperparameter Granger regression is sufficient and no ensemble/ranked network is needed — the ensemble and Borda aggregation would add unnecessary compute
- For branching trajectories where branch-specific networks are required without combining results, run Aggregate per-branch on separately stored GLG results rather than letting v0.5.0 combine all branches

## Inputs

- MAT-file with ordered single-cell expression data: sparse matrix X, pseudotime array ptime, optional regulator index array regix, optional branching matrix branches (e.g., data1/X_SCODE_data.mat, data_bifurcated/X_data_bifurcated.mat)
- MAT-file with gene list corresponding to rows of X (e.g., data1/gene_list.mat)
- Hyperparameter file with GLG hyperparameter combinations (e.g., default_hyperparameters.txt, generable via scripts/generate_hyperparameters.sh)
- Output directory for individual GLG test results (outdir)

## Outputs

- SINGE_Ranked_Edge_List.txt - ranked list of regulator-target edges by SINGE score
- SINGE_Gene_Influence.txt - genes ranked by SINGE influence
- Per-GLG-test partial network outputs stored in the outdir before aggregation

## How to apply

First run SINGE in GLG mode over every hyperparameter combination in the hyperparameter file (lambda sparsity, dT time resolution, num-lags, kernel-width, family, prob-zero-removal, prob-remove-samples=0.2), generating multiple subsampled replicates per combination with unique --ID and --replicate indices; avoid lambda=0 since it yields a non-sparse (complete) solution. For large count-based datasets, log-transform with X1 = log(1+X) and set --family gaussian, because glmnet_matlab encounters memory segmentation violations with the alternative setting. Then run SINGE in Aggregate mode, which applies the modified Borda aggregation, motivated by stability selection, across the ensemble of partial networks to compile SINGE_Ranked_Edge_List.txt and SINGE_Gene_Influence.txt. GLG and Aggregate can be run serially (standalone MATLAB, SINGE.m) or split across machines (compiled executables/Docker) and recombined afterward; reproducibility depends on supplying the same hyperparameters including the --date input, which seeds pseudo-random subsampling and zero-removal.

## Related tools

- **SINGE** (Runs the GLG tests across hyperparameters/replicates and the Aggregate stage with modified Borda aggregation to produce the ranked edge list) — https://github.com/gitter-lab/SINGE
- **MATLAB** (Environment (tested R2014b, R2018a, R2020a) for running SINGE.m standalone or SINGE_GLG_Test.m plus SINGE_Aggregate.m separately)
- **glmnet_matlab** (Dependency used by the GLG regression stage; unzip into the root or code subdirectory)
- **Docker** (Alternative execution mode via agitter/singe images and the SINGE.sh wrapper for standalone, GLG, or Aggregate modes)

## Examples

```
docker run -v $(pwd):/SINGE -w /SINGE agitter/singe:0.5.1 standalone data1/X_SCODE_data.mat data1/gene_list.mat Output default_hyperparameters.txt
```

## Evaluation signals

- Output network is a ranked, non-complete edge list — confirm lambda=0 (non-sparse solution) was avoided among the hyperparameters
- SINGE_Ranked_Edge_List.txt and SINGE_Gene_Influence.txt exist and are correctly formatted rankings after Aggregate mode completes
- Re-running with identical inputs and --date reproduces the same results, since the random seed derives from the hyperparameters including date
- tests/compare_example_output.sh matches generated output against tests/reference/latest adjacency matrices (macOS uses a more permissive threshold for OS-specific differences)
- Hyperparameter IDs are unique for each hyperparameter set and replicate index, and all GLG tests have terminated before aggregation

## Limitations

- glmnet_matlab encounters memory segmentation violations with certain settings on large count-based datasets; use log-transform X1 = log(1+X) with --family gaussian instead
- lambda = 0 results in a non-sparse (complete graph) solution and must be avoided
- The master branch may be unstable; use a versioned release for stable data analysis
- Compiled executables are not available for Windows; use Docker there
- Subsampling and zero-removal use pseudo-random sample removals, so results are only reproducible when the same inputs and date are provided
- On branching trajectories (v0.5.0), ranked edge list and gene influence are combined across all branches by default; branch-specific inference requires separate per-branch aggregation

## Evidence

- [readme] The full SINGE pipeline runs multiple Generalized Lasso Granger (GLG) tests to infer different directed networks for different hyperparameters and subsamples of the data.: [evidence span withheld — anchor status: external_doc]
- [methods] Ensembling and modified Borda aggregation motivated by stability selection: "SINGE's ensembling and modified Borda aggregation is motivated by stability selection from these networks."
- [abstract] It aggregates predictions from an ensemble of regression analyses to compile a ranked list of candidate interactions between transcriptional regulators and target genes.: "It aggregates predictions from an ensemble of regression analyses to compile a ranked list of candidate interactions between transcriptional regulators and target genes."
- [other] Avoid lambda=0 that yields a complete graph; avoid glmnet_matlab memory segmentation violations: [evidence span withheld — anchor status: not_found]
- [readme] SINGE generates a random seed using input hyperparameters, including the date input: [evidence span withheld — anchor status: external_doc]
