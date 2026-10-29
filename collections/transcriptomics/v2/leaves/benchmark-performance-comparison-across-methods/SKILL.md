---
name: benchmark-performance-comparison-across-methods
description: Use when comparing GRN inference methods (e.g., SINGE, SINCERITIES, SCODE)
  across pseudotime/trajectory variants on ordered single-cell expression data, scored
  with AUROC/AUPRC overall and per regulator.
when_to_use_negative:
- The task is to run a single GRN inference method once on a well-ordered pseudotime
  without any comparative evaluation — no benchmark comparison is needed
- Input expression data are unordered (no pseudotime or trajectory available), since
  all three compared methods (SINGE, SINCERITIES, SCODE) require pseudotemporal ordering
- Only a single-cell count matrix with no ground-truth or reference network and no
  per-regulator labels, making AUROC/AUPRC evaluation infeasible
edam_topics:
- http://edamontology.org/topic_0203
tools:
- name: SINGE
  role: Primary GRN inference method benchmarked across pseudotime variants via the
    SINGE GLG and SINGE Aggregate workflow with modified Borda aggregation
  repo: https://github.com/gitter-lab/SINGE
- name: glmnet_matlab
  role: Dependency required by SINGE for the underlying sparse regression (GLG) tests
    during benchmarking
  repo: http://web.stanford.edu/~hastie/glmnet_matlab/download.html
- name: dynverse
  role: Suite used to infer reference trajectories and generate pseudotimes for the
    benchmark's informative-pseudotime condition
  repo: http://dynverse.org/
- name: dynwrap
  role: Used via the add_pseudotime function to assign pseudotemporal values post
    hoc when the trajectory method does not provide them
  repo: https://rdrr.io/github/dynverse/dynwrap/man/add_pseudotime.html
provenance:
  source_task_ids:
  - task_003
  source_papers:
  - doi: 10.1016/j.celrep.2022.110333
    title: Network inference with Granger causality ensembles on single-cell transcriptomics
  source_content_hash: sha256:71dac4a429149e806f52ebad5fa08c1669ebc55a274cd39953c06ed1c38e0730
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/gitter-lab/SINGE
  tools:
  - SINGE
  - dynverse
  - dynwrap
  - glmnet_matlab
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: critique
yes_rate: 0.5714285714285714
n_claims: 7
---

# benchmark-performance-comparison-across-methods

## Summary

Systematic benchmarking of pseudotime-based GRN inference methods (SINGE, SINCERITIES, SCODE) by holding the expression data fixed, varying the pseudotime assignments, and comparing network reconstruction performance with AUROC and AUPRC metrics, both overall and per regulator.

## When to use

Use when comparing GRN inference methods (e.g., SINGE, SINCERITIES, SCODE) across pseudotime/trajectory variants on ordered single-cell expression data, scored with AUROC/AUPRC overall and per regulator.

## When NOT to use

- The task is to run a single GRN inference method once on a well-ordered pseudotime without any comparative evaluation — no benchmark comparison is needed
- Input expression data are unordered (no pseudotime or trajectory available), since all three compared methods (SINGE, SINCERITIES, SCODE) require pseudotemporal ordering
- Only a single-cell count matrix with no ground-truth or reference network and no per-regulator labels, making AUROC/AUPRC evaluation infeasible

## Inputs

- Ordered single-cell expression data as .mat files with sparse matrix X, pseudotime array ptime, optional regulator indices regix, and optional branches matrix (e.g., data1/X_SCODE_data.mat, data_bifurcated/X_data_bifurcated.mat)
- Gene list file (e.g., data1/gene_list.mat) with gene names corresponding to rows of X
- GLG hyperparameter file (e.g., default_hyperparameters.txt) with lambda, dT, num-lags, kernel-width, replicate, family, prob-zero-removal, prob-remove-samples, date
- Reference pseudotimes from dynverse trajectory inference
- Pseudotime variants, including uninformative (shuffled) pseudotimes and alternative trajectory methods such as PAGA Tree

## Outputs

