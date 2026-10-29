---
name: nmi-ari-clustering-evaluation
description: Use when a clustering method's predicted cell assignments must be scored
  against ground-truth labels (e.g., Splatter-simulated or annotated scRNA-seq datasets)
  with NMI and ARI to build benchmark comparison tables.
when_to_use_negative:
- No ground-truth cell-type labels exist for the dataset, since NMI/ARI require reference
  assignments
- The task is assessing clustering stability without labels rather than accuracy against
  known cell types
edam_operation: http://edamontology.org/operation_3432
edam_topics:
- http://edamontology.org/topic_3308
tools:
- name: MATLAB
  role: Implementation environment in which NMI/ARI utilities (calc2_nmis, RandIndex)
    are invoked to score SSRE and competing clustering results
  repo: https://github.com/CSUBioGroup/SSRE
- name: Python
  role: Alternative implementation environment for SSRE and for computing NMI/ARI
    across benchmark datasets
  repo: https://github.com/CSUBioGroup/SSRE
- name: MPSSC (ishspsy/project)
  role: Provides the MATLAB evaluation utilities calc2_nmis (NMI) and RandIndex (ARI/purity)
    referenced in the surrounding ecosystem of spectral clustering benchmarks
  repo: https://github.com/ishspsy/project
provenance:
  source_task_ids:
  - task_002
  - task_004
  source_papers:
  - doi: 10.1016/j.gpb.2020.09.004
    title: 'SSRE: Cell Type Detection Based on Sparse Subspace Representation and
      Similarity Enhancement'
  source_content_hash: sha256:e4d81f938fc77b56f9a9ca76d8d96a711cfbbdc820126d137bde7aece748f89d
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/CSUBioGroup/SSRE
  tools:
  - MATLAB
  - MPSSC (ishspsy/project)
  - Python
  edam_operation: http://edamontology.org/operation_3432
  edam_topics:
  - http://edamontology.org/topic_3308
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.6363636363636364
n_claims: 11
---

# nmi-ari-clustering-evaluation

## Summary

Evaluate single-cell clustering output by computing Normalized Mutual Information (NMI) and Adjusted Rand Index (ARI) between predicted cluster assignments and known cell-type labels, as used throughout the SSRE paper to benchmark cell type detection on scRNA-seq data.

## When to use

Use when a clustering method's predicted cell assignments must be scored against ground-truth labels (e.g., Splatter-simulated or annotated scRNA-seq datasets) with NMI and ARI to build benchmark comparison tables.

## When NOT to use

- No ground-truth cell-type labels exist for the dataset, since NMI/ARI require reference assignments
- The task is assessing clustering stability without labels rather than accuracy against known cell types

## Inputs

- Predicted cluster assignment vector for cells from a clustering method (e.g., SSRE output)
- Ground-truth cell-type label vector (simulated Splatter labels or curated real-dataset annotations)
- scRNA-seq count matrices (n cells x p genes, .mat format in MATLAB workflows)

## Outputs

- Per-(method, dataset) NMI and ARI scores
- Benchmark comparison tables mirroring Tables S1 and S2 (e.g., tables_s1_s2_reproduction.csv)
- Parameter-sensitivity table (e.g., rho_lambda_sensitivity.csv) and NMI/ARI stability curves vs rho/lambda

## How to apply

Run the clustering method (e.g., SSRE from CSUBioGroup/SSRE with default settings, or a competing method) on a dataset whose true labels are retained, obtaining cluster assignments per cell. Compute NMI and ARI between the predicted assignments and the true labels — in MATLAB the MPSSC/SSRE-style utilities calc2_nmis and RandIndex can be used for this. For the SSRE benchmark, repeat across the ten real scRNA-seq datasets and the five Splatter-simulated datasets (group.prob=(0.65,0.25,0.1); nCells=1000/1000/1000/500/1500; sparsity 0.61/0.8/0.94/0.94/0.94). For parameter-sweep stability analyses (rho/lambda in {2,4,...,30}, default 10), compute NMI/ARI for every (dataset, ratio) pair and aggregate across datasets before plotting curves. Assemble scores into tables mirroring supplementary Tables S1/S2 and sensitivity CSVs such as rho_lambda_sensitivity.csv.

## Related tools

- **MATLAB** (Implementation environment in which NMI/ARI utilities (calc2_nmis, RandIndex) are invoked to score SSRE and competing clustering results) — https://github.com/CSUBioGroup/SSRE
- **Python** (Alternative implementation environment for SSRE and for computing NMI/ARI across benchmark datasets) — https://github.com/CSUBioGroup/SSRE
- **MPSSC (ishspsy/project)** (Provides the MATLAB evaluation utilities calc2_nmis (NMI) and RandIndex (ARI/purity) referenced in the surrounding ecosystem of spectral clustering benchmarks) — https://github.com/ishspsy/project

## Examples

```
[NMI, ~, clus_labs, ~] = calc2_nmis(max(true_labs), double(P), true_labs); ARI = RandIndex(clus_labs, true_labs)
```

## Evaluation signals

- Reproduced NMI/ARI values match those reported in supplementary Tables S1/S2 within numerical tolerance
- SSRE scores are superior to competing methods in the majority of the ten real and five simulated datasets
- Swept rho/lambda curves show stable NMI/ARI in the interval 6–20 with the default rho/lambda=10 performing well
- The output CSV contains NMI and ARI entries for SSRE and every competing method across all five simulated datasets

## Limitations

- SSRE (whose clustering output is being scored) is relatively time-consuming on large-scale datasets
- Evaluation depends on retained true labels; simulated data rely on Splatter's known generative labels

## Evidence

- [abstract] Benchmarking on ten real and five simulated datasets: "Tested on ten real scRNA-seq datasets and five simulated datasets, SSRE achieved the superior performance in most cases compared to several state-of-the-art single-cell clustering methods."
- [discussion] Evaluation across multiple scRNA-seq datasets: "The systematic performance evaluations on multiple scRNA-seq datasets showed that SSRE achieves superior performance among all competing methods."
- [readme] MATLAB NMI computation pattern: [evidence span withheld — anchor status: not_found]
- [discussion] Runtime limitation caveat: "One limitation of SSRE is relatively time-consuming in large-scale datasets"
