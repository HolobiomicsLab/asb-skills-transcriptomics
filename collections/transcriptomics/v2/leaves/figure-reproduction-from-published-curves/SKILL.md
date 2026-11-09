---
name: figure-reproduction-from-published-curves
description: Use when a published clustering paper claims a hyperparameter (e.g.,
  a penalty ratio rho/lambda) is stable over an interval and you must verify that
  claim by sweeping the parameter on its original benchmark datasets.
when_to_use_negative:
- The goal is to compare SSRE against competing clustering methods rather than to
  verify a specific hyperparameter's stability range — a benchmark comparison workflow
  is more appropriate
- The dataset is a large-scale scRNA-seq set where SSRE's known runtime limitation
  makes a full 15-value sweep prohibitively expensive without parallel computing
- The target figure does not depend on a continuous numeric hyperparameter (e.g.,
  qualitative t-SNE visualization figures), so a ratio sweep cannot reproduce it
edam_operation: http://edamontology.org/operation_0284
edam_topics:
- http://edamontology.org/topic_0203
- http://edamontology.org/topic_3308
tools:
- name: MATLAB
  role: Runtime for executing the SSRE parameter sweep across rho/lambda values and
    computing NMI/ARI per dataset
  repo: https://github.com/CSUBioGroup/SSRE
- name: Python
  role: Alternative runtime for the SSRE implementation when reproducing the sweep
    and plotting the stability curves
  repo: https://github.com/CSUBioGroup/SSRE
provenance:
  source_task_ids:
  - task_004
  source_papers:
  - doi: 10.1016/j.gpb.2020.09.004
    title: 'SSRE: Cell Type Detection Based on Sparse Subspace Representation and
      Similarity Enhancement'
  source_content_hash: sha256:50fb800238881bbdc892df80df446de99825f8c21e59437df1338230156a000a
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSUBioGroup/SSRE
  tools:
  - MATLAB
  - Python
  edam_operation: http://edamontology.org/operation_0284
  edam_topics:
  - http://edamontology.org/topic_0203
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 1.0
n_claims: 7
---

# figure-reproduction-from-published-curves

## Summary

Reproduces published parameter-sensitivity curves (e.g., SSRE's NMI/ARI stability versus the rho/lambda penalty ratio in Figures 2 and S1) by systematically sweeping the parameter on the paper's original benchmark datasets and comparing against its stated stable range (6–20) and default (rho/lambda = 10).

## When to use

Use when a published clustering paper claims a hyperparameter (e.g., a penalty ratio rho/lambda) is stable over an interval and you must verify that claim by sweeping the parameter on its original benchmark datasets.

## When NOT to use

- The goal is to compare SSRE against competing clustering methods rather than to verify a specific hyperparameter's stability range — a benchmark comparison workflow is more appropriate
- The dataset is a large-scale scRNA-seq set where SSRE's known runtime limitation makes a full 15-value sweep prohibitively expensive without parallel computing
- The target figure does not depend on a continuous numeric hyperparameter (e.g., qualitative t-SNE visualization figures), so a ratio sweep cannot reproduce it

## Inputs

- Ten real scRNA-seq benchmark datasets from the CSUBioGroup/SSRE repository (cell-by-gene expression matrices plus known cell type labels)
- SSRE MATLAB or Python implementation from https://github.com/CSUBioGroup/SSRE
- Penalty coefficient values for rho/lambda: {2, 4, 6, 8, 10, 12, 14, 16, 18, 20, 22, 24, 26, 28, 30}

## Outputs

- NMI and ARI scores for every (dataset, rho/lambda) pair
- rho_lambda_sensitivity.csv containing the full parameter-sweep table
- Stability curves of NMI and ARI versus rho/lambda mirroring Figures 2 and S1

## How to apply

Obtain the ten real scRNA-seq benchmark datasets and the SSRE MATLAB/Python implementation from the CSUBioGroup/SSRE repository. For each dataset, run SSRE with the penalty ratio rho/lambda set to each value in {2, 4, 6, ..., 30}, keeping all other parameters (e.g., the three pairwise similarities used by default for gene selection and similarity enhancement) at their defaults. Compare the resulting cluster assignments to the known cell type labels by computing Normalized Mutual Information (NMI) and Adjusted Rand Index (ARI) for every (dataset, ratio) pair. Aggregate the scores across the ten datasets and plot NMI and ARI versus rho/lambda to mirror Figures 2 and S1, then write the full sweep table to rho_lambda_sensitivity.csv. The reproduction succeeds when the curves show stable NMI/ARI for ratios in 6–20 and the default rho/lambda = 10 is recovered.

## Related tools

- **MATLAB** (Runtime for executing the SSRE parameter sweep across rho/lambda values and computing NMI/ARI per dataset) — https://github.com/CSUBioGroup/SSRE
- **Python** (Alternative runtime for the SSRE implementation when reproducing the sweep and plotting the stability curves) — https://github.com/CSUBioGroup/SSRE

## Examples

```
rho=2; lam=0.2*rho; [P] = clus_fin_update(rho, lam, lam2, eta, c, in_X, true_labs); [NMI, ~,clus_labs,~]=calc2_nmis(max(true_labs), double(P),true_labs);
```

## Evaluation signals

- The reproduced NMI/ARI curves should match Figures 2 and S1 of the SSRE paper
- NMI and ARI should remain stable (no large drops) for rho/lambda in the interval 6–20
- The default setting rho/lambda = 10 should yield performance consistent with the paper's reported default best performance
- The sweep table covers all ten real scRNA-seq datasets at all 15 ratio values with no missing (dataset, ratio) entries
- Computed ARI/NMI values agree with the paper's clustering performance claims, e.g., SSRE's superior performance on the ten real datasets

## Limitations

- SSRE is relatively time-consuming on large-scale datasets, which constrains the practical sweep grid size; parallel computing is proposed as a future acceleration strategy
- The claimed stable interval (6–20) and default (10) are validated only on the paper's ten real scRNA-seq and five simulated datasets and may not transfer to other data
- The stable-range claim reflects overall/default parameter choices; users can substitute one or two of the three pairwise similarities, which may shift the sensitivity profile

## Evidence

- [results] SSRE's clustering performance is basically stable when rho/lambda is in the interval of 6 and 20, supporting the default setting of rho/lambda = 10.: [evidence span withheld — anchor status: not_found]
- [abstract] Tested on ten real scRNA-seq datasets and five simulated datasets, SSRE achieved the superior performance in most cases compared to several state-of-the-art single-cell clustering methods.: "Tested on ten real scRNA-seq datasets and five simulated datasets, SSRE achieved the superior performance in most cases compared to several state-of-the-art single-cell clustering methods."
- [discussion] One limitation of SSRE is relatively time-consuming in large-scale datasets; therefore, parallel computing is a possible strategy to accelerate the framework: "One limitation of SSRE is relatively time-consuming in large-scale datasets; therefore, parallel computing is a possible strategy to accelerate the framework"
- [supplementary] The matlab and python implementations of SSRE are available at https://github.com/CSUBioGroup/SSRE.: "The matlab and python implementations of SSRE are available at https://github.com/CSUBioGroup/SSRE."
- [readme] [NMI, ~,clus_labs,~]=calc2_nmis(max(true_labs), double(P),true_labs);   %% Compute performance measures: [evidence span withheld — anchor status: not_found]
