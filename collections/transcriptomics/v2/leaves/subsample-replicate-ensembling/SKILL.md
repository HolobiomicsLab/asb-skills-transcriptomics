---
name: subsample-replicate-ensembling
description: 'Use when tuning the cost/accuracy trade-off of a SINGE run: you must
  decide how many subsampled replicates per hyperparameter combination are needed
  before precision-recall performance plateaus on pseudotemporal single-cell expression
  data.'
when_to_use_negative:
- The goal is a single best network rather than characterizing replicate-count sensitivity
  — just run the full SINGE standalone pipeline once
- The dataset is so small that subsampled replicates remove too many cells; sample
  removal probability 0.2 assumes enough cells per subsample for GLG regression
- The algorithm is not SINGE — this skill is specific to SINGE's GLG/Aggregate ensemble
  and its modified Borda aggregation
edam_operation: http://edamontology.org/operation_3439
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: SINGE
  role: Runs the GLG kernel-based Granger causality tests on each subsampled replicate
    and the Aggregate step with modified Borda ranking to produce the ranked edge
    list
  repo: https://github.com/gitter-lab/SINGE
- name: glmnet_matlab
  role: Dependency required to run SINGE GLG regression through MATLAB
  repo: http://web.stanford.edu/~hastie/glmnet_matlab/download.html
- name: MATLAB R2018a
  role: Tested MATLAB environment for running SINGE GLG Test and SINGE Aggregate serially
    (standalone mode) or via compiled executables
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.celrep.2022.110333
    title: Network inference with Granger causality ensembles on single-cell transcriptomics
  source_content_hash: sha256:c6f5da84c7166300116fe3c3d0d1b2283f51cd1ad7c717fc6ee76dfdfb459e90
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gitter-lab/SINGE
  tools:
  - MATLAB R2018a
  - SINGE
  - glmnet_matlab
  edam_operation: http://edamontology.org/operation_3439
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.8
n_claims: 10
---

# subsample-replicate-ensembling

## Summary

Sensitivity-analysis skill for the SINGE ensemble: vary the number of subsampled replicates per hyperparameter combination (with prob-remove-samples = 0.2) and measure how average precision and average early precision of the inferred gene regulatory network change, since subsampling drives SINGE's stability-selection-style aggregation.

## When to use

Use when tuning the cost/accuracy trade-off of a SINGE run: you must decide how many subsampled replicates per hyperparameter combination are needed before precision-recall performance plateaus on pseudotemporal single-cell expression data.

## When NOT to use

- The goal is a single best network rather than characterizing replicate-count sensitivity — just run the full SINGE standalone pipeline once
- The dataset is so small that subsampled replicates remove too many cells; sample removal probability 0.2 assumes enough cells per subsample for GLG regression
- The algorithm is not SINGE — this skill is specific to SINGE's GLG/Aggregate ensemble and its modified Borda aggregation

## Inputs

- Pseudotime-ordered single-cell expression matrix (sparse matrix X with ptime array in a .mat file, e.g. data1/X_SCODE_data.mat)
- Gene list file corresponding to rows of X (e.g. data1/gene_list.mat)
- Reference regulatory network for scoring average precision and average early precision
- Hyperparameter files generated via scripts/generate_hyperparameters.sh with prob-remove-samples = 0.2 and replicate counts of 2, 5, and 10

## Outputs

- SINGE_Ranked_Edge_List.txt per replicate count (ranked regulator-target edges from SINGE Aggregate)
- SINGE_Gene_Influence.txt per replicate count
- Replicate-sweep table of average precision (AUPR) and average early precision versus replicate count (2, 5, 10)

## How to apply

Generate SINGE hyperparameter files with scripts/generate_hyperparameters.sh that fix lambda and prob-zero-removal, set prob-remove-samples to 0.2, and vary only the replicate count among two, five, and ten. For each replicate count, run SINGE GLG (kernel-based Granger causality regression) on each subsample of the pseudotime-ordered expression matrix, then run SINGE Aggregate with the modified Borda ranking to merge the GLG outputs into one ranked regulator-target list. Score each aggregated network against the reference regulatory network to compute average precision (AUPR) and average early precision, and tabulate these against replicate count. The rationale is that SINGE's ensembling is motivated by stability selection, so the replicate count controls ensemble diversity; the expected result is only modest precision-recall improvement beyond five replicates, meaning two-five replicates are acceptable for larger sweeps.

## Related tools

- **SINGE** (Runs the GLG kernel-based Granger causality tests on each subsampled replicate and the Aggregate step with modified Borda ranking to produce the ranked edge list) — https://github.com/gitter-lab/SINGE
- **glmnet_matlab** (Dependency required to run SINGE GLG regression through MATLAB) — http://web.stanford.edu/~hastie/glmnet_matlab/download.html
- **MATLAB R2018a** (Tested MATLAB environment for running SINGE GLG Test and SINGE Aggregate serially (standalone mode) or via compiled executables)

## Examples

```
bash SINGE.sh PATH_TO_RUNTIME standalone data1/X_SCODE_data.mat data1/gene_list.mat Output hyperparameters_rep10.txt
```

## Evaluation signals

- The replicate-sweep table shows two-five replicates approach the precision-recall performance of ten replicates with only moderate degradation
- No substantial gain in average precision or average early precision is observed beyond five replicates
- All GLG hyperparameter sets other than replicate count (lambda, prob-zero-removal, prob-remove-samples = 0.2) are held fixed across the sweep
- Each aggregated output is a valid SINGE_Ranked_Edge_List.txt produced by SINGE Aggregate over exactly the GLG outputs for that replicate count
- Reproduced runs match previous results when the same hyperparameters (including the --date seed input) are supplied

## Limitations

- Subsampling and zero-removal are pseudo-random, so results are reproducible only when the same hyperparameters, including the date input used to generate the random seed, are provided
- The master branch of the SINGE repository may be unstable; a versioned release should be used for stable data analysis
- The glmnet_matlab package can encounter memory segmentation violations with certain settings, which may limit feasible replicate counts
- The finding (modest gains beyond five replicates) is grounded in the retinoic acid dataset analysis and may not transfer unchanged to other datasets

## Evidence

- [readme] prob-remove-samples default: [evidence span withheld — anchor status: not_found]
- [abstract] ensemble aggregation: "It aggregates predictions from an ensemble of regression analyses to compile a ranked list of candidate interactions between transcriptional regulators and target genes."
- [methods] stability selection motivation: "SINGE's ensembling and modified Borda aggregation is motivated by stability selection from these networks."
- [readme] reproducibility via date seed: [evidence span withheld — anchor status: external_doc]
- [readme] hyperparameter generation scripts: [evidence span withheld — anchor status: external_doc]