- Ranked edge lists per method-variant (e.g., SINGE_Ranked_Edge_List.txt)
- Gene influence rankings (e.g., SINGE_Gene_Influence.txt)
- Compiled performance table (e.g., pseudotime_sensitivity_results.csv) with AUROC/AUPRC overall and per regulator for each method-pseudotime-variant pair

## How to apply

Generate reference pseudotimes by inferring trajectories with the dynverse suite, assigning pseudotemporal values post hoc with dynwrap's add_pseudotime function when the trajectory method does not provide them. Create pseudotime variants (e.g., uninformative/shuffled assignments, alternative methods such as PAGA Tree) while keeping the expression data fixed, so that any performance change is attributable to pseudotime quality. Run the full SINGE workflow — SINGE GLG followed by SINGE Aggregate with modified Borda aggregation — using default_hyperparameters.txt as the ensemble configuration, and run SINCERITIES and SCODE on each variant to obtain comparison predictions. Evaluate each method-variant pair with AUROC/AUPRC overall and per regulator, then compile results (e.g., into pseudotime_sensitivity_results.csv) and check whether the pattern is reproduced, such as SINGE degradation under uninformative pseudotimes relative to informative trajectory orderings.

## Related tools

- **SINGE** (Primary GRN inference method benchmarked across pseudotime variants via the SINGE GLG and SINGE Aggregate workflow with modified Borda aggregation) — https://github.com/gitter-lab/SINGE
- **glmnet_matlab** (Dependency required by SINGE for the underlying sparse regression (GLG) tests during benchmarking) — http://web.stanford.edu/~hastie/glmnet_matlab/download.html
- **dynverse** (Suite used to infer reference trajectories and generate pseudotimes for the benchmark's informative-pseudotime condition) — http://dynverse.org/
- **dynwrap** (Used via the add_pseudotime function to assign pseudotemporal values post hoc when the trajectory method does not provide them) — https://rdrr.io/github/dynverse/dynwrap/man/add_pseudotime.html

## Examples

```
bash SINGE.sh PATH_TO_RUNTIME standalone data1/X_SCODE_data.mat data1/gene_list.mat Output default_hyperparameters.txt
```

## Evaluation signals

- The compiled results table (pseudotime_sensitivity_results.csv) reproduces the reported pattern: SINGE performance degrades under uninformative (shuffled) pseudotimes relative to informative trajectory orderings
- Each method-variant pair yields AUROC and AUPRC computed both overall and per regulator, enabling detection of poor performance for individual regulators
- The complete SINGE workflow (SINGE GLG followed by SINGE Aggregate) is executed for every pseudotime variant with the same default_hyperparameters.txt ensemble configuration, so variants are directly comparable
- Expression data are identical across variants — only the ptime values differ — confirming that performance differences are attributable to pseudotime quality
- SINGE outputs match the expected format (SINGE_Ranked_Edge_List.txt and SINGE_Gene_Influence.txt) and, where applicable, reproduce reference output within test thresholds

## Limitations

- Assigning uninformative pseudotime values to ordered cells can be detrimental to network inference performance, degrading GRN reconstruction
- Poor performance can occur for individual regulators even when overall AUROC/AUPRC look acceptable
- The subsampling and zero-removal stages are pseudo-random: results are reproducible only by providing the same inputs and date hyperparameter, and the master branch may be unstable (use versioned releases)
- SINGE's glmnet_matlab dependency can encounter memory segmentation violations with certain settings, and there are no compiled Windows executables (Docker is recommended)

## Evidence

- [abstract] Caveats about poor performance for individual regulators and uninformative pseudotimes: "However, a more detailed examination reveals caveats about poor performance for individual regulators and uninformative pseudotimes."
- [methods] SINGE workflow modes used for benchmarking: "the complete SINGE workflow, which includes the SINGE GLG and SINGE Aggregate modes."
- [methods] Post hoc pseudotime assignment with dynwrap: "they can be assigned after the fact using the [`add_pseudotime`](https://rdrr.io/github/dynverse/dynwrap/man/add_pseudotime.html) function."
- [readme] Supplemental analyses repository supporting the manuscript results: [evidence span withheld — anchor status: external_doc]
- [readme] Running SINGE stages separately for the benchmark: [evidence span withheld — anchor status: not_found]
