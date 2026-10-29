---
name: hyperparameter-sensitivity-analysis
description: Use when deciding how many SINGE ensemble members (replicates/hyperparameter
  combinations) are necessary, or when checking whether simplifying the hyperparameter
  grid (e.g., 2–5 replicates instead of 10) degrades precision-recall performance
  on pseudotemporal single-cell data.
when_to_use_negative:
- Data lack pseudotemporal ordering — SINGE requires ordered single-cell expression
  and kernel-based Granger regression on pseudotimes
- The goal is tuning hyperparameters like lambda, kernel-width, or dT rather than
  ensemble size — those require their own sweeps
- No reference regulatory network is available to compute average precision or average
  early precision
edam_operation: http://edamontology.org/operation_0279
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: SINGE
  role: Runs the GLG Granger causality regression per subsampled replicate and the
    Aggregate mode (modified Borda ranking) to produce ranked edge lists per replicate
    count
  repo: https://github.com/gitter-lab/SINGE
- name: glmnet_matlab
  role: Dependency for the GLG sparse regression within SINGE (unzipped in the SINGE
    root or code subdirectory)
- name: MATLAB R2018a
  role: MATLAB environment in which the replicate-sweep SINGE GLG/Aggregate runs are
    executed
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.celrep.2022.110333
    title: Network inference with Granger causality ensembles on single-cell transcriptomics
  source_content_hash: sha256:d6b357fbc40158a7685a2b137058927f4a6208c4b32fdefea26d356b6ed9a100
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
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# hyperparameter-sensitivity-analysis

## Summary

Systematically varies SINGE GLG hyperparameter combinations (e.g., number of subsampled replicates with prob-remove-samples = 0.2) and measures how average precision (AUPR) and average early precision of the inferred gene regulatory network change, to determine the minimum ensemble size needed for stable network inference.

## When to use

Use when deciding how many SINGE ensemble members (replicates/hyperparameter combinations) are necessary, or when checking whether simplifying the hyperparameter grid (e.g., 2–5 replicates instead of 10) degrades precision-recall performance on pseudotemporal single-cell data.

## When NOT to use

- Data lack pseudotemporal ordering — SINGE requires ordered single-cell expression and kernel-based Granger regression on pseudotimes
- The goal is tuning hyperparameters like lambda, kernel-width, or dT rather than ensemble size — those require their own sweeps
- No reference regulatory network is available to compute average precision or average early precision

## Inputs

- Pseudotime-ordered single-cell expression matrix (.mat file with sparse matrix X and ptime array, e.g., data1/X_SCODE_data.mat)
- Gene list file (e.g., data1/gene_list.mat)
- SINGE GLG hyperparameter file (generated via scripts/generate_hyperparameters.sh from scripts/lambda.txt, kernel.txt, time.txt, probzeroremoval.txt, probremovesample.txt)
- Reference regulatory network for precision-recall scoring

## Outputs

- Per-replicate-count SINGE_Ranked_Edge_List.txt and SINGE_Gene_Influence.txt
- Table of average precision (AUPR) and average early precision versus replicate count
- Replicate-sweep validation table showing precision-recall performance of 2–5 replicates relative to 10 replicates

## How to apply

Generate hyperparameter files with scripts/generate_hyperparameters.sh that fix sparsity-related settings (lambda, prob-remove-samples = 0.2, prob-zero-removal) and vary only the replicate count (e.g., two, five, ten). For each replicate count, run SINGE GLG (kernel-based Granger causality regression) on each subsampled replicate of the pseudotime-ordered expression matrix, then run SINGE Aggregate (modified Borda ranking, stability-selection-style ensembling) to get one ranked regulator-target list per count. Score each aggregated network against the reference regulatory network to compute average precision (AUPR) and average early precision. Tabulate both metrics versus replicate count; the retinoic acid dataset shows only modest improvement beyond five replicates, so two-five replicates incur only moderate degradation. Because subsampling is pseudo-random, reproduce sweeps by providing the same inputs and date from a previous experiment.

## Related tools

- **SINGE** (Runs the GLG Granger causality regression per subsampled replicate and the Aggregate mode (modified Borda ranking) to produce ranked edge lists per replicate count) — https://github.com/gitter-lab/SINGE
- **glmnet_matlab** (Dependency for the GLG sparse regression within SINGE (unzipped in the SINGE root or code subdirectory))
- **MATLAB R2018a** (MATLAB environment in which the replicate-sweep SINGE GLG/Aggregate runs are executed)

## Examples

```
bash SINGE.sh PATH_TO_RUNTIME standalone data1/X_SCODE_data.mat data1/gene_list.mat Output default_hyperparameters.txt
```

## Evaluation signals

- Replicate-sweep table shows precision-recall performance with 2–5 replicates approaches that of 10 replicates, with only moderate degradation
- Average precision and average early precision show no substantial gain beyond five subsampled replicates per hyperparameter combination
- Each hyperparameter set has a unique --ID combining the hyperparameter set and replicate index
- Re-running with the same inputs and --date reproduces the same networks despite pseudo-random subsampling

## Limitations

- Subsampling and zero-removal stages involve pseudo-random sample removals; results are only reproducible if the same inputs and date hyperparameter are supplied
- glmnet_matlab can encounter memory segmentation violations with certain settings
- The master branch of the SINGE repository may be unstable; use a versioned release for stable data analysis

## Evidence

- [methods] stability-selection ensembling: "SINGE's ensembling and modified Borda aggregation is motivated by stability selection from these networks."
- [readme] replicate hyperparameter: [evidence span withheld — anchor status: not_found]
- [readme] hyperparameter generation script: [evidence span withheld — anchor status: external_doc]
- [readme] reproducibility via date: [evidence span withheld — anchor status: external_doc]
