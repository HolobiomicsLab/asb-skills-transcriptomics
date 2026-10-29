---
name: pseudotime-perturbation-sensitivity-analysis
description: Use when ordered single-cell expression data are available and you need
  to test whether inferred or assigned pseudotemporal values (rather than the expression
  matrix itself) drive the quality of pseudotime-based network inference.
when_to_use_negative:
- The data are unordered (no pseudotemporal information or trajectory available) —
  SINGE requires ordered single-cell expression data and pseudotime values
- The goal is to assess sensitivity to hyperparameters, subsampling, or zero-handling
  rather than to the pseudotime assignments themselves
- Only a single trajectory/pseudotime source exists and perturbation of assignments
  is out of scope — a plain SINGE run (SINGE.m or SINGE.sh) suffices without the variant
  comparison
edam_operation: http://edamontology.org/operation_2428
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: SINGE
  role: Primary GRN inference method run as the two-stage workflow (SINGE GLG followed
    by SINGE Aggregate with modified Borda aggregation) on each pseudotime variant
  repo: https://github.com/gitter-lab/SINGE
- name: glmnet_matlab
  role: Dependency required by SINGE GLG for the generalized lasso Granger regression
  repo: http://web.stanford.edu/~hastie/glmnet_matlab/download.html
- name: dynverse
  role: Suite used to infer reference trajectories and generate the informative pseudotime
    values used as the baseline variant
- name: dynwrap
  role: Provides add_pseudotime to assign pseudotemporal values post hoc when the
    selected trajectory method does not provide them
  repo: https://rdrr.io/github/dynverse/dynwrap/man/add_pseudotime.html
- name: SINCERITIES
  role: Comparison pseudotime-based GRN inference method run on each pseudotime variant
- name: SCODE
  role: Comparison pseudotime-based GRN inference method run on each pseudotime variant
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.celrep.2022.110333
    title: Network inference with Granger causality ensembles on single-cell transcriptomics
  source_content_hash: sha256:224ec46be87359c4d04b8b47f2ba987652e91318502e507b8c94f87d2ba25299
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gitter-lab/SINGE
  tools:
  - SCODE
  - SINCERITIES
  - SINGE
  - dynverse
  - dynwrap
  - glmnet_matlab
  edam_operation: http://edamontology.org/operation_2428
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6923076923076923
n_claims: 13
---

# pseudotime-perturbation-sensitivity-analysis

## Summary

A sensitivity-analysis skill that quantifies how the quality of pseudotime assignments affects GRN inference by comparing method performance (e.g., SINGE, SINCERITIES, SCODE AUROC/AUPRC) on informative trajectory-ordered pseudotimes versus perturbed variants such as shuffled (uninformative) pseudotimes, with expression data held fixed.

## When to use

Use when ordered single-cell expression data are available and you need to test whether inferred or assigned pseudotemporal values (rather than the expression matrix itself) drive the quality of pseudotime-based network inference.

## When NOT to use

- The data are unordered (no pseudotemporal information or trajectory available) — SINGE requires ordered single-cell expression data and pseudotime values
- The goal is to assess sensitivity to hyperparameters, subsampling, or zero-handling rather than to the pseudotime assignments themselves
- Only a single trajectory/pseudotime source exists and perturbation of assignments is out of scope — a plain SINGE run (SINGE.m or SINGE.sh) suffices without the variant comparison

## Inputs

- Expression .mat file with sparse matrix X, pseudotime array ptime, optional regulator indices regix, and optional branching matrix branches (e.g., data1/X_SCODE_data.mat, data_bifurcated/X_data_bifurcated.mat)
- Gene list file corresponding to rows of X (e.g., data1/gene_list.mat)
- Reference pseudotimes inferred via dynverse trajectory inference or assigned with dynwrap add_pseudotime
- SINGE hyperparameter configuration file (default_hyperparameters.txt)
- Candidate regulator list (optional)

## Outputs

- Ranked edge lists per variant (SINGE_Ranked_Edge_List.txt) and gene influence rankings (SINGE_Gene_Influence.txt)
- Comparison network predictions from SINCERITIES and SCODE for each variant
- Compiled AUROC/AUPRC table (overall and per regulator) across all method-variant pairs, e.g., pseudotime_sensitivity_results.csv

