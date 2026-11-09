---
name: type-i-error-and-power-assessment
description: 'Use when validating a differential topology/proportion test on multi-sample
  scRNA-seq data: simulated perturbations (e.g., removing myeloid cells from 0–80%
  of samples'' cells) must show controlled type I error at 0% and monotonically increasing
  power.'
when_to_use_negative:
- When the goal is testing for differential gene expression (TDE) or cell density
  (TCD/XCD) changes rather than trajectory topology/branch proportion changes — the
  topology simulations use branchPropTest specifically
- When samples are not replicated (e.g., one sample per condition), since Lamian's
  type I error control relies on accounting for cross-sample variability, which single-sample
  designs cannot support
- When the input data has not been quality-filtered (e.g., HCA-BM cells with <5000
  reads, <1000 expressed genes, or >10% mitochondrial reads), since low-quality cells
  distort branch proportions and bootstrap detection rates
edam_operation: http://edamontology.org/operation_2239
edam_topics:
- http://edamontology.org/topic_3308
- http://edamontology.org/topic_0203
tools:
- name: Lamian
  role: Provides infer_tree_structure (number.cluster = 5, origin.marker = 'CD34'),
    evaluate_uncertainty (bootstrap detection rates), and branchPropTest (binomial
    logistic regression on branch cell proportions) used to run and validate the topology
    simulations
  repo: https://github.com/Winnie09/Lamian
- name: TSCAN
  role: Default cluster-based minimum spanning tree method Lamian uses to construct
    the pseudotime tree whose branches are subsampled and re-detected in the power/type
    I error simulations
  repo: https://github.com/zji90/TSCAN
provenance:
  source_task_ids:
  - task_002
  source_papers:
  - doi: 10.1038/s41467-023-42841-y
    title: A statistical framework for differential pseudotime analysis with multiple
      single-cell RNA-seq samples
  source_content_hash: sha256:b1288e92426d34f92c6314a23ed3292760c2554c147d8d687a3fa4eff6508210
schema_version: 0.2.0
metadata:
  license_tier: open
  repo_url: https://github.com/Winnie09/Lamian
  tools:
  - Lamian
  - TSCAN
  edam_operation: http://edamontology.org/operation_2239
  edam_topics:
  - http://edamontology.org/topic_3308
  - http://edamontology.org/topic_0203
  provenance_tier: literature
license: CC-BY-4.0
triage_status: promoted
yes_rate: 0.7142857142857143
n_claims: 7
---

# type-i-error-and-power-assessment

## Summary

A simulation-based validation skill that assesses whether Lamian's differential topology tests control the type I error rate under no difference (0% cell removal) and gain statistical power as the true difference (myeloid cell removal percentage) increases, using the 8-sample HCA-BM scRNA-seq data.

## When to use

