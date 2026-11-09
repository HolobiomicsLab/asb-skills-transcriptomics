---
name: parameter-sensitivity-sweep-analysis
description: Use when validating a clustering or similarity-learning pipeline whose
  defaults (e.g. penalty ratio rho/lambda) need empirical justification, by sweeping
  parameters on benchmark datasets and plotting performance stability curves.
when_to_use_negative:
- The method has already been validated for parameter sensitivity and the default
  is known — re-running the sweep adds no information
- The task is benchmarking against competing methods rather than tuning/justifying
  the method's own penalty parameters
- Datasets lack ground-truth cell type labels, so NMI and ARI cannot be computed
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: MATLAB
  role: Runs the SSRE implementation for each rho/lambda grid value on the benchmark
    datasets
  repo: https://github.com/CSUBioGroup/SSRE
- name: Python
  role: Alternative implementation of SSRE used to execute the parameter sweep and
    compute NMI/ARI
  repo: https://github.com/CSUBioGroup/SSRE
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.gpb.2020.09.004
    title: 'SSRE: Cell Type Detection Based on Sparse Subspace Representation and
      Similarity Enhancement'
  source_content_hash: sha256:8155afeac31d9c9027c8aef1ccf72b184c06278daca5a48e65f095ce42179183
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSUBioGroup/SSRE
  tools:
  - MATLAB
  - Python
  edam_operation: null
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.625
n_claims: 8
---

# parameter-sensitivity-sweep-analysis

## Summary

Systematically varies a method's key penalty parameters over a grid (e.g. SSRE's rho/lambda ratio over {2, 4, ..., 30}) on real scRNA-seq benchmarks, evaluates clustering quality (NMI, ARI) at each point, and identifies the stable interval supporting a default setting.

## When to use

Use when validating a clustering or similarity-learning pipeline whose defaults (e.g. penalty ratio rho/lambda) need empirical justification, by sweeping parameters on benchmark datasets and plotting performance stability curves.

## When NOT to use

- The method has already been validated for parameter sensitivity and the default is known — re-running the sweep adds no information
- The task is benchmarking against competing methods rather than tuning/justifying the method's own penalty parameters
- Datasets lack ground-truth cell type labels, so NMI and ARI cannot be computed

## Inputs

- Ten real scRNA-seq benchmark datasets with ground-truth cell type labels (cell-by-gene expression matrix plus labels, e.g. .mat files with in_X and true_labs)
- SSRE MATLAB or Python implementation from https://github.com/CSUBioGroup/SSRE
- Parameter grid of rho/lambda values {2, 4, 6, ..., 30}
- Known cell type labels for NMI/ARI computation

## Outputs

- rho_lambda_sensitivity.csv — full sweep table of NMI/ARI per (dataset, rho/lambda) pair
- Stability curves of NMI and ARI versus rho/lambda (reproducing Figure 2 and Figure S1)
- Identified stable interval (6–20) and confirmed default setting (rho/lambda = 10)

## How to apply

Obtain the method implementation and the real benchmark data (for SSRE, the MATLAB/Python code from CSUBioGroup/SSRE and the ten real scRNA-seq datasets). For each dataset, run the method with the target parameter (rho/lambda) set to each grid value while keeping all other parameters at their defaults. Score every (dataset, parameter-value) pair by comparing predicted cluster assignments to known cell type labels using NMI and ARI. Aggregate scores across datasets and plot NMI and ARI versus the parameter value, mirroring the paper's Figure 2 and Figure S1. Verify that performance is flat within a stable interval (for SSRE, rho/lambda between 6 and 20) and that the default (rho/lambda = 10) falls in it, then export the full sweep table (e.g. rho_lambda_sensitivity.csv).

## Related tools

- **MATLAB** (Runs the SSRE implementation for each rho/lambda grid value on the benchmark datasets) — https://github.com/CSUBioGroup/SSRE
- **Python** (Alternative implementation of SSRE used to execute the parameter sweep and compute NMI/ARI) — https://github.com/CSUBioGroup/SSRE

## Evaluation signals

- Reproduced NMI and ARI curves match the paper's Figure 2 and Figure S1 in shape and stability range
- NMI and ARI are approximately flat for rho/lambda in the interval 6–20 across all ten datasets
- Best or near-best performance occurs at the default rho/lambda = 10
- Sweep table contains one NMI/ARI entry for every (dataset, ratio) pair with no missing values
- Computed NMI and ARI are within valid ranges and consistent with the labelled cell-type ground truth

## Limitations

- SSRE is relatively time-consuming on large-scale datasets, which inflates sweep runtime; the authors suggest parallel computing as an unimplemented acceleration strategy
- Stability was demonstrated only on the paper's ten real scRNA-seq datasets; the stable 6–20 interval may not transfer to other data
- The sweep holds all other parameters (e.g. the three pairwise similarities used by default) fixed, so interactions between rho/lambda and other settings are not explored

## Evidence

- [abstract] SSRE MATLAB/Python availability: "The matlab and python implementations of SSRE are available at https://github.com/CSUBioGroup/SSRE."
- [discussion] Runtime limitation: "One limitation of SSRE is relatively time-consuming in large-scale datasets; therefore, parallel computing is a possible strategy to accelerate the framework [60]."
- [readme] Penalty parameter specification example: "%% Penalty parameters. We use the following specification:
rho=0.2; lam=0.0001; lam2=lam; eta=1; c=0.1;"
- [discussion] Default similarity configuration: "users can choose one or two of the three pairwise similarities mentioned in this study to perform gene selection and similarity enhancement procedures, and all three are used by default"