## How to apply

First generate reference pseudotimes by inferring trajectories with the dynverse suite; when the selected trajectory method does not output pseudotimes, assign them post hoc with the dynwrap add_pseudotime function. Then construct pseudotime variants while keeping the expression matrix X fixed, including an uninformative variant created by shuffling the pseudotime values across cells and alternative trajectory methods such as PAGA Tree. Run the SINGE workflow (SINGE GLG followed by SINGE Aggregate with modified Borda aggregation) on each variant using default_hyperparameters.txt as the ensemble configuration, and run SINCERITIES and SCODE on each variant for comparison. Evaluate AUROC/AUPRC overall and per regulator for every method-variant pair and compile the results into a single table (pseudotime_sensitivity_results.csv). The expected validation signal is degradation of inference performance — notably SINGE's — under uninformative pseudotimes relative to informative trajectory orderings, confirming that pseudotime quality, not just expression data, governs reconstruction quality.

## Related tools

- **SINGE** (Primary GRN inference method run as the two-stage workflow (SINGE GLG followed by SINGE Aggregate with modified Borda aggregation) on each pseudotime variant) — https://github.com/gitter-lab/SINGE
- **glmnet_matlab** (Dependency required by SINGE GLG for the generalized lasso Granger regression) — http://web.stanford.edu/~hastie/glmnet_matlab/download.html
- **dynverse** (Suite used to infer reference trajectories and generate the informative pseudotime values used as the baseline variant)
- **dynwrap** (Provides add_pseudotime to assign pseudotemporal values post hoc when the selected trajectory method does not provide them) — https://rdrr.io/github/dynverse/dynwrap/man/add_pseudotime.html
- **SINCERITIES** (Comparison pseudotime-based GRN inference method run on each pseudotime variant)
- **SCODE** (Comparison pseudotime-based GRN inference method run on each pseudotime variant)

## Examples

```
bash SINGE.sh PATH_TO_RUNTIME standalone shuffled_variant/X_SCODE_data.mat data1/gene_list.mat Output_variant default_hyperparameters.txt
```

## Evaluation signals

- The shuffled (uninformative) pseudotime variant must use the exact same expression matrix X as the reference variant — only ptime changes
- Compiled AUROC/AUPRC per variant should reproduce the reported pattern: performance degradation for pseudotime-based methods (especially SINGE) under uninformative pseudotimes relative to informative trajectory orderings
- Per-regulator AUROC/AUPRC values should be computed in addition to overall metrics, since SINGE can perform poorly for individual regulators even when overall metrics look acceptable
- SINGE results must be reproducible: rerunning with the same hyperparameter file, subsampling settings, and date input should regenerate identical ranked edge lists
- Each variant run must use the same ensemble configuration (default_hyperparameters.txt) so that differences are attributable to pseudotime quality alone

## Limitations

- Uninformative pseudotimes are a caveat documented by the authors: 'poor performance for individual regulators and uninformative pseudotimes' — shuffled assignments can be detrimental to network inference performance
- For branching trajectories, SINGE combines results across all branches in v0.5.0; branch-specific analysis requires separately storing per-branch GLG results and calling SINGE Aggregate individually
- glmnet_matlab may encounter memory segmentation violations with certain settings, which can block variant runs
- The master branch of the SINGE repository may be unstable; a versioned release should be used for stable data analysis
- Windows users cannot use compiled executables and may need Docker if a compatible MATLAB environment is unavailable

## Evidence

- [abstract] Caveats about poor performance for individual regulators and uninformative pseudotimes: "However, a more detailed examination reveals caveats about poor performance for individual regulators and uninformative pseudotimes."
- [methods] add_pseudotime for post hoc pseudotime assignment: "they can be assigned after the fact using the [`add_pseudotime`](https://rdrr.io/github/dynverse/dynwrap/man/add_pseudotime.html) function."
- [methods] dynverse usefulness for trajectory inference: "We found the `dynverse` (http://dynverse.org/) package to be extremely useful for this purpose."
- [methods] SINGE two-stage workflow naming: "the complete SINGE workflow, which includes the SINGE GLG and SINGE Aggregate modes."
- [readme] Standalone invocation example: [evidence span withheld — anchor status: external_doc]