Use when validating a differential topology/proportion test on multi-sample scRNA-seq data: simulated perturbations (e.g., removing myeloid cells from 0–80% of samples' cells) must show controlled type I error at 0% and monotonically increasing power.

## When NOT to use

- When the goal is testing for differential gene expression (TDE) or cell density (TCD/XCD) changes rather than trajectory topology/branch proportion changes — the topology simulations use branchPropTest specifically
- When samples are not replicated (e.g., one sample per condition), since Lamian's type I error control relies on accounting for cross-sample variability, which single-sample designs cannot support
- When the input data has not been quality-filtered (e.g., HCA-BM cells with <5000 reads, <1000 expressed genes, or >10% mitochondrial reads), since low-quality cells distort branch proportions and bootstrap detection rates

## Inputs

- HCA-BM scRNA-seq expression matrix (hca_bm_saver, raw counts from 8 healthy donors, Human Cell Atlas 10x Genomics data)
- PCA embedding (hca_bm_pca) and cell annotation table (hca_bm_cellanno) with myeloid-lineage labels relative to CD34+ origin
- Simulated design matrix encoding which samples have myeloid cells removed (Sim 1: all 8 samples; Sim 2: 4 of 8)

## Outputs

- Branch detection rate per removal level (0%–80%) from bootstrap tree reconstruction
- Type I error estimate at 0% removal (should be near nominal level)
- Power curve across removal percentages for Simulation 2 (should increase monotonically)
- Myeloid-branch detection rate at 80% removal in Sim 1 (~0.106)

## How to apply

Load the HCA-BM expression matrix (hca_bm_saver), PCA (hca_bm_pca), and cell annotations (hca_bm_cellanno), then infer the pseudotime tree with infer_tree_structure (number.cluster = 5, origin.marker = 'CD34'). Subsample myeloid-lineage cells at 0%–80% removal levels under two designs: Simulation 1 removes myeloid cells from all 8 samples, Simulation 2 removes them from 4 of 8 samples. For each removal level, call evaluate_uncertainty with n.permute >= 100 (per the paper's uncertainty framework, detection rate is characterized via bootstrap tree reconstruction using Jaccard statistics and overlap coefficient). Compute per-sample branch cell proportions and test differential topology with branchPropTest, which fits a binomial logistic regression against the simulated design matrix. Tabulate type I error at 0% removal and power across removal levels, comparing the myeloid-branch detection rate at 80% removal in Simulation 1 against the reference value ~0.106.

## Related tools

- **Lamian** (Provides infer_tree_structure (number.cluster = 5, origin.marker = 'CD34'), evaluate_uncertainty (bootstrap detection rates), and branchPropTest (binomial logistic regression on branch cell proportions) used to run and validate the topology simulations) — https://github.com/Winnie09/Lamian
- **TSCAN** (Default cluster-based minimum spanning tree method Lamian uses to construct the pseudotime tree whose branches are subsampled and re-detected in the power/type I error simulations) — https://github.com/zji90/TSCAN

## Examples

```
res <- infer_tree_structure(pca = man_tree_data[['pca']], number.cluster = 5, origin.marker = 'CD34'); result <- evaluate_uncertainty(res, n.permute=100)
```

## Evaluation signals

- Type I error at 0% removal is controlled (near nominal level) under Simulation 2, confirming false positives are not inflated when no difference exists
- Power increases monotonically as the myeloid removal percentage rises from 0% to 80% in Simulation 2
- Myeloid-branch detection rate at 80% removal in Simulation 1 is approximately 0.106, matching the reference value
- evaluate_uncertainty uses n.permute >= 100 (the paper characterizes uncertainty via detection rate in 1000 bootstrap samples) to stabilize detection-rate estimates
- branchPropTest fits a binomial logistic regression per branch and yields differential topology calls consistent with the simulated 4-of-8 sample design

## Limitations

- Lamian modules 1 and 2 do not support Monocle2, Monocle3, or Slingshot trajectories, so the simulation workflow is tied to the TSCAN default trajectory
- Simulations for TDE, TCD, and XCD detection are deferred to Supplementary Notes; this skill's validation covers differential topology only
- Accounting simultaneously for all sources of uncertainty (pseudotime inference, cell abundance, cross-sample variability) remains a stated future research topic
- Installation requires removing any pre-existing TSCAN and installing the updated version from GitHub (zji90/TSCAN), plus ComplexHeatmap which may need removal from DESCRIPTION Imports if installation fails (heatmaps then unavailable)

## Evidence

- [discussion] For each of the branch, we characterize its uncertainty using its detection rate in 1000 bootstrap samples.: "For each of the branch, we characterize its uncertainty using its detection rate in 1000 bootstrap samples."
- [discussion] Differential topology testing via binomial logistic regression: "To test differential topology, by default a binomial logistic regression model is fitted for each branch."
- [abstract] Lamian accounts for cross-sample variability: [evidence span withheld — anchor status: not_found]
- [readme] Lamian controls FDR in multi-sample data: "Lamian is able to more appropriately control the false discovery rate (FDR) when analyzing multi-sample data"
- [methods] evaluate_uncertainty invocation example: "result <- evaluate_uncertainty(res, n.permute=5)"
